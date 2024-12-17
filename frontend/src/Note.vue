<!-- frontend/src/Note.vue -->
<script setup>
import { ref, onMounted, defineEmits } from 'vue';
import { HOST } from './config';

// Define props
const props = defineProps({
  note: {
    type: Object,
    required: true,
  },
});

// Define emits for deleting a note
const emit = defineEmits(['delete-note']);

// Reactive variables
const tasks = ref([]);              // List of tasks
const isLoading = ref(true);        // Loading state
const errorMessage = ref('');       // Error message
const newTaskDescription = ref(''); // New task input

// Fetch tasks from the backend
const fetchTasks = async () => {
  try {
    const response = await fetch(`${HOST}/api/notes/${props.note.id}/tasks`);
    if (!response.ok) {
      throw new Error(`Error: ${response.status} ${response.statusText}`);
    }
    const data = await response.json();
    tasks.value = data;
  } catch (error) {
    console.error('Error fetching tasks:', error);
    errorMessage.value = 'Failed to load tasks.';
  } finally {
    isLoading.value = false;
  }
};

// Create a new task
const createTask = async () => {
  const description = newTaskDescription.value.trim();
  if (description === '') {
    alert('Task description cannot be empty.');
    return;
  }

  try {
    const response = await fetch(`${HOST}/api/notes/${props.note.id}/tasks`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        content: description,
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.error || 'Failed to create task.');
    }

    const newTask = await response.json();
    tasks.value.push(newTask); // Add the new task to the list
    newTaskDescription.value = ''; // Clear the input field
  } catch (error) {
    console.error('Error creating task:', error);
    alert(error.message);
  }
};

// Handle the Enter key press
const handleKeyPress = (event) => {
  if (event.key === 'Enter') {
    createTask();
  }
};

// Delete a task
const deleteTask = async (taskId) => {
  if (!confirm('Are you sure you want to delete this task?')) return;

  try {
    const response = await fetch(`${HOST}/api/tasks/${taskId}`, {
      method: 'DELETE',
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.error || 'Failed to delete task.');
    }

    // Remove the task from the list
    tasks.value = tasks.value.filter((task) => task.id !== taskId);
  } catch (error) {
    console.error('Error deleting task:', error);
    alert(error.message);
  }
};

// Delete the entire note
const deleteNote = () => {
  if (!confirm('Are you sure you want to delete this note?')) return;
  emit('delete-note', props.note.id);
};

// Fetch tasks when the component is mounted
onMounted(() => {
  fetchTasks();
});
</script>

<template>
  <div :class="['note', note.status]">
    <!-- Delete Note Button -->
    <div class="delete-button" @click="deleteNote" title="Delete Note">&#x1F5D1;</div>
    
    <!-- Note Title -->
    <h2>{{ note.title }}</h2>

    <div class="tasks">
      <!-- Loading Indicator -->
      <div v-if="isLoading">Loading tasks...</div>

      <!-- Error Message -->
      <div v-else-if="errorMessage" class="error">
        {{ errorMessage }}
      </div>

      <!-- Tasks List -->
      <div v-else>
        <!-- Render each task dynamically -->
        <div v-for="task in tasks" :key="task.id" class="task">
          <div class="content">{{ task.content }}</div>
          <div class="delete-button" @click="deleteTask(task.id)" title="Delete Task">&#x1F5D1;</div>
        </div>

        <!-- New Task Input -->
        <div class="new-task">
          <input
            class="content"
            placeholder="Enter new task..."
            v-model="newTaskDescription"
            @keyup="handleKeyPress"
          />
          <button class="create-btn" @click="createTask">+</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use 'assets/mediaQueryScreens.scss' as *;
@use 'assets/colors.scss' as *;

$gutter-size: 15px;

.note {
  min-height: 200px;
  border-radius: 10px;
  padding: 10px;
  color: white;
  position: relative;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

  /* Dynamic background color based on status */
  &.unimportant {
    background-color: $dark-green;

    & > input {
      background-color: $light-green;
      color: darken($dark-green, 20%);
    }

    .create-btn {
      color: $dark-green;
    }
  }

  &.serious {
    background-color: $dark-yellow;

    & > input {
      background-color: $light-yellow;
      color: darken($dark-yellow, 20%);
    }

    .create-btn {
      color: $dark-yellow;
    }
  }

  &.urgent {
    background-color: $dark-red;

    & > input {
      background-color: $light-red;
      color: darken($dark-red, 20%);
    }

    .create-btn {
      color: $dark-red;
    }
  }

  /* Delete Button Styling */
  & > .delete-button {
    position: absolute;
    top: 10px;
    right: 10px;
    font-size: 20px;
    cursor: pointer;
    color: inherit;
    transition: color 0.3s;

    &:hover {
      color: rgba(255, 255, 255, 0.7);
    }
  }

  /* Note Title Styling */
  h2 {
    padding-bottom: 10px;
    margin: 0;
    font-size: 1.5em;
    border-bottom: 1px solid rgba(255, 255, 255, 0.3);
  }

  /* Tasks Section */
  .tasks {
    margin-top: 15px;

    /* Error Message Styling */
    .error {
      color: rgba(255, 0, 0, 0.8);
      font-weight: bold;
      margin-bottom: 10px;
    }

    /* Individual Task Styling */
    .task {
      padding: 10px;
      margin-bottom: 10px;
      border-radius: 5px;
      background-color: rgba(255, 255, 255, 0.1);
      display: flex;
      align-items: center;
      transition: background-color 0.3s;

      &:hover {
        background-color: rgba(255, 255, 255, 0.2);
      }

      .content {
        flex-grow: 1;
        font-size: 1em;
      }

      /* Delete Task Button Styling */
      .delete-button {
        font-size: 18px;
        color: rgba(255, 255, 255, 0.6);
        transition: color 0.3s;

        &:hover {
          color: rgba(255, 255, 255, 0.9);
        }
      }
    }

    /* New Task Input and Button */
    .new-task {
      display: flex;
      align-items: center;
      margin-top: 10px;

      & > input {
        flex-grow: 1;
        padding: 10px;
        border: none;
        border-radius: 5px;
        font-size: 1em;
        margin-right: 10px;

        &::placeholder {
          color: rgba(255, 255, 255, 0.7);
          font-style: italic;
        }

        &:focus {
          outline: none;
          box-shadow: 0 0 5px rgba(255, 255, 255, 0.5);
        }
      }

      & > .create-btn {
        width: 40px;
        height: 40px;
        border: none;
        border-radius: 50%;
        background-color: white;
        color: inherit;
        font-size: 1.5em;
        cursor: pointer;
        transition: background-color 0.3s, transform 0.2s;

        &:hover {
          background-color: rgba(255, 255, 255, 0.8);
          transform: scale(1.1);
        }

        &:active {
          transform: scale(0.95);
        }
      }
    }
  }
}

@include smallScreen {
  .note {
    width: 100%;
  }
}

@include mediumScreen {
  .note {
    width: calc((100% - $gutter-size) / 2);
  }
}

@include largeScreen {
  .note {
    width: calc((100% - ($gutter-size * 2)) / 3);
  }
}

/* Optional: Additional styles for error messages outside the .note component */
.error {
  color: red;
  font-weight: bold;
}
</style>

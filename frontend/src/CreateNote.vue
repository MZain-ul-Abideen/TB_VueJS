<!-- frontend/src/CreateNote.vue -->
<script setup>
import { ref } from 'vue';
import { HOST } from './config'; // Ensure this path is correct
import { defineEmits } from 'vue';

// Define the events this component can emit
const emit = defineEmits(['note-created']);

// Reactive variables
const newNoteTitle = ref('');           // Holds the new note's title
const newNoteStatus = ref('unimportant'); // Holds the new note's status

// Function to create a new note
const createNote = async () => {
  const title = newNoteTitle.value.trim();
  const status = newNoteStatus.value;

  if (title === '') {
    alert('Note title cannot be empty.');
    return;
  }

  try {
    const response = await fetch(`${HOST}/notes`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ title, status }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.error || 'Failed to create note.');
    }

    const newNote = await response.json();
    emit('note-created', newNote); // Emit the new note to the parent component

    // Reset input fields
    newNoteTitle.value = '';
    newNoteStatus.value = 'unimportant';
  } catch (error) {
    console.error('Error creating note:', error);
    alert(error.message);
  }
};

// Handle the Enter key press in the title input
const handleKeyPress = (event) => {
  if (event.key === 'Enter') {
    createNote();
  }
};
</script>

<template>
  <div :class="['note', newNoteStatus]">
    <!-- Note Title Input -->
    <input
      class="new-note-title"
      placeholder="Enter note title..."
      v-model="newNoteTitle"
      @keyup="handleKeyPress"
    />

    <!-- Status Selection -->
    <div class="status-select">
      <div>
        <input
          type="radio"
          id="unimportant"
          v-model="newNoteStatus"
          value="unimportant"
        />
        <label for="unimportant">Unimportant</label>
      </div>

      <div>
        <input
          type="radio"
          id="serious"
          v-model="newNoteStatus"
          value="serious"
        />
        <label for="serious">Serious</label>
      </div>

      <div>
        <input
          type="radio"
          id="urgent"
          v-model="newNoteStatus"
          value="urgent"
        />
        <label for="urgent">Urgent</label>
      </div>
    </div>

    <!-- Create Note Button -->
    <button class="create-btn" @click="createNote">Create new note</button>
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
  position: relative; /* To position delete button if needed */

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

  /* Title Input Styling */
  & > .new-note-title {
    padding: 15px 5px;
    border: 0;
    font-size: 20px;
    width: 100%;
    border-radius: 5px;

    &::placeholder {
      font-style: italic;
      color: white;
    }
  }

  /* Status Selection Styling */
  .status-select {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 10px;
    margin-bottom: 10px;

    input {
      margin-right: 5px;
    }
  }

  /* Create Button Styling */
  & > .create-btn {
    padding: 10px 5px;
    width: 100%;
    background-color: white;
    color: darken($dark-green, 20%); /* This line was commented out; ensure it's styled dynamically */
    font-weight: bold;
    font-size: 15px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s, color 0.3s;
  }

  & > .create-btn:hover {
    background-color: #f0f0f0;
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
</style>

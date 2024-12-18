<script setup>
import { ref, onMounted, computed } from 'vue';
import Note from './Note.vue';
import { HOST } from './config.js';
import CreateNote from './CreateNote.vue';
import NoteStatusFilter from './NoteStatusFilter.vue';

const notesList = ref([]); // Reactive list for notes
const selectedFilter = ref("all"); // Default to "all" to show all notes initially

// Function to fetch notes
const fetchNotes = async () => {
  try {
    const response = await fetch(`${HOST}/notes`);
    if (!response.ok) {
      throw new Error('Failed to fetch notes');
    }
    notesList.value = await response.json();
  } catch (error) {
    console.error(error);
    alert('Unable to load notes. Please try again later.');
  }
};

// Add new note to the list
const addNoteToList = (newNote) => {
  notesList.value.push(newNote);
};

// Delete note from the list
const deleteNoteFromList = (noteId) => {
  notesList.value = notesList.value.filter(note => note.id !== noteId);
};

// Handle filter change
const onFilterSelected = (filter) => {
  selectedFilter.value = filter; // Update the selected filter
};

// Compute filtered notes based on the selected filter
const filteredNotes = computed(() => {
  if (selectedFilter.value === "all") {
    return notesList.value; // Return all notes if "all" is selected
  }
  return notesList.value.filter(note => note.status === selectedFilter.value);
});

// Fetch notes when the component is mounted
onMounted(fetchNotes);
</script>

<template>
  <div class="app-title">
    <h1>EMSE NoteApp</h1>
  </div>
  
  <div class="row justify-content-center">
    <CreateNote @note-created="addNoteToList" />
  </div>

  <div v-if="notesList.length === 0" class="empty-notes-message">
    No notes available. Create one!
  </div>

  <div v-else>
    <div class="row filter-div">
      <NoteStatusFilter class="my-4" @filter-selected="onFilterSelected" />
    </div>
    <div class="notes-container">
      <Note 
        v-for="note in filteredNotes" 
        :key="note.id" 
        :note="note" 
        :host="HOST"
        @delete-note="deleteNoteFromList"
      />
    </div>
  </div>
</template>

<style lang="scss" scoped>
@use 'assets/mediaQueryScreens.scss' as *;

$gutter-size: 15px;

.notes-container {
  display: flex;
  flex-wrap: wrap;
  gap: $gutter-size * 1;
  margin: auto;
  border: 1px solid gray;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: $gutter-size;
  margin-bottom: 25px;
  @include smallScreen {
    width: 90vw;
  }

  @include mediumScreen {
    width: 75vw;
  }

  @include largeScreen {
    width: 960px;
  }
}

.filter-div {
  width: 50vw;
  margin: 0 auto; /* Centers the element horizontally */
  display: flex; /* Optional: To align child elements inside */
  justify-content: center; /* Optional: Centers child content inside */
}

.empty-notes-message {
  font-size: 18px;
  color: gray;
  text-align: center;
  margin: 20px 0;
}
</style>

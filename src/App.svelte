<script>
  import { onMount } from 'svelte';

  let notes = [];
  let title = '';
  let content = '';
  let isEditing = false;
  let editId = null;
  let confirmDeleteId = null;

  const API_URL = 'https://684ea381f0c9c9848d28b086.mockapi.io/api/v1/notes';

  onMount(fetchNotes);

  async function fetchNotes() {
    const res = await fetch(API_URL);
    notes = await res.json();
  }

  async function createNote() {
    await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content })
    });
    title = '';
    content = '';
    fetchNotes();
  }

  function startEdit(note) {
    title = note.title;
    content = note.content;
    isEditing = true;
    editId = note.id;
  }

  async function updateNote() {
    await fetch(`${API_URL}/${editId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content })
    });
    isEditing = false;
    editId = null;
    title = '';
    content = '';
    fetchNotes();
  }

  function askDelete(id) {
    confirmDeleteId = id;
  }

  function cancelDelete() {
    confirmDeleteId = null;
  }

  async function confirmAndDelete() {
    await fetch(`${API_URL}/${confirmDeleteId}`, { method: 'DELETE' });
    confirmDeleteId = null;
    fetchNotes();
  }
</script>

<main class="min-h-screen bg-slate-100 text-slate-800 px-4 py-10 flex flex-col items-center">
  <h1 class="text-4xl font-bold mb-8 flex items-center gap-2 border-b pb-4 w-full max-w-xl justify-center">
    📝 Notes App
  </h1>

  <!-- Form -->
  <form
    on:submit|preventDefault={isEditing ? updateNote : createNote}
    class="w-full max-w-xl bg-white p-6 rounded-lg shadow space-y-4 mb-8"
  >
    <div class="mb-4">
      <label class="block text-gray-700 font-medium mb-1">Title</label>
      <input
        bind:value={title}
        class="w-full p-3 text-base rounded border border-gray-300 bg-white text-slate-800"
        placeholder="Title"
        required
      />
    </div>

    <div class="mb-4">
      <label class="block text-gray-700 font-medium mb-1">Content</label>
      <textarea
        bind:value={content}
        rows="4"
        class="w-full p-3 text-base rounded border border-gray-300 bg-white text-slate-800 resize-none"
        placeholder="Content"
        required
      ></textarea>
    </div>

    <button
      class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded"
    >
      {isEditing ? 'Update' : 'Create'} Note
    </button>
  </form>

  <!-- Notes List -->
  <div class="w-full max-w-xl space-y-4">
    {#each notes as note}
      <div class="bg-white p-4 rounded-lg shadow space-y-2">
        <h2 class="text-lg font-bold">{note.title}</h2>
        <p class="text-gray-700">{note.content}</p>
        <div class="flex gap-4 mt-2">
          <button
            on:click={() => startEdit(note)}
            class="bg-yellow-400 hover:bg-yellow-500 text-black font-medium px-4 py-1 rounded"
          >
            Edit
          </button>
          <button
            on:click={() => askDelete(note.id)}
            class="bg-red-500 hover:bg-red-600 text-white font-medium px-4 py-1 rounded"
          >
            Delete
          </button>
        </div>
      </div>
    {/each}
  </div>

  <!-- Confirmation Modal -->
  {#if confirmDeleteId}
    <div class="fixed inset-0 bg-black bg-opacity-40 flex items-center justify-center z-50">
      <div class="bg-white text-slate-800 p-6 rounded shadow space-y-4 w-80">
        <h3 class="text-lg font-semibold">Are you sure you want to delete?</h3>
        <div class="flex justify-end gap-4">
          <button on:click={cancelDelete} class="px-3 py-1 rounded bg-gray-300 hover:bg-gray-400">
            Cancel
          </button>
          <button on:click={confirmAndDelete} class="px-3 py-1 rounded bg-red-600 text-white hover:bg-red-700">
            Delete
          </button>
        </div>
      </div>
    </div>
  {/if}
</main>

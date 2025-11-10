<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-8 px-4">
    <div class="max-w-2xl mx-auto">
      <!-- Header -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <h1 class="text-3xl font-bold text-gray-800 mb-2">📝 My Todos</h1>
        <p class="text-gray-600">Organizza le tue attività giornaliere</p>
      </div>

      <!-- Form aggiungi todo -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <form @submit.prevent="addTodo" class="flex gap-2">
          <input
            v-model="newTodoTitle"
            type="text"
            placeholder="Aggiungi una nuova todo..."
            class="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
            @keyup.enter="addTodo"
          />
          <button
            type="submit"
            :disabled="!newTodoTitle.trim()"
            class="bg-blue-500 hover:bg-blue-600 disabled:bg-gray-400 text-white px-6 py-2 rounded-lg font-semibold transition"
          >
            Aggiungi
          </button>
        </form>
      </div>

      <!-- Loading state -->
      <div v-if="isLoading" class="bg-white rounded-lg shadow p-6 text-center">
        <p class="text-gray-500">⏳ Caricamento...</p>
      </div>

      <!-- Error state -->
      <div v-if="errorMessage" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg mb-6">
        ❌ {{ errorMessage }}
      </div>

      <!-- Lista todos -->
      <div class="space-y-3">
        <div
          v-if="todos.length === 0 && !isLoading"
          class="bg-white rounded-lg shadow p-6 text-center text-gray-500"
        >
          Nessuna todo! 🎉 Aggiungi una nuova attività per iniziare.
        </div>

        <div
          v-for="todo in todos"
          :key="todo.id"
          class="bg-white rounded-lg shadow-lg p-4 flex items-center gap-4 hover:shadow-xl transition"
        >
          <!-- Checkbox per completare -->
          <input
            type="checkbox"
            :checked="todo.completed"
            @change="toggleTodo(todo)"
            class="w-5 h-5 cursor-pointer accent-blue-500"
          />

          <!-- Titolo -->
          <span
            class="flex-1 text-lg transition"
            :class="
              todo.completed
                ? 'line-through text-gray-400'
                : 'text-gray-800'
            "
          >
            {{ todo.title }}
          </span>

          <!-- Badge completata -->
          <span
            v-if="todo.completed"
            class="bg-green-100 text-green-800 px-3 py-1 rounded-full text-sm font-semibold"
          >
            ✓ Completata
          </span>

          <!-- Bottone elimina -->
          <button
            @click="deleteTodo(todo.id)"
            class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg transition"
          >
            Elimina
          </button>
        </div>
      </div>

      <!-- Stats -->
      <div v-if="todos.length > 0" class="mt-8 bg-white rounded-lg shadow p-6">
        <div class="grid grid-cols-3 gap-4 text-center">
          <div>
            <p class="text-2xl font-bold text-blue-600">{{ todos.length }}</p>
            <p class="text-gray-600">Totali</p>
          </div>
          <div>
            <p class="text-2xl font-bold text-green-600">{{ completedCount }}</p>
            <p class="text-gray-600">Completate</p>
          </div>
          <div>
            <p class="text-2xl font-bold text-orange-600">{{ percentualeCompletate }}%</p>
            <p class="text-gray-600">Progresso</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Costanti
const API_URL = 'http://localhost:8000/api'

// State
const todos = ref([])
const newTodoTitle = ref('')
const isLoading = ref(false)
const errorMessage = ref('')

// Lifecycle: carica le todos al mount
onMounted(async () => {
  await fetchTodos()
})

// Fetch tutte le todos dall'API
async function fetchTodos() {
  isLoading.value = true
  errorMessage.value = ''

  try {
    const response = await fetch(`${API_URL}/todos`)
    if (!response.ok) throw new Error('Errore nel caricamento')
    todos.value = await response.json()
  } catch (error) {
    console.error('Errore:', error)
    errorMessage.value = 'Impossibile caricare le todos'
  } finally {
    isLoading.value = false
  }
}

// Aggiungi una nuova todo
async function addTodo() {
    if (!newTodoTitle.value.trim()) return

    try {
        const response = await fetch(`${API_URL}/todos`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                title: newTodoTitle.value,
                completed: false,
            }),
        })

        if (!response.ok) throw new Error('Errore nell\'aggiunta');

        const newTodo = await response.json();
        todos.value.push(newTodo);
        newTodoTitle.value = '';
        errorMessage.value = '';

    } catch (error) {
        console.error('Errore:', error)
        errorMessage.value = 'Impossibile aggiungere la todo'
    }
}

// Toggle completed (mark as done/not done)
async function toggleTodo(todo) {
    try {
        const response = await fetch(`${API_URL}/todos/${todo.id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                completed: !todo.completed,
            }),
        })

        if (!response.ok) throw new Error('Errore nell\'aggiornamento')

        const updatedTodo = await response.json();
        const index = todos.value.findIndex((t) => t.id === todo.id);
        todos.value[index] = updatedTodo;
        errorMessage.value = '';

    } catch (error) {
        console.error('Errore:', error)
        errorMessage.value = 'Impossibile aggiornare la todo'
    }
}

// Elimina una todo
async function deleteTodo(id) {
    try {
        const response = await fetch(`${API_URL}/todos/${id}`, {
            method: 'DELETE',
        })

        if (!response.ok) throw new Error('Errore nell\'eliminazione')

        todos.value = todos.value.filter((t) => t.id !== id);
        errorMessage.value = '';

    } catch (error) {
        console.error('Errore:', error)
        errorMessage.value = 'Impossibile eliminare la todo'
    }
}

// Computed: conteggio completate
const completedCount = computed(() => {
    return todos.value.filter((t) => t.completed).length
})

// Computed: percentuale di completamento
const percentualeCompletate = computed(() => {
    if (todos.value.length === 0) return 0
    return Math.round((completedCount.value / todos.value.length) * 100)
})
</script>

<style scoped>
/* Tailwind CSS gestisce tutto! */
</style>
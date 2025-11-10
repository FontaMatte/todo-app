<template>
  <div class="app">
    <!-- Sidebar Left -->
    <div class="sidebar">
      <div class="logo">
        <h1>Tasks</h1>
        <p>Simple & Clean</p>
      </div>

      <!-- Stats -->
      <div v-if="todos.length > 0" class="stats">
        <div class="stat">
          <span class="stat-value">{{ todos.length }}</span>
          <span class="stat-label">Totali</span>
        </div>
        <div class="stat">
          <span class="stat-value">{{ completedCount }}</span>
          <span class="stat-label">Completate</span>
        </div>
        <div class="stat">
          <span class="stat-value">{{ percentualeCompletate }}%</span>
          <span class="stat-label">Fatto</span>
        </div>
      </div>

      <!-- Progress Bar -->
      <div v-if="todos.length > 0" class="progress-bar">
        <div 
          class="progress-fill"
          :style="{ width: percentualeCompletate + '%' }"
        ></div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main">
      <!-- Header -->
      <div class="header">
        <h2>My Tasks</h2>
        <p class="subtitle">{{ todos.length }} attività totali</p>
      </div>

      <!-- Input Form -->
      <form @submit.prevent="addTodo" class="input-form">
        <input
          v-model="newTodoTitle"
          type="text"
          placeholder="Aggiungi una nuova attività..."
          class="input"
        />
        <button 
          type="submit" 
          :disabled="!newTodoTitle.trim()"
          class="btn"
        >
          Aggiungi
        </button>
      </form>

      <!-- Loading -->
      <div v-if="isLoading" class="loading">
        Caricamento...
      </div>

      <!-- Error -->
      <div v-if="errorMessage" class="error">
        {{ errorMessage }}
      </div>

      <!-- Empty State -->
      <div v-if="todos.length === 0 && !isLoading" class="empty">
        <div class="empty-icon">📝</div>
        <p>Nessuna attività. Inizia ad aggiungerne una!</p>
      </div>

      <!-- Todo List -->
      <div v-if="todos.length > 0" class="todo-list">
        <div
          v-for="todo in todos"
          :key="todo.id"
          class="todo-item"
          :class="{ completed: todo.completed }"
        >
          <input
            type="checkbox"
            :checked="todo.completed"
            @change="toggleTodo(todo)"
            class="checkbox"
          />
          <span class="title">{{ todo.title }}</span>
          <button
            @click="deleteTodo(todo.id)"
            class="delete-btn"
          >
            ✕
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const API_URL = 'http://localhost:8000/api'

const todos = ref([])
const newTodoTitle = ref('')
const isLoading = ref(false)
const errorMessage = ref('')

onMounted(async () => {
  await fetchTodos()
})

async function fetchTodos() {
  isLoading.value = true
  errorMessage.value = ''

  try {
    const response = await fetch(`${API_URL}/todos`)
    if (!response.ok) throw new Error('Errore nel caricamento')
    todos.value = await response.json()
  } catch (error) {
    console.error('Errore:', error)
    errorMessage.value = 'Impossibile caricare le attività'
  } finally {
    isLoading.value = false
  }
}

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

    if (!response.ok) throw new Error('Errore nell\'aggiunta')

    const newTodo = await response.json()
    todos.value.push(newTodo)
    newTodoTitle.value = ''
    errorMessage.value = ''
  } catch (error) {
    console.error('Errore:', error)
    errorMessage.value = 'Impossibile aggiungere l\'attività'
  }
}

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

    const updatedTodo = await response.json()
    const index = todos.value.findIndex((t) => t.id === todo.id)
    todos.value[index] = updatedTodo
    errorMessage.value = ''
  } catch (error) {
    console.error('Errore:', error)
    errorMessage.value = 'Impossibile aggiornare l\'attività'
  }
}

async function deleteTodo(id) {
  try {
    const response = await fetch(`${API_URL}/todos/${id}`, {
      method: 'DELETE',
    })

    if (!response.ok) throw new Error('Errore nell\'eliminazione')

    todos.value = todos.value.filter((t) => t.id !== id)
    errorMessage.value = ''
  } catch (error) {
    console.error('Errore:', error)
    errorMessage.value = 'Impossibile eliminare l\'attività'
  }
}

const completedCount = computed(() => {
  return todos.value.filter((t) => t.completed).length
})

const percentualeCompletate = computed(() => {
  if (todos.value.length === 0) return 0
  return Math.round((completedCount.value / todos.value.length) * 100)
})
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.app {
  display: grid;
  grid-template-columns: 280px 1fr;
  height: 100vh;
  background: #f5f5f5;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* SIDEBAR */
.sidebar {
  background: #ffffff;
  padding: 40px 24px;
  border-right: 1px solid #e5e5e5;
  display: flex;
  flex-direction: column;
  gap: 40px;
  overflow-y: auto;
}

.logo h1 {
  font-size: 28px;
  font-weight: 800;
  color: #1a1a1a;
  margin-bottom: 4px;
}

.logo p {
  font-size: 13px;
  color: #999;
  font-weight: 500;
}

/* STATS */
.stats {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.stat-value {
  font-size: 32px;
  font-weight: 800;
  color: #667eea;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 12px;
  color: #999;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* PROGRESS BAR */
.progress-bar {
  width: 100%;
  height: 6px;
  background: #e5e5e5;
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea, #764ba2);
  transition: width 0.3s ease;
  border-radius: 3px;
}

/* MAIN */
.main {
  display: flex;
  flex-direction: column;
  padding: 40px 48px;
  overflow-y: auto;
}

/* HEADER */
.header {
  margin-bottom: 40px;
}

.header h2 {
  font-size: 32px;
  font-weight: 800;
  color: #1a1a1a;
  margin-bottom: 8px;
}

.subtitle {
  font-size: 14px;
  color: #999;
}

/* INPUT FORM */
.input-form {
  display: flex;
  gap: 12px;
  margin-bottom: 32px;
}

.input {
  flex: 1;
  padding: 12px 16px;
  border: 1px solid #e5e5e5;
  border-radius: 8px;
  font-size: 14px;
  font-family: inherit;
  transition: all 0.2s ease;
}

.input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.1);
}

.btn {
  padding: 12px 24px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.btn:hover:not(:disabled) {
  background: #5568d3;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* LOADING */
.loading {
  text-align: center;
  padding: 40px;
  color: #999;
  font-size: 14px;
}

/* ERROR */
.error {
  background: #fee;
  color: #c33;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 14px;
  margin-bottom: 24px;
  border-left: 3px solid #c33;
}

/* EMPTY STATE */
.empty {
  text-align: center;
  padding: 60px 40px;
  color: #999;
}

.empty-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.empty p {
  font-size: 14px;
}

/* TODO LIST */
.todo-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  background: white;
  border: 1px solid #e5e5e5;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.todo-item:hover {
  border-color: #d0d0d0;
  background: #fafafa;
}

.todo-item.completed {
  opacity: 0.6;
}

.checkbox {
  width: 18px;
  height: 18px;
  border: 1px solid #d0d0d0;
  border-radius: 4px;
  cursor: pointer;
  accent-color: #667eea;
  flex-shrink: 0;
}

.title {
  flex: 1;
  font-size: 14px;
  color: #1a1a1a;
  word-break: break-word;
}

.todo-item.completed .title {
  text-decoration: line-through;
  color: #999;
}

.delete-btn {
  background: transparent;
  border: none;
  color: #ccc;
  font-size: 18px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
  transition: all 0.2s ease;
  flex-shrink: 0;
}

.delete-btn:hover {
  color: #e74c3c;
  background: #fee;
}

/* RESPONSIVE */
@media (max-width: 768px) {
  .app {
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr;
  }

  .sidebar {
    border-right: none;
    border-bottom: 1px solid #e5e5e5;
    flex-direction: row;
    gap: 24px;
    padding: 20px 24px;
    align-items: center;
  }

  .logo {
    flex-shrink: 0;
  }

  .logo h1 {
    font-size: 20px;
  }

  .stats {
    flex-direction: row;
    flex: 1;
    gap: 32px;
  }

  .progress-bar {
    width: 100px;
  }

  .main {
    padding: 24px;
  }

  .header h2 {
    font-size: 24px;
  }

  .input-form {
    flex-direction: column;
  }

  .btn {
    width: 100%;
  }
}

/* SCROLLBAR */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: #ddd;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #bbb;
}
</style>
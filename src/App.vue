<script setup>
import { ref, computed } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue 3', completed: true },
  { id: 2, text: 'Build a beautiful todo app', completed: false },
  { id: 3, text: 'Deploy to Netlify', completed: false }
])

const newTodo = ref('')
const filter = ref('all')
const editingId = ref(null)
const editText = ref('')

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value.trim(),
      completed: false
    })
    newTodo.value = ''
  }
}

const toggleTodo = (id) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.completed = !todo.completed
}

const deleteTodo = (id) => {
  todos.value = todos.value.filter(t => t.id !== id)
}

const startEdit = (todo) => {
  editingId.value = todo.id
  editText.value = todo.text
}

const saveEdit = () => {
  if (editText.value.trim() && editingId.value) {
    const todo = todos.value.find(t => t.id === editingId.value)
    if (todo) todo.text = editText.value.trim()
  }
  editingId.value = null
  editText.value = ''
}

const cancelEdit = () => {
  editingId.value = null
  editText.value = ''
}

const clearCompleted = () => {
  todos.value = todos.value.filter(t => !t.completed)
}

const filteredTodos = computed(() => {
  if (filter.value === 'active') return todos.value.filter(t => !t.completed)
  if (filter.value === 'completed') return todos.value.filter(t => t.completed)
  return todos.value
})

const stats = computed(() => {
  const total = todos.value.length
  const completed = todos.value.filter(t => t.completed).length
  const active = total - completed
  return { total, completed, active }
})

const allCompleted = computed(() => stats.value.total > 0 && stats.value.completed === stats.value.total)
</script>

<template>
  <div data-theme="night" class="min-h-screen bg-base-200">
    <div class="container mx-auto px-4 py-8 max-w-2xl">
      
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl md:text-5xl font-bold text-primary mb-2">✨ Super Todo</h1>
        <p class="text-base-content/60 text-lg">Stay organized, get things done</p>
      </div>

      <!-- Add Form -->
      <div class="card bg-base-100 shadow-xl mb-6">
        <div class="card-body p-4">
          <form @submit.prevent="addTodo" class="flex flex-col sm:flex-row gap-2">
            <input
              v-model="newTodo"
              type="text"
              placeholder="What needs to be done?"
              class="input input-bordered input-primary flex-1 w-full"
            />
            <button type="submit" class="btn btn-primary">
              <span class="hidden sm:inline">Add Task</span>
              <span class="sm:hidden text-xl">+</span>
            </button>
          </form>
        </div>
      </div>

      <!-- Filter Tabs -->
      <div class="tabs tabs-boxed bg-base-100 shadow-lg mb-6 justify-center">
        <button
          v-for="f in ['all', 'active', 'completed']"
          :key="f"
          @click="filter = f"
          :class="['tab', { 'tab-active': filter === f }]"
        >
          {{ f.charAt(0).toUpperCase() + f.slice(1) }}
          <span v-if="f === 'all'" class="badge badge-ghost ml-1">{{ stats.total }}</span>
          <span v-if="f === 'active'" class="badge badge-primary ml-1">{{ stats.active }}</span>
          <span v-if="f === 'completed'" class="badge badge-secondary ml-1">{{ stats.completed }}</span>
        </button>
      </div>

      <!-- Todo List -->
      <div class="card bg-base-100 shadow-xl">
        <div class="card-body p-0">
          
          <!-- Stats Bar -->
          <div v-if="stats.total > 0" class="px-4 py-3 bg-base-200/50 border-b border-base-300 flex flex-wrap justify-between items-center gap-2">
            <span class="text-sm text-base-content/60">
              {{ stats.active }} {{ stats.active === 1 ? 'task' : 'tasks' }} left
            </span>
            <div v-if="allCompleted" class="badge badge-success gap-1">
              🎉 All done!
            </div>
          </div>

          <!-- Items -->
          <ul class="menu menu-lg divide-y divide-base-200">
            <li v-for="todo in filteredTodos" :key="todo.id">
              
              <!-- Normal View -->
              <div v-if="editingId !== todo.id" class="flex items-center gap-3 px-4 py-3 hover:bg-base-200/50 transition-colors">
                <input
                  type="checkbox"
                  :checked="todo.completed"
                  @change="toggleTodo(todo.id)"
                  class="checkbox checkbox-primary checkbox-lg"
                />
                
                <span
                  @dblclick="startEdit(todo)"
                  :class="['flex-1 text-lg cursor-pointer', { 'line-through text-base-content/40': todo.completed }]"
                >
                  {{ todo.text }}
                </span>
                
                <div class="flex gap-1">
                  <button @click="startEdit(todo)" class="btn btn-ghost btn-sm btn-square">
                    <span class="text-lg">✏️</span>
                  </button>
                  <button @click="deleteTodo(todo.id)" class="btn btn-ghost btn-sm btn-square text-error">
                    <span class="text-lg">🗑️</span>
                  </button>
                </div>
              </div>

              <!-- Edit View -->
              <div v-else class="px-4 py-3 bg-base-200/50">
                <div class="flex flex-col sm:flex-row gap-2">
                  <input
                    v-model="editText"
                    type="text"
                    @keyup.enter="saveEdit"
                    @keyup.escape="cancelEdit"
                    class="input input-bordered input-primary flex-1"
                    autofocus
                  />
                  <div class="flex gap-1">
                    <button @click="saveEdit" class="btn btn-success btn-sm">Save</button>
                    <button @click="cancelEdit" class="btn btn-ghost btn-sm">Cancel</button>
                  </div>
                </div>
              </div>
            </li>
          </ul>

          <!-- Empty State -->
          <div v-if="filteredTodos.length === 0" class="p-8 text-center">
            <div class="text-6xl mb-4">
              {{ filter === 'all' ? '📝' : filter === 'active' ? '🎯' : '✅' }}
            </div>
            <p class="text-base-content/50 text-lg">
              {{ filter === 'all' ? 'No tasks yet' : filter === 'active' ? 'No active tasks' : 'No completed tasks' }}
            </p>
            <p class="text-base-content/30 text-sm mt-2">
              {{ filter === 'all' ? 'Add one above!' : 'Try a different filter' }}
            </p>
          </div>

          <!-- Footer -->
          <div v-if="stats.completed > 0" class="px-4 py-3 bg-base-200/50 border-t border-base-300">
            <button @click="clearCompleted" class="btn btn-error btn-sm btn-outline gap-2">
              <span>🗑️</span>
              Clear {{ stats.completed }} completed {{ stats.completed === 1 ? 'task' : 'tasks' }}
            </button>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <div class="text-center mt-8 space-y-1">
        <p class="text-base-content/40 text-sm">
          Double-click to edit • Built with Vue 3 + Tailwind CSS + daisyUI
        </p>
        <p class="text-base-content/30 text-xs">
          Powered by ☁️ Netlify • Deployed with ❤️
        </p>
      </div>

    </div>
  </div>
</template>

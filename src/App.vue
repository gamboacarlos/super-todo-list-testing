<script setup>
import { ref, computed } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue 3', completed: true },
  { id: 2, text: 'Build a todo app', completed: false },
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
  <div class="min-h-screen py-12 px-4">
    <div class="max-w-xl mx-auto">
      <!-- Header -->
      <div class="text-center mb-10">
        <h1 class="text-6xl font-bold text-white mb-3 tracking-tight">✨ Super Todo</h1>
        <p class="text-white/60 text-lg">Stay organized, get things done</p>
      </div>

      <!-- Add Todo Form -->
      <div class="bg-white/10 backdrop-blur-lg rounded-3xl shadow-2xl p-2 mb-8 border border-white/20">
        <form @submit.prevent="addTodo" class="flex gap-2">
          <input
            v-model="newTodo"
            type="text"
            placeholder="Add a new task..."
            class="flex-1 px-6 py-4 text-lg bg-transparent text-white placeholder-white/50 focus:outline-none rounded-2xl"
          />
          <button
            type="submit"
            class="px-8 py-4 bg-gradient-to-r from-emerald-500 to-teal-500 text-white font-bold rounded-2xl hover:from-emerald-600 hover:to-teal-600 transition-all duration-200 shadow-lg hover:shadow-xl transform hover:scale-105"
          >
            <span class="text-xl">+</span>
          </button>
        </form>
      </div>

      <!-- Filter Pills -->
      <div class="flex justify-center gap-3 mb-6">
        <button
          v-for="f in ['all', 'active', 'completed']"
          :key="f"
          @click="filter = f"
          :class="[
            'px-6 py-2 rounded-full font-medium transition-all duration-200',
            filter === f 
              ? 'bg-white text-purple-600 shadow-lg' 
              : 'text-white/70 hover:text-white hover:bg-white/10'
          ]"
        >
          {{ f.charAt(0).toUpperCase() + f.slice(1) }}
        </button>
      </div>

      <!-- Todo List -->
      <div class="bg-white/10 backdrop-blur-lg rounded-3xl shadow-2xl overflow-hidden border border-white/20">
        <!-- Header -->
        <div v-if="stats.total > 0" class="px-6 py-4 bg-white/5 border-b border-white/10 flex justify-between items-center">
          <span class="text-white/70 text-sm">
            {{ stats.active }} {{ stats.active === 1 ? 'task' : 'tasks' }} remaining
          </span>
          <button
            v-if="allCompleted && stats.total > 0"
            class="text-emerald-400 text-sm font-medium hover:text-emerald-300 transition"
          >
            🎉 All done!
          </button>
        </div>

        <!-- Items -->
        <ul v-if="filteredTodos.length">
          <li
            v-for="todo in filteredTodos"
            :key="todo.id"
            class="group px-6 py-5 border-b border-white/10 last:border-b-0 hover:bg-white/5 transition-all duration-150"
          >
            <!-- Normal View -->
            <div v-if="editingId !== todo.id" class="flex items-center gap-4">
              <button
                @click="toggleTodo(todo.id)"
                :class="[
                  'w-7 h-7 rounded-full border-2 flex items-center justify-center transition-all duration-200',
                  todo.completed 
                    ? 'bg-emerald-500 border-emerald-500' 
                    : 'border-white/40 hover:border-white/60'
                ]"
              >
                <span v-if="todo.completed" class="text-white text-sm">✓</span>
              </button>
              
              <span
                @dblclick="startEdit(todo)"
                :class="[
                  'flex-1 text-lg cursor-pointer',
                  todo.completed ? 'line-through text-white/40' : 'text-white'
                ]"
              >
                {{ todo.text }}
              </span>
              
              <button
                @click="startEdit(todo)"
                class="text-white/30 hover:text-white/60 transition opacity-0 group-hover:opacity-100"
              >
                ✎
              </button>
              
              <button
                @click="deleteTodo(todo.id)"
                class="text-white/30 hover:text-red-400 transition opacity-0 group-hover:opacity-100"
              >
                ✕
              </button>
            </div>

            <!-- Edit View -->
            <div v-else class="flex items-center gap-2">
              <input
                v-model="editText"
                type="text"
                @keyup.enter="saveEdit"
                @keyup.escape="cancelEdit"
                class="flex-1 px-4 py-2 text-lg bg-white/10 text-white rounded-xl border border-white/20 focus:border-white/40 focus:outline-none"
                autofocus
              />
              <button
                @click="saveEdit"
                class="px-4 py-2 bg-emerald-500 text-white rounded-xl hover:bg-emerald-600 transition"
              >
                Save
              </button>
              <button
                @click="cancelEdit"
                class="px-4 py-2 text-white/60 hover:text-white transition"
              >
                Cancel
              </button>
            </div>
          </li>
        </ul>

        <!-- Empty State -->
        <div v-else class="px-6 py-16 text-center">
          <div class="text-6xl mb-4">
            {{ filter === 'all' ? '📝' : filter === 'active' ? '🎯' : '✅' }}
          </div>
          <p class="text-white/50 text-lg">
            {{ filter === 'all' ? 'No tasks yet' : filter === 'active' ? 'No active tasks' : 'No completed tasks' }}
          </p>
          <p class="text-white/30 text-sm mt-2">
            {{ filter === 'all' ? 'Add one above!' : 'Try a different filter' }}
          </p>
        </div>

        <!-- Footer -->
        <div v-if="stats.completed > 0" class="px-6 py-4 bg-white/5 border-t border-white/10">
          <button
            @click="clearCompleted"
            class="text-red-400/70 hover:text-red-400 text-sm font-medium transition"
          >
            🗑️ Clear {{ stats.completed }} completed {{ stats.completed === 1 ? 'task' : 'tasks' }}
          </button>
        </div>
      </div>

      <!-- Footer -->
      <div class="text-center mt-8 space-y-2">
        <p class="text-white/40 text-sm">
          Double-click to edit • Built with Vue 3 + Tailwind CSS
        </p>
        <p class="text-white/30 text-xs">
          Powered by ☁️ Netlify
        </p>
      </div>
    </div>
  </div>
</template>

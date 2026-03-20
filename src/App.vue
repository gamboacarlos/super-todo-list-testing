<script setup>
import { ref, computed } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue 3', completed: true },
  { id: 2, text: 'Build a todo app', completed: false },
  { id: 3, text: 'Deploy to Netlify', completed: false }
])

const newTodo = ref('')
const filter = ref('all')

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
</script>

<template>
  <div class="min-h-screen py-10 px-4">
    <div class="max-w-lg mx-auto">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold text-white mb-2">🚀 Super Todo List</h1>
        <p class="text-white/70">Built with Vue 3 + Tailwind CSS</p>
      </div>

      <!-- Add Todo Form -->
      <div class="bg-white rounded-2xl shadow-xl p-6 mb-6">
        <form @submit.prevent="addTodo" class="flex gap-2">
          <input
            v-model="newTodo"
            type="text"
            placeholder="What needs to be done?"
            class="flex-1 px-4 py-3 rounded-xl border-2 border-gray-200 focus:border-purple-500 focus:outline-none transition"
          />
          <button
            type="submit"
            class="px-6 py-3 bg-purple-600 text-white rounded-xl font-semibold hover:bg-purple-700 transition"
          >
            Add
          </button>
        </form>
      </div>

      <!-- Filter Buttons -->
      <div class="flex justify-center gap-2 mb-4">
        <button
          v-for="f in ['all', 'active', 'completed']"
          :key="f"
          @click="filter = f"
          :class="[
            'px-4 py-2 rounded-lg font-medium transition',
            filter === f ? 'bg-white text-purple-600' : 'text-white/70 hover:text-white'
          ]"
        >
          {{ f.charAt(0).toUpperCase() + f.slice(1) }}
        </button>
      </div>

      <!-- Todo List -->
      <div class="bg-white rounded-2xl shadow-xl overflow-hidden">
        <ul v-if="filteredTodos.length">
          <li
            v-for="todo in filteredTodos"
            :key="todo.id"
            class="flex items-center gap-4 px-6 py-4 border-b border-gray-100 last:border-b-0 hover:bg-gray-50 transition"
          >
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleTodo(todo.id)"
              class="w-5 h-5 rounded border-gray-300 text-purple-600 focus:ring-purple-500"
            />
            <span
              :class="[
                'flex-1 text-lg',
                todo.completed ? 'line-through text-gray-400' : 'text-gray-800'
              ]"
            >
              {{ todo.text }}
            </span>
            <button
              @click="deleteTodo(todo.id)"
              class="text-red-400 hover:text-red-600 transition"
            >
              ✕
            </button>
          </li>
        </ul>
        <div v-else class="px-6 py-8 text-center text-gray-400">
          <p v-if="filter === 'all'">No todos yet! Add one above.</p>
          <p v-else>No {{ filter }} todos.</p>
        </div>

        <!-- Stats Footer -->
        <div class="px-6 py-4 bg-gray-50 flex justify-between items-center text-sm text-gray-500">
          <span>{{ stats.active }} items left</span>
          <button
            v-if="stats.completed > 0"
            @click="clearCompleted"
            class="text-red-500 hover:text-red-600 transition"
          >
            Clear completed ({{ stats.completed }})
          </button>
        </div>
      </div>

      <!-- Footer -->
      <p class="text-center text-white/50 mt-6 text-sm">
        Made with ❤️ using Vue 3 + Tailwind CSS • Deployed on Netlify
      </p>
    </div>
  </div>
</template>

<template>
  <div>
    <h2 class="text-2xl font-bold mb-4">My Account</h2>
    <div v-if="user" class="mb-6">
      <p><strong>Name:</strong> {{ user.name }}</p>
      <p><strong>Primary Email:</strong> {{ user.email }}</p>
    </div>
    
    <h3 class="text-xl font-bold mb-2">All Emails</h3>
    <ul v-if="user && user.email" class="mb-4">
      <li class="mb-2">
        {{ user.email }} (Primary)
      </li>
      <li v-for="email in user.additionalEmails" :key="email" class="mb-2">
        {{ email }}
        <button @click="removeEmail(email)" class="ml-2 text-red-500 hover:text-red-700">Remove</button>
      </li>
    </ul>
    <p v-else class="mb-4">No emails found.</p>
    
    <form @submit.prevent="addEmail" class="mb-6">
      <div class="flex items-center">
        <input v-model="newEmail" type="email" placeholder="Add new email" required class="shadow appearance-none border rounded py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline mr-2">
        <button type="submit" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
          Add Email
        </button>
      </div>
    </form>
    
    <div v-if="statusMessage" :class="['mb-4 p-2 rounded', statusMessage.type === 'error' ? 'bg-red-100 text-red-700' : 'bg-green-100 text-green-700']">
      {{ statusMessage.text }}
    </div>
    
    <button @click="logout" class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
      Logout
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useAuthStore } from '~/stores/auth'

const authStore = useAuthStore()
const user = ref(null)
const newEmail = ref('')
const statusMessage = ref(null)

onMounted(async () => {
  if (authStore.isAuthenticated) {
    try {
      user.value = await authStore.getUser()
    } catch (error) {
      setStatusMessage('Failed to fetch user data', 'error')
    }
  } else {
    navigateTo('/login')
  }
})

const addEmail = async () => {
  try {
    await authStore.addEmail(newEmail.value)
    user.value = await authStore.getUser() // Refresh user data
    newEmail.value = ''
    setStatusMessage('Email added successfully', 'success')
  } catch (error) {
    setStatusMessage(error.message || 'Failed to add email', 'error')
  }
}

const removeEmail = async (email) => {
  try {
    await authStore.removeEmail(email)
    user.value = await authStore.getUser() // Refresh user data
    setStatusMessage('Email removed successfully', 'success')
  } catch (error) {
    setStatusMessage(error.message || 'Failed to remove email', 'error')
  }
}

const logout = () => {
  authStore.logout()
  navigateTo('/')
}

const setStatusMessage = (text, type = 'success') => {
  statusMessage.value = { text, type }
  setTimeout(() => {
    statusMessage.value = null
  }, 5000)
}
</script>
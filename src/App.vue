<template>
  <div style="padding: 40px">
    <h1>Frontend OK</h1>

    <button @click="callBackend">
      Call Backend
    </button>

    <p v-if="message">
      Backend Response: {{ message }}
    </p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import api from './api'

const message = ref('')

const callBackend = async () => {
  const res = await api.get('/')
  message.value = res.data
}
</script>

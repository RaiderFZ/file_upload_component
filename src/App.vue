<script setup lang="ts">
import { ref } from 'vue'
import FileUploadComponents from './components/FileUploadComponents.vue'

const file = ref<File | null>(null)
const fileError = ref('')
const isLoading = ref(false)

function handleFiles(files: FileList | null) {
  fileError.value = ''
  file.value = null

  if (!files || files.length === 0) return

  const selected = files[0]

  const maxSizeMB = 2
  const maxSizeBytes = maxSizeMB * 1024 * 1024
  const allowedTypes = ['image/jpeg', 'image/png', 'image/webp']

  if (!allowedTypes.includes(selected.type)) {
    fileError.value = 'Неверный тип файла. Допустимы: JPG, PNG, WEBP.'
    return
  }

  if (selected.size > maxSizeBytes) {
    fileError.value = `Файл слишком большой. Максимум ${maxSizeMB}MB.`
    return
  }

  file.value = selected
  isLoading.value = true

  // Симуляция загрузки
  setTimeout(() => {
    isLoading.value = false
  }, 2000)
}
</script>

<template>
  <div class="container">
    <FileUploadComponents 
      label="Фотография"
      accept="image/*"
      :file="file"
      :error="fileError"
      :loading="isLoading"
      hint="Максимум 2MB. Допустимые форматы: JPG, PNG, WEBP."
      @change="handleFiles"
    />
  </div>
</template>

<style scoped>
.container {
  max-width: 400px;
  margin: 40px auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}
</style>

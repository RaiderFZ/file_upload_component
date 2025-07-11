<script setup lang="ts">
import { ref, nextTick } from 'vue'
import FileUploadComponents from './components/FileUploadComponents.vue'

const file = ref<File | null>(null)
const fileError = ref('')
const isLoading = ref(false)

const handleFiles = (newFile: File | null) => {
  fileError.value = '';
  file.value = newFile;

  if (!newFile) {
    isLoading.value = false;
    return;
  }

  isLoading.value = true;
  setTimeout(() => {
    isLoading.value = false;
  }, 2000);
}

const handleError= async (message: string) => {
  fileError.value = ''
  await nextTick() ;
  fileError.value = message;
  isLoading.value = false;
}
</script>

<template>
  <div class="container">
    <FileUploadComponents 
      label="Label"
      accept="image/*"
      :file="file"
      :error="fileError"
      :loading="isLoading"
      :allowedTypes="['image/jpeg', 'image/png', 'image/webp']"
      :maxSize="2 * 1024 * 1024"
      hint="Hint"
      @change="handleFiles"
      @error="handleError"
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

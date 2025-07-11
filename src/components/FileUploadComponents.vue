<script setup lang="ts">
import { ref, computed } from 'vue'

interface FileUploadProps {
    file?: File | null
    accept?: string
    multiple?: boolean
    label: string
    error?: string
    hint?: string
    loading?: boolean
    disabled?: boolean
}

const props = defineProps<FileUploadProps>()

const emit = defineEmits<{
    (e: 'change', files: FileList | null): void
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const hasFile = computed(() => !!props.file)

const selectedFileText = computed(() => {
    if (!props.file) return 'Файл не выбран'
    return props.file.name
})

function onFileChange(event: Event): void {
    const target = event.target as HTMLInputElement
    emit('change', target.files)
}

function clearFile(): void {
    if (fileInput.value) fileInput.value.value = ''
    emit('change', null)
}

function handleButtonClick(): void {
    if (props.disabled || props.loading) return

    if (hasFile.value) {
        clearFile()
    } else {
        fileInput.value?.click()
    }
}
</script>

<template>
    <div class="file-upload">
        <label class="upload-label">{{ props.label }}</label>

        <div class="upload-row">
        <input
            ref="fileInput"
            type="file"
            class="hidden-input"
            :accept="props.accept"
            :multiple="props.multiple"
            :disabled="props.disabled"
            @change="onFileChange"
        />

        <button
            type="button"
            class="upload-button"
            @click="handleButtonClick"
            :disabled="props.loading || props.disabled"
        >
            {{ hasFile ? 'Отменить' : 'Выбрать файл' }}
        </button>

        <div
            class="file-status"
            @click="hasFile && !props.disabled && !props.loading && clearFile()"
        >
            <span v-if="props.loading" class="spinner" />
            <span class="filename">{{ selectedFileText }}</span>
        </div>
        </div>

        <div v-if="props.error" class="error-message">{{ props.error }}</div>
        <div v-else-if="props.hint" class="hint-text">{{ props.hint }}</div>
    </div>
</template>

<style scoped>
.file-upload {
    display: inline-block;
    position: relative;
    width: 100%;
}
.upload-label {
    display: block;
    margin-bottom: 12px;
}
.upload-row {
    display: flex;
    align-items: center;
    gap: 12px;
}
.hidden-input {
    display: none;
}
.upload-button {

    padding: 12px 20px;
    font-size: 14px;
    font-weight: 400;
    border: 1px solid #E5E7EB;
    background-color: #fff;
    border-radius: 12px;
    transition: background-color 0.2s;
    cursor: pointer;
}
.upload-button:hover {
    background-color: #eaeaea;
}
.upload-button:disabled {
    background-color: #f5f5f5;
    cursor: not-allowed;
    color: #aaa;
}
.file-status {
    font-size: 13px;
    color: #555;
    cursor: pointer;
}
.file-status.disabled {
    cursor: default;
    color: #aaa;
}
.spinner {
    margin-right: 8px;
    width: 14px;
    height: 14px;
    border: 2px solid #ccc;
    border-top: 2px solid #333;
    border-radius: 50%;
    display: inline-block;
    animation: spin 1s linear infinite;
}
@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}
.error-message {
    color: red;
    font-size: 12px;
    margin-top: 8px;
}
.hint-text {
    font-size: 12px;
    color: #888;
    margin-top: 4px;
}
</style>

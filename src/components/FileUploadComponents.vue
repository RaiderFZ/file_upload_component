<script setup lang="ts">
import { ref, computed } from 'vue';

interface FileUploadProps {
    file?: File | null
    accept?: string
    label: string
    error?: string
    hint?: string
    loading?: boolean
    disabled?: boolean
    allowedTypes?: string[]
    maxSize?: number
}

const props = defineProps<FileUploadProps>()

const emit = defineEmits<{
    (e: 'change', file: File | null): void
    (e: 'error', message: string): void
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const hasFile = computed(() => !!props.file)

const selectedFileText = computed(() => {
    if (!props.file) return 'Файл не выбран'
    return props.file.name
})

const validateFile = (file: File): { valid: boolean; message?: string } => {
  if (props.allowedTypes && props.allowedTypes.length > 0 && !props.allowedTypes.includes(file.type)) {
    return { 
        valid: false, 
        message: 'Неверный тип файла. Допустимы: JPG, PNG, WEBP.' 
    };
  };
  if (props.maxSize && file.size > props.maxSize) {
    return { 
        valid: false, 
        message: `Файл слишком большой. Максимум ${props.maxSize / (1024 * 1024)}MB.` 
    }
  };
  return { valid: true };
}

const onFileChange = (event: Event): void => {
    const target = event.target as HTMLInputElement;
    const files = target.files;
    if (!files || files.length === 0) {
        emit('change', null);
        return;
    };

    const file = files[0];

    const validation = validateFile(file);
    if (!validation.valid) {
        emit('error', validation.message || 'Ошибка валидации');
        clearFile();
        return;
    };

    emit('change', file);
};

const clearFile = (): void => {
    if (fileInput.value) fileInput.value.value = '';
    emit('change', null);
};

const handleButtonClick = (): void => {
    if (props.disabled || props.loading) return;
    if (hasFile.value) {
        clearFile();
    } else {
        fileInput.value?.click();
    };
};
</script>

<template>
    <div class="file-upload">
         <label class="upload-label " :for="props.disabled ? undefined : 'file-input'">{{ props.label }}</label>

        <div class="upload-row">
            <input
                id="file-input"
                ref="fileInput"
                type="file"
                class="hidden-input"
                :accept="props.accept"
                :disabled="props.disabled"
                @change="onFileChange"
                :aria-describedby="props.error ? 'file-error' : (props.hint ? 'file-hint' : undefined)"
            />

            <div class="visually-hidden" aria-live="polite">
                {{ hasFile ? 'Файл выбран: ' + selectedFileText : '' }}
            </div>

            <button
                type="button"
                class="upload-button"
                @click="handleButtonClick"
                :disabled="props.loading || props.disabled"
                :aria-label="hasFile ? 'Очистить файл' : 'Выбрать файл'"
            >
                {{ hasFile ? 'Отменить' : 'Выбрать файл' }}
            </button>

            <div
                class="file-status"
                role="button"
                tabindex="0"
                @click="hasFile && !props.disabled && !props.loading && clearFile()"
                @keydown.enter.space="hasFile && !props.disabled && !props.loading && clearFile()"
                :aria-label="hasFile ? 'Очистить выбранный файл' : undefined"
            >
                <span v-if="props.loading" class="spinner" />
                <span 
                    class="filename"
                    :class="{ 'filename--active': hasFile && !props.loading }" 
                >
                    {{ selectedFileText }}
                </span>
            </div>
        </div>

        <div 
            v-if="props.error" 
            class="error-message"
            id="file-error"  
            role="alert"
        >
            {{ props.error }}
        </div>
        <div 
            v-else-if="props.hint"
            id="file-hint" 
            class="hint-text"
        >
            {{ props.hint }}
        </div>
    </div>
</template>

<style scoped lang="scss">
@use '../styles/mixin' as *;
@use '../styles/variables' as *;

.spinner {
    width: 16px;
    height: 16px;
    border: 2px solid transparent;
    border-top-color: $web-primary-500-base; 
    border-radius: 50%;
    animation: spin 1s linear infinite;
    display: inline-block;
}
@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

.visually-hidden {
    position: absolute !important;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
    border: 0;
}

.file-upload {
    @include text-md;
    display: inline-block;
    position: relative;
    width: 100%;
}
.upload-label {
    color: $web-neutral-900;
    display: block;
    margin-bottom: 12px;
}
.upload-row {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 8px;
}
.hidden-input {
    display: none;
}
.upload-button {
    color: $web-neutral-700;
    padding: 12px 20px;
    font-size: 14px;
    font-weight: 400;
    border:  1px solid $web-neutral-200;
    background-color: $background-white;
    border-radius: 12px;
    transition: background-color 0.2s;
    cursor: pointer;
    box-shadow: 0px 3px 7px -3px rgba(0, 0, 0, 0.08), 0px 4px 8px 0px rgba(0, 0, 0, 0.02);
}
.upload-button:hover {
    border: 1px solid $web-neutral-300;
    background-color: #F3F4F6;
    box-shadow: 0px 0px 0px 2px rgba(107, 114, 128, 0.10)
}
.upload-button:focus {
    box-shadow: 0px 0px 0px 2px rgba(107, 114, 128, 0.10);
}
.upload-button:disabled {
    color: $web-neutral-400;
    border: 1px solid $web-neutral-200;
    box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.02);
    cursor: default;
}
.upload-button:active {
    border: 1px solid $web-neutral-300;
    background: $web-neutral-200;
}
.file-status {
    color: $web-neutral-400;
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 13px;
    
    cursor: pointer;
}

.error-message {
    color:  $web-error-500-base;
    font-size: 12px;
    margin-top: 8px;
}
.hint-text {
    color: $web-neutral-500-base;
    font-size: 12px;
    margin-top: 4px;
}
.filename {
    @include text-md;
    color: $web-neutral-400;
}
.filename--active {
    color: $web-neutral-500-base;
}
</style>

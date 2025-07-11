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

const props = withDefaults(defineProps<FileUploadProps>(), {
  accept: '',
  file: null,
  error: '',
  hint: '',
  loading: false,
  disabled: false,
  allowedTypes: () => [],
  maxSize: 0
})

const emit = defineEmits<{
    (e: 'change', file: File | null): void
    (e: 'error', message: string): void
    (e: 'cancel'): void
}>()

const fileInput = ref<HTMLInputElement | null>(null)

const hasFile = computed(() => !!props.file)

const selectedFileText = computed(() => {
    if (!props.file) return 'Файл не выбран'
    return props.file.name
})

const validateFile = (file: File): { valid: boolean; message?: string } => {
  if (props.allowedTypes && props.allowedTypes.length > 0 && !props.allowedTypes.includes(file.type)) {
    const typesList = props.allowedTypes.map(type => type.split('/')[1].toUpperCase()).join(', ')
    return { 
        valid: false, 
        message: `Недопустимый тип файла. Разрешены: ${typesList}.`
    };
  }

  if (props.maxSize && file.size > props.maxSize) {
    const sizeMb = (props.maxSize / (1024 * 1024)).toFixed(2);
    return { 
        valid: false, 
        message: `Файл слишком большой. Максимальный размер: ${sizeMb} MB.`
    };
  }

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

const buttonLabel = computed(() => {
  if (props.loading) return 'Отменить'
  if (hasFile.value) return 'Удалить'
  return 'Выбрать файл'
})

const handleButtonClick = (): void => {
    if (props.disabled) return;

    if (props.loading) {
        emit('cancel');
        return;
    }

    if (hasFile.value) {
        clearFile();
        return;
    }

    fileInput.value?.click();
};

const clearFile = (): void => {
    if (fileInput.value) fileInput.value.value = '';
    emit('change', null);
};


</script>

<template>
    <div class="file-upload text-md">
         <label class="upload-label text-sm" :for="props.disabled ? undefined : 'file-input'">{{ props.label }}</label>

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
                :disabled="props.disabled"
                :aria-label="buttonLabel"
            >
                {{ buttonLabel }}
            </button>

            <div
                class="file-status"
                role="presentation"
                tabindex="0"
                @click="!props.disabled && !props.loading && fileInput?.click()"
                @keydown.enter.space="!props.disabled && !props.loading && fileInput?.click()"
                :aria-label="hasFile ? 'Файл выбран: ' + selectedFileText : 'Нажмите, чтобы выбрать файл'"
                >
                <span v-if="props.loading" class="spinner" />
                <span 
                    class="filename text-md"
                    :class="{ 'filename--active': hasFile && !props.loading }" 
                >
                    {{ selectedFileText }}
                </span>
            </div>
        </div>

        <div 
            v-if="props.error" 
            class="error-message"
            id="file-error text-sm"  
            role="alert"
        >
            {{ props.error }}
        </div>
        <div 
            v-else-if="props.hint"
            id="file-hint" 
            class="hint-text text-sm"
        >
            {{ props.hint }}
        </div>
    </div>
</template>

<style scoped>
.spinner {
    width: 16px;
    height: 16px;
    border: 2px solid transparent;
    border-top-color: var(--web-primary-500-base); 
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

.text-md {
    font-family: 'Inter';
    font-size: 14px;
    font-style: normal;
    font-weight: 400;
    line-height: 16px;
    letter-spacing: -0.084px;
}
.text-sm {
    font-family: 'Inter';
    font-size: 13px;
    font-style: normal;
    font-weight: 500;
    line-height: 120%;
}

.file-upload {
    display: inline-block;
    position: relative;
    width: 100%;
}
.upload-label {
    color: var(--web-neutral-900);
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
    color: var(--web-neutral-700);
    padding: 12px 20px;
    font-size: 14px;
    font-weight: 400;
    border:  1px solid var(--web-neutral-200);
    background-color: var(--background-white);
    border-radius: 12px;
    transition: background-color 0.2s;
    cursor: pointer;
    box-shadow: 0px 3px 7px -3px rgba(0, 0, 0, 0.08), 0px 4px 8px 0px rgba(0, 0, 0, 0.02);
}
.upload-button:hover {
    border: 1px solid var(--web-neutral-300);
    background-color: #F3F4F6;
    box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.02);
}
.upload-button:focus {
    outline: none;
    box-shadow: 0px 0px 0px 2px rgba(107, 114, 128, 0.10);
}
.upload-button:disabled {
    color: var(--web-neutral-400);
    border: 1px solid var(--web-neutral-200);
    box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.02);
    cursor: default;
}
.upload-button:active {
    box-shadow: none;
    border: 1px solid var(--web-neutral-300);
    background: var(--web-neutral-200);
}
.file-status {
    color: var(--web-neutral-400);
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 13px;
    user-select: none;
}

.file-status:focus {
    outline: none;
}
.error-message {
    color:  var(--web-error-500-base);
    margin-top: 8px;
}
.hint-text {
    color: var(--web-neutral-500-base);
    margin-top: 4px;
}
.filename {
    color: var(--web-neutral-400);
}

.filename--active {
    color: var(--web-neutral-500-base);
}
</style>

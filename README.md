# File Upload Component

Универсальный, семантически корректный и доступный компонент загрузки файлов, разработанный на Vue 3 с использованием TypeScript.

## Установка

1. Клонируйте репозиторий:
   ```bash
   git clone <repository-url>
   ```
2. Установите зависимости:
   ```bash
   npm install
   ```
3. Запустите проект:
   ```bash
   npm run dev
   ```

## Использование

Компонент можно использовать в любом Vue 3 проекте. Пример интеграции:

```vue
<template>
  <div class="container">
    <FileUploadComponents
      label="Upload File"
      accept="image/*"
      :allowedTypes="['image/jpeg', 'image/png', 'image/webp']"
      :maxSize="2 * 1024 * 1024"
      hint="Only images up to 2MB"
      @change="handleFileChange"
      @error="handleError"
    />
  </div>
</template>

<script setup lang="ts">
import FileUploadComponents from './components/FileUploadComponents.vue';
import { ref } from 'vue';

const file = ref<File | null>(null);
const fileError = ref('');

const handleFileChange = (newFile: File | null) => {
  file.value = newFile;
  console.log('Selected file:', newFile);
};

const handleError = (message: string) => {
  fileError.value = message;
  console.error('Error:', message);
};
</script>

<style scoped>
.container {
  max-width: 400px;
  margin: 40px auto;
  padding: 16px;
}
</style>
```

## Props

| Prop          | Type              | Default | Description                              |
|---------------|-------------------|---------|------------------------------------------|
| `label`       | String            | -       | Текст метки для поля загрузки            |
| `accept`      | String            | -       | MIME-типы, принимаемые полем `<input>`   |
| `file`        | File \| null      | null    | Выбранный файл                           |
| `error`       | String            | -       | Сообщение об ошибке                      |
| `hint`        | String            | -       | Подсказка для пользователя               |
| `loading`     | Boolean           | false   | Показать индикатор загрузки              |
| `disabled`    | Boolean           | false   | Отключить компонент                      |
| `allowedTypes`| string[]          | -       | Массив допустимых MIME-типов             |
| `maxSize`     | Number            | -       | Максимальный размер файла (в байтах)     |

## Events

| Event   | Parameters         | Description                          |
|---------|--------------------|--------------------------------------|
| `change`| `file: File \| null`| Срабатывает при выборе или очистке файла |
| `error` | `message: string`  | Срабатывает при ошибке валидации     |

## Доступность (Accessibility)

- **Клавиатурная навигация**: Полная поддержка навигации через `Tab`, `Enter` и `Space` для выбора и очистки файла.

- **Экранные читалки**: Совместимость с NVDA, VoiceOver и другими благодаря использованию ARIA-атрибутов (`aria-label`, `aria-describedby`, `aria-live`, `role="alert"`).

- **Семантика**: Используются корректные HTML-элементы (`<label>`, `<input type="file">`, `<button>`) для логичной структуры DOM.

## Требования

- Vue 3
- TypeScript

## Структура проекта

```
├── src/
│   ├── components/
│   │   └── FileUploadComponents.vue  # Компонент загрузки файлов
│   └── App.vue                      # Основной компонент с примером
├── README.md                        # Документация
├── package.json                     # Зависимости и скрипты
└── vite.config.ts                   # Конфигурация Vite
```
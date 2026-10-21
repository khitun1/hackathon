<!-- src/components/UploadPanel.vue -->
<template>
  <div class="card bg-white p-4 rounded-xl shadow-sm mb-4">

    <!-- 🌟 СЕКЦИЯ 1: Компактный ввод ID заёмщика -->
    <div class="flex items-center justify-between w-fit mb-4 pb-3 border-b border-slate-200">
      <div class="flex items-center gap-3 flex-1 max-w-3xl">
        <label class="text-sm font-bold text-slate-700 whitespace-nowrap">Поиск по ID:</label>

        <InputText
            v-model="localId"
            placeholder="Введите ID..."
            size="small"
            class="w-full sm:w-fit text-center"
            :disabled="disabled"
            @keyup.enter="handleSearch"
        />
      </div>

      <Button
          label="Проверить риски"
          icon="pi pi-bolt"
          severity="primary"
          size="small"
          :disabled="disabled || !localId.trim()"
          @click="handleSearch"
          class="ml-3 whitespace-nowrap"
      />
    </div>

    <!-- 🌟 СЕКЦИЯ 2: FileUpload с программным сбросом (ref="fileUploadRef") -->
    <FileUpload
        ref="fileUploadRef"
        name="demo[]"
        accept=".csv"
        :maxFileSize="10000000"
        :disabled="disabled"
        customUpload
        @select="onSelectedFiles"
    >
      <template #header="{ chooseCallback }">
        <div class="flex flex-wrap justify-between items-center flex-1 gap-4">
          <div class="flex gap-2">
            <Button
                @click="chooseCallback()"
                icon="pi pi-file"
                rounded
                variant="outlined"
                severity="secondary"
                :disabled="disabled"
            />
          </div>
        </div>
      </template>

      <template #empty>
        <div class="flex flex-col items-center justify-center py-8 border border-dashed border-slate-300 rounded-lg bg-slate-50/50">
          <i class="pi pi-cloud-upload p-4 text-4xl text-slate-400" />
          <p class="mt-3 font-medium text-sm text-slate-500 text-center px-4">
            Перетащите .csv файл сюда для запуска скоринга ИИ
          </p>
        </div>
      </template>
    </FileUpload>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Button from "primevue/button";
import InputText from "primevue/inputtext";
import FileUpload from "primevue/fileupload";

defineProps({
  disabled: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['analyze']);

const localId = ref('');

// 🌟 Ссылка на компонент FileUpload для вызова метода .clear()
const fileUploadRef = ref(null);

// Поиск по конкретному ID заёмщика
const handleSearch = () => {
  if (!localId.value.trim()) return;
  emit('analyze', { type: 'id', value: localId.value.trim() });
};

// Срабатывает при выборе файла через проводник или при Drag-and-Drop
const onSelectedFiles = (event) => {
  // 1. Отправляем выбранный файл(ы) наружу в родительский компонент
  emit("analyze", { type: 'file', files: event.files });

  // 2. Мгновенно сбрасываем внутренний список файлов в PrimeVue,
  // чтобы они не рендерились в зоне drag-drop и снова показался слот #empty
  if (fileUploadRef.value) {
    fileUploadRef.value.clear();
  }
};
</script>

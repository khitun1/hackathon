<template>
  <div class="ml-4 mt-4">
    <FileUploadComponent @analyze="startLoading"/>

    <Dialog
        v-model:visible="isAnalyzing"
        modal
        :showHeader="false"
        :closable="false"
        unstyled
        :pt="{
          root: { class: 'bg-transparent border-none shadow-none focus:outline-none' },
          mask: { class: 'flex items-center justify-center bg-black/50 backdrop-blur-sm' }
        }"
    >
      <div class="flex flex-col items-center justify-center gap-5 select-none">
        <ProgressSpinner
            style="width: 96px; height: 96px"
            strokeWidth="3.5"
            animationDuration=".7s"
            class="!text-blue-500"
        />
        <span class="text-xl font-bold text-white tracking-wider drop-shadow-lg animate-pulse">
          Анализ ИИ...
        </span>
      </div>
    </Dialog>
  </div>

  <!-- Кнопку выносим ОТДЕЛЬНО, чтобы она не попадала в финальный PDF -->
  <div v-if="stateData" class="max-w-7xl mx-auto px-4 mt-4 flex justify-end">
    <Button @click="exportToPDF" severity="success" icon="pi pi-download" label="Скачать отчет в PDF" />
  </div>

  <!-- Контейнер для экспорта (теперь без кнопки внутри) -->
  <div
      v-if="stateData"
      class="flex flex-col md:flex-row items-start gap-6 w-full max-w-7xl mx-auto p-4 bg-white pdf-export-container"
      ref="pdfContent"
  >
    <!-- Левая колонка: Карточка -->
    <div class="w-full md:w-auto flex-shrink-0" >
      <CardComponent :data="stateData" @change-feature-in-c-s-v="changeFeature"/>
    </div>

    <!-- Правая колонка: График -->
    <div class="w-full md:flex-1 bg-white border border-slate-100 rounded-xl shadow-sm p-5">
      <ChartComponent />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Dialog from 'primevue/dialog';
import ProgressSpinner from "primevue/progressspinner";
import FileUploadComponent from "@/components/FileUploadComponent.vue";
import { clearAppState, stateData } from "@/composables/useAppState.js";
import axios from "axios";
import Button from "primevue/button";
import CardComponent from "@/components/CardComponent.vue";
import ChartComponent from "@/components/ChartComponent.vue";
import { useToast } from "primevue/usetoast";
import { toPng } from 'html-to-image';
import { jsPDF } from 'jspdf';

const isAnalyzing = ref(false);
const toast = useToast();
const pdfContent = ref(null);
const fileRef = ref(null);



const exportToPDF = async () => {
  if (!pdfContent.value) return;

  try {
    // 1. Превращаем HTML-компонент в PNG-картинку (поддерживает oklch из коробки!)
    const dataUrl = await toPng(pdfContent.value, {
      quality: 0.95,
      backgroundColor: '#ffffff' // гарантирует белый фон под графиком
    });

    // 2. Создаем чистый PDF-документ (альбомная ориентация)
    const pdf = new jsPDF({
      orientation: 'landscape',
      unit: 'px',
      format: [pdfContent.value.offsetWidth + 40, pdfContent.value.offsetHeight + 40]
      // Авто-размер страницы под размер вашего компонента на экране
    });

    // 3. Вставляем картинку в PDF
    pdf.addImage(dataUrl, 'PNG', 20, 20, pdfContent.value.offsetWidth, pdfContent.value.offsetHeight);

    // 4. Скачиваем
    pdf.save('ai-analysis-report.pdf');
  } catch (error) {
    console.error('Не удалось сгенерировать PDF:', error);
  }
};

const changeFeature = async payload => {};

const startLoading = async (payload) => {
  clearAppState();
  isAnalyzing.value = true;

  const isMock = import.meta.env.VITE_IS_MOCK_MODE === 'true';
  const url = isMock ? "/mock-data.json" : import.meta.env.VITE_API_URL + "/result";

  try {
    let res;
    if (isMock) {
      res = await axios.get(url);
    } else if (payload?.type === 'id') {
      res = await axios.get(url, { params: { id: payload.value } });
    } else if (payload?.type === 'file') {
      fileRef.value = payload.files;
      const formData = new FormData();
      if (payload.files && payload.files.length > 0) {
        formData.append('file', payload.files[0]);
      }
      res = await axios.post(url, formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
      });
    }

    if (res) {
      stateData.value = res.data;
    }
  } catch (e) {
    console.error('Ошибка при выполнении запроса:', e);
    toast.add({
      severity: "error",
      summary: "Error",
      detail: e.response?.data?.message || e.message,
      life: 3000,
    });
  } finally {
    isAnalyzing.value = false;
  }
}
</script>

<style scoped>
/* Перезаписываем oklch-переменные Tailwind/PrimeVue для контейнера PDF */
.pdf-export-container {
  background-color: #ffffff !important;
  color: #0f172a !important; /* slate-900 в hex */
}

/* Если внутри компонентов есть элементы с oklch,
   можно точечно переписать их для дочерних элементов */
.pdf-export-container :deep(*) {
  /* Принудительно заменяем oklch-бордюры на обычный hex, если они ломают скрипт */
  border-color: #e2e8f0 !important;
}
</style>

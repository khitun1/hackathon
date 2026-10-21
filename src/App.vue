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

  <!--
    Общий контейнер для карточки и графика.
    flex-col (на мобильных друг под другом) -> flex-row (на экранах md и больше в ряд).
  -->
  <div
      v-if="stateData"
      class="flex flex-col md:flex-row items-start gap-6 w-full max-w-7xl mx-auto p-4"
  >
    <!-- Левая колонка: Карточка (фиксированная ширина) -->
    <div class="w-full md:w-auto flex-shrink-0">
      <CardComponent :data="stateData"
      @change-feature-in-c-s-v="changeFeature"/>
    </div>

    <!-- Правая колонка: График (занимает все оставшееся место) -->
    <div class="w-full md:flex-1 bg-white border border-slate-100 rounded-xl shadow-sm p-5">
      <ChartComponent />
    </div>
  </div>

</template>


<script setup>
import {computed, ref} from 'vue';
import Dialog from 'primevue/dialog';
import ProgressSpinner from "primevue/progressspinner";
import FileUploadComponent from "@/components/FileUploadComponent.vue";
import {clearAppState, stateData} from "@/composables/useAppState.js";
import axios from "axios";
import CardComponent from "@/components/CardComponent.vue";
import ChartComponent from "@/components/ChartComponent.vue";
import {useToast} from "primevue/usetoast";

const isAnalyzing = ref(false);

const toast = useToast();


const fileRef = ref(null);

const changeFeature = async payload => {

};

const startLoading = async (payload) => {
  clearAppState();
  isAnalyzing.value = true;

  const isMock = import.meta.env.VITE_IS_MOCK_MODE === 'true';
  const url = isMock ?
      "/mock-data.json" :
      import.meta.env.VITE_API_URL + "/result";

  try {
    let res;

    if (isMock) {
      res = await axios.get(url);
    }
    else if (payload?.type === 'id') {
      console.log('Поиск по ID:', payload.value);
      res = await axios.get(url, {
        params: { id: payload.value }
      });
    }
    // 3. Реальный API режим для загрузки файлов
    else if (payload?.type === 'file') {
      console.log('Загрузка файла:', payload.files);
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

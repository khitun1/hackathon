<template>
  <div class="ml-4 mt-4">
    <file-upload-component/>

    <Button
        label="Запустить ML-анализ"
        icon="pi pi-play"
        severity="primary"
        size="large"
        @click="startLoading"
    />

    <Dialog
        v-model:visible="isDialogVisible"
        modal
        header="Анализ данных алгоритмами ML"
        :style="{ width: '100%', maxWidth: '30rem' }"
        class="custom-v4-dialog"
        @hide="endLoading"
    >
      <TimelineComponent v-if="isDialogVisible" @close="isDialogVisible = false" />
    </Dialog>

  </div>
</template>

<script setup>
import {ref, shallowRef} from 'vue';
import Button from 'primevue/button';
import Dialog from 'primevue/dialog';
import TimelineComponent from './components/TimelineComponent.vue';
import {clearAppState} from "@/composables/useAppState.js";
import FileUploadComponent from "@/components/FileUploadComponent.vue";

const isDialogVisible = ref(false);
let isAnalyzing = ref(false);


const startLoading = () => {
  clearAppState();
  isDialogVisible.value = true;
  isAnalyzing.value = true;
}

const endLoading = () => {
  isAnalyzing.value = false;
}
</script>

<style scoped>

/* Стили центрирования шапки диалога остаются на родителе, так как Dialog рендерится в корне родителя */
.custom-v4-dialog ::v-deep(.p-dialog-header) {
  display: flex !important;
  justify-content: center !important;
  text-align: center !important;
  padding-bottom: 0 !important;
  position: relative;
}

.custom-v4-dialog ::v-deep(.p-dialog-header-actions) {
  position: absolute;
  right: 1.5rem;
  top: 1.5rem;
}

.custom-v4-dialog ::v-deep(.p-dialog-title) {
  width: 100% !important;
  text-align: center !important;
  color: var(--p-primary-color) !important;
  font-weight: 700 !important;
  font-size: 1.5rem !important;
}

.custom-v4-dialog ::v-deep(.p-dialog-content) {
  overflow: visible !important;
}
</style>

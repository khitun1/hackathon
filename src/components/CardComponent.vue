<!-- src/components/ScoringResultCard.vue -->
<template>
  <div class="bg-white border border-slate-100 rounded-xl shadow-sm p-4 max-w-md w-full">
    <!-- Шапка карточки: ID и Вероятность -->
    <div class="flex items-center justify-between border-b border-slate-100 pb-3 mb-3">
      <div class="flex items-center gap-2">
        <span class="text-xs font-semibold uppercase tracking-wider text-slate-400">Физлицо:</span>
        <span class="text-base font-bold text-slate-800">#{{ data.id }}</span>
      </div>

      <div class="flex items-center gap-2">
        <span class="text-xs font-medium text-slate-500">Риск банкротства:</span>
        <!-- Динамический цвет в зависимости от уровня вероятности -->
        <span
            class="text-lg font-black px-2.5 py-0.5 rounded-md"
            :class="probabilityColor"
        >
          {{ (data.probability * 100).toFixed(0) }}%
        </span>
      </div>
    </div>

    <!-- Заголовок секции метрик -->
    <div class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-2">
      Ключевые факторы влияния (отклонение от эталона)
    </div>

    <!-- Компактный список метрик -->
    <div class="space-y-1.5 max-h-[240px] overflow-y-auto pr-1">
      <div
          v-for="feature in data.top_features"
          :key="feature.name"
          @click="changeFeature(feature)"
          class="flex items-center justify-between text-sm py-1 border-b border-slate-50 last:border-0 hover:bg-slate-50/50 px-1 rounded transition-colors cursor-pointer"
      >
        <!-- Название фичи и индикатор влияния (вверх/вниз) -->
        <div class="flex items-center gap-1.5 text-slate-700 font-medium truncate max-w-[180px]">
          <i
              :class="[
              'pi text-[10px]',
              feature.shap_value > 0 ? 'pi-arrow-up-right text-rose-500' : 'pi-arrow-down-left text-emerald-500'
            ]"
          />
          <span :title="feature.name">{{ feature.name }}</span>
        </div>

        <!-- Значения: Текущее (Базовое) -->
        <div class="flex items-center gap-1.5 whitespace-nowrap">
          <span class="font-semibold text-slate-800">{{ feature.raw_value }}</span>
          <span class="text-xs text-slate-400 font-normal" title="Базовое значение (Baseline)">
            ({{ feature.baseline_value }})
          </span>
        </div>
      </div>
    </div>

    <Dialog
        v-model:visible="visibleChangeFeature"
    >
      <ChangeFeatureComponent
          :old-value="oldValue"
          @change="change"/>
    </Dialog>

  </div>
</template>

<script setup>
import {computed, ref} from 'vue';
import Dialog from "primevue/dialog";
import ChangeFeatureComponent from "@/components/ChangeFeatureComponent.vue";

const props = defineProps({
  data: {
    type: Object,
    required: true,
    default: () => ({
      id: null,
      probability: 0,
      top_features: []
    })
  }
});

const emit = defineEmits(['changeFeatureInCSV']);

const oldValue = ref();
const changedFeature = ref('');
const visibleChangeFeature = ref(false);

const changeFeature = feature => {
  oldValue.value = feature.raw_value;
  changedFeature.value = feature.name;
  visibleChangeFeature.value = true;

};

const change = newValue => {
  emit('changeFeatureInCSV', {
    newValue,
    nameFeature: changedFeature.value,
  })
  visibleChangeFeature.value = false;

}

// Динамическое окрашивание плашки вероятности банкротства
const probabilityColor = computed(() => {
  const p = props.data.probability;
  if (p >= 0.7) return 'bg-rose-50 text-rose-700 border border-rose-200';
  if (p >= 0.4) return 'bg-amber-50 text-amber-700 border border-amber-200';
  return 'bg-emerald-50 text-emerald-700 border border-emerald-200';
});
</script>

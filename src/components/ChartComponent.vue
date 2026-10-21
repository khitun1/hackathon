<template>
  <div class="chart-container">
    <h3 class="chart-title">
      Анализ факторов риска (Отклонение от эталона)
    </h3>
    <div class="model-summary">
      Вероятность банкротства:
      <span :class="['prob-badge', stateData?.bankrupt ? 'risk' : 'safe']">
        {{ ((stateData?.probability ?? 0) * 100).toFixed(1) }}%
      </span>
    </div>

    <!-- Компонент графика от vue-echarts -->
    <v-chart
        class="shap-chart"
        :option="shapChartOption"
        autoresize
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import VChart from 'vue-echarts';

// Импортируем ядро ECharts и только необходимые для Bar Chart модули
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { BarChart } from 'echarts/charts';
import { GridComponent, TooltipComponent } from 'echarts/components';

// Регистрируем модули ECharts
use([
  CanvasRenderer,
  BarChart,
  GridComponent,
  TooltipComponent
]);

// Интерфейсы для строгой типизации (TS)
interface ShapFeature {
  name: string;
  raw_value: number;
  baseline_value: number;
  shap_value: number;
}

interface StateData {
  id: number;
  bankrupt: boolean;
  probability: number;
  top_features: ShapFeature[];
}

// Реактивный стейт с вашими входными данными
const stateData = ref<StateData | null>({
  id: 111,
  bankrupt: true,
  probability: 0.84,
  top_features: [
    { name: "feature_1", raw_value: 0.4,  baseline_value: 0.9,  shap_value: 0.9 },
    { name: "feature_2", raw_value: 0.8,  baseline_value: 0.2,  shap_value: 0.2 },
    { name: "feature_3", raw_value: 0.6,  baseline_value: 0.4,  shap_value: -0.4 },
    { name: "feature_4", raw_value: 0.75, baseline_value: 0.15, shap_value: 0.15 },
    { name: "feature_5", raw_value: 0.15, baseline_value: 0.75, shap_value: -0.75 }
  ]
});

// Вычисляемое свойство для конфигурации ECharts
const shapChartOption = computed(() => {
  if (!stateData.value || !stateData.value.top_features) {
    return {};
  }

  // Разворачиваем массив, чтобы топ-фичи рендерились СВЕРХУ вниз
  const features = [...stateData.value.top_features].reverse();
  const yAxisData = features.map(f => f.name);

  const seriesData = features.map(f => {
    const val = f.shap_value;
    const isPositive = val >= 0;

    return {
      value: val,
      itemStyle: {
        color: isPositive ? '#ef4444' : '#3b82f6',
        // [Top-Left, Top-Right, Bottom-Right, Bottom-Left]
        borderRadius: isPositive ? [0, 4, 4, 0] : [4, 0, 0, 4]
      }
    };
  });

  return {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      },
      formatter: (params: any) => {
        const item = Array.isArray(params) ? params[0] : params;
        if (!item) return '';

        const val = item.value;
        const isPositive = val >= 0;

        const badge = isPositive
            ? '<span style="color: #ef4444; font-weight: 600;">🔴 Повышает вероятность банкротства</span>'
            : '<span style="color: #3b82f6; font-weight: 600;">🔵 Понижает вероятность банкротства</span>';

        return `
          <div style="font-family: sans-serif; padding: 4px;">
            <div style="font-weight: bold; color: #1e293b; margin-bottom: 6px;">${item.name}</div>
            <div style="color: #475569; margin-bottom: 6px;">
              Вклад (SHAP): <b style="color: #0f172a;">${val > 0 ? '+' : ''}${val.toFixed(4)}</b>
            </div>
            <div style="font-size: 12px; border-top: 1px solid #e2e8f0; padding-top: 6px;">${badge}</div>
          </div>
        `;
      }
    },
    grid: {
      left: '4%',
      right: '6%',
      bottom: '5%',
      top: '5%',
      containLabel: true
    },
    xAxis: {
      type: 'value',
      axisLabel: {
        color: '#64748b'
      },
      splitLine: {
        show: true,
        lineStyle: {
          color: '#f1f5f9',
          type: 'dashed'
        }
      }
    },
    yAxis: {
      type: 'category',
      data: yAxisData,
      axisTick: {
        show: false
      },
      axisLine: {
        lineStyle: {
          color: '#cbd5e1'
        }
      },
      axisLabel: {
        color: '#334155',
        fontWeight: 500,
        fontSize: 12
      }
    },
    series: [
      {
        name: 'SHAP Value',
        type: 'bar',
        data: seriesData,
        barWidth: '55%'
      }
    ]
  };
});
</script>

<style scoped>
.chart-container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 24px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

.chart-title {
  margin: 0 0 8px 0;
  color: #1e293b;
  font-size: 18px;
  font-weight: 600;
}

.model-summary {
  margin-bottom: 20px;
  color: #64748b;
  font-size: 14px;
}

.prob-badge {
  padding: 2px 8px;
  border-radius: 4px;
  font-weight: bold;
}

.prob-badge.risk {
  background-color: #fee2e2;
  color: #991b1b;
}

.prob-badge.safe {
  background-color: #dcfce7;
  color: #166534;
}

/* Обязательно задаем высоту графику, иначе ECharts сожмется в 0px */
.shap-chart {
  height: 450px;
  width: 100%;
}
</style>

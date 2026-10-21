<script setup>

  import Button from "primevue/button";
  import {ref} from "vue";

  const newFeature = ref(0);

  const props = defineProps({
    oldValue: Number
  });

  const emit = defineEmits(['change']);

  const change = () => {
    emit("change", Number(newFeature.value));
  }
</script>

<template>
  <label for="newValue">старое значение: {{props.oldValue}}</label><br>
  <input id="newValue" placeholder="newValue" v-model="newFeature"/><br>
  <Button @click="change">Изменить</Button>
</template>

<style scoped>

</style>
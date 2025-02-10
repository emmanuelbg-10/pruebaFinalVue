<script setup>
import { ref, watch, onMounted, computed } from 'vue';
const props = defineProps(['budget', 'price']);
const budget = ref(props.budget || localStorage.getItem('budget') || 0);
const emit = defineEmits(['update:budget']);

watch(budget, (newBudget) => {
  if (newBudget < 0) {
    budget.value = 0;
  }
  localStorage.setItem('budget', newBudget);
  emit('update:budget', newBudget);
});

onMounted(() => {
  const storedBudget = localStorage.getItem('budget');
  if (storedBudget) {
    budget.value = parseFloat(storedBudget);
  }
});

const budgetClass = computed(() => {
  if (props.price > budget.value) {
    return 'over-budget';
  } else if (props.price >= budget.value * 0.9) {
    return 'near-budget';
  } else {
    return 'under-budget';
  }
});
</script>

<template>
  <p>Indica tu presupuesto <input type="number" v-model="budget"></p>
  <h2 v-if="budget > 0">Su presupuesto es <span :class="budgetClass">{{ budget }} €</span></h2>
</template>

<style scoped>
.over-budget {
  color: red;
}

.near-budget {
  color: orange;
}

.under-budget {
  color: green;
}
</style>
<script setup>
defineProps({
  inputId: String,
  label: String,
  value: String,
  usdAllocation: String,
  rate: String,
  rateLabel: String,
});
</script>

<template>
  <div class="allocation-card">
    <div class="field-group">
      <label :for="inputId">{{ label }}</label>
      <input :id="inputId" :value="value" readonly />
    </div>

    <div class="allocation-info">
      <p><b>Allocation (USD):</b> {{ usdAllocation }}</p>
      <p><b>Rate:</b> {{ rate }} {{ rateLabel }}</p>
    </div>
  </div>
</template>

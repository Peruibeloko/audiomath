<script setup lang="ts">
defineProps(['modelValue']);
defineEmits(['update:modelValue', 'select']);
</script>

<template>
  <select
    :value="modelValue"
    @change="$emit('update:modelValue', ($event?.target as HTMLSelectElement).value), $emit('select')"
  >
    <option value="8000">8,000 Hz</option>
    <option value="11025">11,025 Hz</option>
    <option value="16000">16,000 Hz</option>
    <option value="22050">22,050 Hz</option>
    <option value="44100">44,100 Hz</option>
    <option value="48000">48,000 Hz</option>
    <option value="88200">88,200 Hz</option>
    <option value="96000">96,000 Hz</option>
    <option value="176400">176,400 Hz</option>
    <option value="192000">192,000 Hz</option>
    <option value="352800">352,800 Hz</option>
    <option value="384000">384,000 Hz</option>
  </select>
</template>

<style scoped>
select {
  display: block;
  padding: 0.5rem;

  font-family: 'PT Serif', sans-serif;
  font-size: 2rem;

  border: none;
  border: 3px solid hsla(40, 100%, 92%, 0.863);
  border-radius: 1rem;
}
</style>

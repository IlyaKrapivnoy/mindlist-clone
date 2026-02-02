<template>
  <div>
    <div class="text-lg font-bold text-gray-800">{{ currentDate }}</div>
  </div>
</template>

<script setup>
const currentDate = new Date().toLocaleDateString("en-US", {
  month: "long",
  day: "numeric",
});
</script>

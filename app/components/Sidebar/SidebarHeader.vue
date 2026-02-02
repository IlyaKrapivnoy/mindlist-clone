<template>
  <div>
    <SidebarHeaderTop />
    <SidebarHeaderBottom />
  </div>
</template>

<script setup>
const currentDate = new Date().toLocaleDateString("en-US", {
  month: "long",
  day: "numeric",
});
</script>

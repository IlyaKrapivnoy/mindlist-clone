<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>

<script setup>
import { appName, appDescription } from "~/constants";

useHead({
  title: appName,
  meta: [
    {
      name: "description",
      content: appDescription,
    },
  ],
});
</script>

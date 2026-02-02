<template>
  <div class="flex items-center justify-between text-[11px] text-gray-500">
    <div class="uppercase">{{ currentDay }}</div>
    <div>
      <Cog8ToothIcon
        class="w-5 h-5 cursor-pointer hover:text-gray-800 transition-colors duration-200"
        aria-label="Settings"
        title="Settings"
        @click="console.log('Settings clicked')"
      />
    </div>
  </div>
</template>

<script setup>
import { Cog8ToothIcon } from "@heroicons/vue/24/outline";

const currentDay = new Date().toLocaleDateString("en-US", { weekday: "long" });
</script>

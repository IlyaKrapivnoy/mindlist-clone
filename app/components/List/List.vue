<template>
  <div class="p-4 h-screen flex flex-col">
    <h2>List</h2>
    <div class="mt-auto">
      <div class="relative">
        <PlusCircleIconSolid
          class="absolute left-3 top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400"
        />
        <input
          type="text"
          placeholder="Add new task"
          class="w-full pl-10 pr-3 py-2 text-xs text-black placeholder:text-gray-500 border border-gray-300 rounded-md focus:outline-none focus:ring-1 focus:ring-gray-500 focus:border-transparent"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { PlusCircleIcon as PlusCircleIconSolid } from "@heroicons/vue/24/solid";
</script>

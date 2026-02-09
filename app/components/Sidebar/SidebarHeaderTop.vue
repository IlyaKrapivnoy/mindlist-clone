<template>
  <div class="flex items-center justify-between text-[11px] text-gray-500">
    <div class="uppercase">{{ currentDay }}</div>
    <div>
      <InformationCircleIcon
        class="w-5 h-5 cursor-pointer hover:text-gray-800 transition-colors duration-200 mr-[10px]"
        aria-label="Info"
        title="Info"
        @click="showModal = true"
      />
    </div>
  </div>

  <!-- Modal -->
  <div
    v-if="showModal"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    @click.self="showModal = false"
  >
    <div class="bg-white rounded-lg p-6 w-64 shadow-xl">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-lg font-semibold text-gray-900">Info</h3>
        <button
          @click="showModal = false"
          class="text-gray-400 hover:text-gray-600 transition-colors"
        >
          <XMarkIcon class="w-5 h-5" />
        </button>
      </div>

      <div class="space-y-3">
        <a
          href="https://github.com/IlyaKrapivnoy"
          target="_blank"
          rel="noopener noreferrer"
          class="flex items-center justify-center w-full px-4 py-3 bg-gray-900 text-white rounded-lg hover:bg-gray-800 transition-colors duration-200"
        >
          GitHub
        </a>

        <a
          href="https://professional-card-one.vercel.app/"
          target="_blank"
          rel="noopener noreferrer"
          class="flex items-center justify-center w-full px-4 py-3 bg-gray-100 text-gray-900 rounded-lg hover:bg-gray-200 transition-colors duration-200"
        >
          About
        </a>
      </div>

      <div class="mt-4 text-center text-xs text-gray-500">Version 1.0.0</div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { InformationCircleIcon, XMarkIcon } from "@heroicons/vue/24/outline";

const currentDay = new Date().toLocaleDateString("en-US", { weekday: "long" });
const showModal = ref(false);
</script>

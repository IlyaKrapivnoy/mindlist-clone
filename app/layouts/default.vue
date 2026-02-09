<template>
  <div class="flex h-screen relative">
    <!-- Mobile Menu Button -->
    <button
      v-if="isMobile"
      @click="toggleSidebar"
      class="fixed top-4 right-4 z-50 p-2 bg-gray-100 rounded-lg shadow-md lg:hidden"
    >
      <Bars3Icon v-if="!showSidebar" class="w-6 h-6 text-gray-700" />
      <XMarkIcon v-else class="w-6 h-6 text-gray-700" />
    </button>

    <!-- Sidebar -->
    <aside
      :class="[
        'bg-gray-100/70 transition-all duration-300 z-40',
        isMobile
          ? showSidebar
            ? 'fixed inset-0 w-full'
            : 'fixed -left-full w-full'
          : 'w-[16%] min-w-[260px] relative',
      ]"
    >
      <Sidebar />
    </aside>

    <!-- Overlay for mobile -->
    <div
      v-if="isMobile && showSidebar"
      @click="toggleSidebar"
      class="fixed inset-0 bg-black bg-opacity-50 z-30 lg:hidden"
    />

    <!-- Main Content -->
    <main :class="[isMobile ? 'w-full' : 'flex-1']">
      <slot />
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { Bars3Icon, XMarkIcon } from "@heroicons/vue/24/outline";

const isMobile = ref(false);
const showSidebar = ref(false);

const checkMobile = () => {
  isMobile.value = window.innerWidth < 1024; // lg breakpoint
};

const toggleSidebar = () => {
  showSidebar.value = !showSidebar.value;
};

onMounted(() => {
  checkMobile();
  window.addEventListener("resize", checkMobile);
});

onUnmounted(() => {
  window.removeEventListener("resize", checkMobile);
});
</script>

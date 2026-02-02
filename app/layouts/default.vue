<template>
  <div class="flex h-screen">
    <aside class="w-[20%] bg-gray-100/70">
      <Sidebar />
    </aside>
    <main class="flex-1">
      <slot />
    </main>
  </div>
</template>

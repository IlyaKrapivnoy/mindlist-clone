<template>
  <div class="flex h-screen">
    <aside class="w-[16%] min-w-[260px] bg-gray-100/70">
      <Sidebar />
    </aside>
    <main class="flex-1">
      <slot />
    </main>
  </div>
</template>

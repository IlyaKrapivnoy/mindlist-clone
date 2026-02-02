<template>
  <div class="mt-6">
    <h3 class="text-xs font-semibold text-gray-500 uppercase mb-3">Lists</h3>
    <ul class="space-y-1">
      <li v-for="list in lists" :key="list.id">
        <button
          class="w-full text-left px-2 py-1 rounded hover:bg-gray-100 text-sm text-gray-700 hover:text-gray-900 transition-colors duration-150"
          @click="selectList(list.id)"
        >
          <div class="flex items-center gap-2">
            <TagIcon class="w-4 h-4 text-gray-500" />
            <span>{{ list.name }}</span>
          </div>
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { TagIcon } from "@heroicons/vue/24/outline";
import { ref, onMounted } from "vue";

const lists = ref([]);

onMounted(() => {
  const savedLists = localStorage.getItem("lists");
  if (savedLists) {
    lists.value = JSON.parse(savedLists);
  } else {
    lists.value = [{ id: "my-list", name: "My List", icon: "tag" }];
    localStorage.setItem("lists", JSON.stringify(lists.value));
  }
});

const selectList = (listId) => {
  console.log("Selected list:", listId);
  // TODO: Implement navigation between lists
};
</script>

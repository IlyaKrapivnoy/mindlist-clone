<template>
  <div class="mt-6">
    <h3 class="text-xs font-semibold text-gray-500 uppercase mb-3">Lists</h3>
    <ul class="space-y-1">
      <li v-for="list in lists" :key="list.id">
        <button
          :class="[
            'w-full text-left px-2 py-2 rounded text-sm text-gray-700 transition-colors duration-150',
            (list.id === 'main-list' && route.path === '/') ||
            (list.id !== 'main-list' && route.path === `/lists/${list.id}`)
              ? 'bg-gray-200 text-gray-900'
              : '',
          ]"
          @click="selectList(list.id)"
        >
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-2">
              <TagIcon class="w-4 h-4 text-gray-500" />
              <span>{{ list.name }}</span>
            </div>
            <span
              v-if="getTaskCount(list.id) > 0"
              class="bg-gray-200 text-gray-700 px-2 py-0.5 rounded-[6px] text-xs font-medium"
            >
              {{ getTaskCount(list.id) }}
            </span>
          </div>
        </button>
      </li>
    </ul>

    <!-- New List Button -->
    <button
      class="w-full text-left px-2 py-2 rounded text-sm text-gray-700 transition-colors duration-150 mt-2"
      @click="showNewListModal = true"
    >
      <div class="flex items-center gap-2">
        <PlusIcon class="w-4 h-4 text-gray-500" />
        <span>New list</span>
      </div>
    </button>

    <!-- New List Modal -->
    <div
      v-if="showNewListModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-6 w-96">
        <h2 class="text-lg font-semibold mb-4">Create new list</h2>
        <input
          v-model="newListName"
          type="text"
          placeholder="List name"
          class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-1 focus:ring-gray-900 focus:border-transparent mb-4"
          @keyup.enter="createNewList"
        />
        <div class="flex justify-end gap-2">
          <button
            @click="showNewListModal = false"
            class="px-4 py-2 text-gray-600"
          >
            Cancel
          </button>
          <button
            @click="createNewList"
            class="px-4 py-2 bg-gray-800 text-white rounded-md"
          >
            Create
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { TagIcon, PlusIcon } from "@heroicons/vue/24/outline";
import { ref, onMounted, computed } from "vue";
import { useRouter, useRoute } from "vue-router";

const router = useRouter();
const route = useRoute();

const lists = ref([]);
const showNewListModal = ref(false);
const newListName = ref("");
const taskCounts = ref({});

const getTaskCount = (listId) => {
  return taskCounts.value[listId] || 0;
};

const updateTaskCount = (listId) => {
  const savedTasks = localStorage.getItem(`tasks_${listId}`);
  if (savedTasks) {
    const tasks = JSON.parse(savedTasks);
    taskCounts.value[listId] = tasks.length;
  } else {
    taskCounts.value[listId] = 0;
  }
};

const updateAllTaskCounts = () => {
  lists.value.forEach((list) => {
    updateTaskCount(list.id);
  });
};

const watchLocalStorage = () => {
  const originalSetItem = localStorage.setItem;
  localStorage.setItem = function (key, value) {
    originalSetItem.call(this, key, value);
    if (key.startsWith("tasks_")) {
      const listId = key.replace("tasks_", "");
      updateTaskCount(listId);
    }
  };
};

onMounted(() => {
  const savedLists = localStorage.getItem("lists");
  if (savedLists) {
    lists.value = JSON.parse(savedLists);
  } else {
    lists.value = [{ id: "main-list", name: "Main" }];
    localStorage.setItem("lists", JSON.stringify(lists.value));
  }

  if (!lists.value.find((l) => l.id === "main-list")) {
    lists.value.unshift({ id: "main-list", name: "Main" });
    localStorage.setItem("lists", JSON.stringify(lists.value));
  }

  lists.value.forEach((list) => {
    updateTaskCount(list.id);
  });

  watchLocalStorage();
});

const selectList = (listId) => {
  if (listId === "main-list") {
    router.push("/");
  } else {
    router.push(`/lists/${listId}`);
  }
};

const createNewList = () => {
  if (newListName.value.trim() === "") return;

  const urlName = newListName.value
    .trim()
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, "")
    .replace(/[\s-]+/g, "-")
    .replace(/^-+|-+$/g, "");

  const newId = urlName || `list-${Date.now()}`;
  const newList = {
    id: newId,
    name: newListName.value.trim(),
    icon: "tag",
  };

  lists.value.push(newList);
  localStorage.setItem("lists", JSON.stringify(lists.value));

  newListName.value = "";
  showNewListModal.value = false;

  selectList(newId);
};
</script>

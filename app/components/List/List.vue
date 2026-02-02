<template>
  <div class="p-4 h-screen flex flex-col">
    <!--  header list -->
    <div class="mb-4">
      <div class="flex gap-2 items-center -mt-[4px]">
        <TagIcon class="w-5 h-5 text-gray-600" />
        <h1 class="text-lg font-bold">{{ currentList.name }}</h1>
      </div>
      <div class="text-[11px] text-gray-500 pl-[30px]">
        {{ tasks.length }} {{ tasks.length === 1 ? "task" : "tasks" }}
      </div>
    </div>

    <!--  todo list -->
    <div
      v-if="sortedTasks.length === 0"
      class="flex-1 flex flex-col items-center justify-center"
    >
      <img
        src="/assets/images/no_result.png"
        alt="No tasks"
        class="max-w-[640px] h-auto"
      />
      <div class="text-center text-xs">
        <p class="text-black font-bold mt-2">It's empty so far</p>
        <p class="text-gray-500 mt-2">Let's add a new task</p>
      </div>
    </div>
    <ul v-else class="text-sm">
      <li
        v-for="task in sortedTasks"
        :key="task.id"
        class="py-2 border-b border-gray-100 flex items-center"
      >
        <input
          type="checkbox"
          v-model="task.completed"
          class="mr-2 w-4 h-4 appearance-none border border-gray-300 rounded relative cursor-pointer checked:border-0 checked:bg-transparent checked:before:content-['✓'] checked:before:absolute checked:before:top-1/2 checked:before:left-1/2 checked:before:transform checked:before:-translate-x-1/2 checked:before:-translate-y-1/2 checked:before:text-gray-600 checked:before:text-sm checked:before:font-bold"
        />
        <span
          :class="{ 'line-through text-gray-400': task.completed }"
          class="ml-2"
          >{{ task.text }}</span
        >
      </li>
    </ul>
    <div class="mt-auto">
      <!-- input list -->
      <div class="relative">
        <PlusCircleIconSolid
          class="absolute left-3 top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400"
        />
        <input
          @keyup.enter="addTask"
          type="text"
          placeholder="Add new task"
          v-model="newTask"
          class="w-full pl-10 pr-3 py-2 text-xs text-black placeholder:text-gray-500 border border-gray-300 rounded-md focus:outline-none focus:ring-1 focus:ring-gray-500 focus:border-transparent"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import {
  PlusCircleIcon as PlusCircleIconSolid,
  TagIcon,
} from "@heroicons/vue/24/outline";
import { useRoute } from "vue-router";

const route = useRoute();

const tasks = ref([]);
const currentList = ref({ id: "my-list", name: "My List" });

const sortedTasks = computed(() => {
  return [...tasks.value].sort((a, b) => {
    if (a.completed !== b.completed) {
      return a.completed ? 1 : -1;
    }
    if (a.completed && b.completed) {
      return (a.completedAt || 0) - (b.completedAt || 0);
    }
    if (!a.completed && !b.completed) {
      return (b.uncompletedAt || 0) - (a.uncompletedAt || 0);
    }
    return 0;
  });
});

onMounted(() => {
  const listId = route.params.id || "my-list";

  const savedLists = localStorage.getItem("lists");
  if (savedLists) {
    const lists = JSON.parse(savedLists);
    const list = lists.find((l) => l.id === listId);
    if (list) {
      currentList.value = list;
    }
  }

  const savedTasks = localStorage.getItem(`tasks_${currentList.value.id}`);
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks);
  } else if (currentList.value.id === "my-list") {
    tasks.value = [
      { id: 1, text: "Learn Polish", completed: false },
      { id: 2, text: "Go to the gym", completed: true },
      { id: 3, text: "Go to the swimming pool", completed: false },
    ];
  } else {
    tasks.value = [];
  }
});

watch(
  tasks,
  (newTasks) => {
    localStorage.setItem(
      `tasks_${currentList.value.id}`,
      JSON.stringify(newTasks),
    );
  },
  { deep: true },
);

watch(
  () => tasks.value.map((task) => ({ id: task.id, completed: task.completed })),
  (newValues, oldValues) => {
    if (!oldValues) return;

    newValues.forEach((newValue, index) => {
      const oldValue = oldValues[index];
      if (oldValue && newValue.completed !== oldValue.completed) {
        const task = tasks.value.find((t) => t.id === newValue.id);
        if (task) {
          if (newValue.completed) {
            task.completedAt = Date.now();
            delete task.uncompletedAt;
          } else {
            task.uncompletedAt = Date.now();
            delete task.completedAt;
          }
        }
      }
    });
  },
  { deep: true },
);

const newTask = ref("");

const addTask = () => {
  if (newTask.value.trim() !== "") {
    const newId =
      tasks.value.length > 0
        ? Math.max(...tasks.value.map((t) => t.id)) + 1
        : 1;
    tasks.value.unshift({
      id: newId,
      text: newTask.value,
      completed: false,
    });
    newTask.value = "";
  }
};
</script>

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
    <ul class="text-sm">
      <li
        v-for="task in tasks"
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

const tasks = ref([]);
const currentList = ref({ id: "my-list", name: "My List" });

onMounted(() => {
  const savedLists = localStorage.getItem("lists");
  if (savedLists) {
    const lists = JSON.parse(savedLists);
    const savedTasks = localStorage.getItem(`tasks_${currentList.value.id}`);
    if (savedTasks) {
      tasks.value = JSON.parse(savedTasks);
    } else {
      tasks.value = [
        { id: 1, text: "Learn Polish", completed: false },
        { id: 2, text: "Go to the gym", completed: true },
        { id: 3, text: "Go to the swimming pool", completed: false },
      ];
    }
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

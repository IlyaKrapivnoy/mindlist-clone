<template>
  <div class="p-4 h-screen flex flex-col">
    <!--  todo list -->
    <ul class="text-sm">
      <li
        v-for="task in tasks"
        :key="task.id"
        class="py-2 border-b border-gray-100"
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
import { PlusCircleIcon as PlusCircleIconSolid } from "@heroicons/vue/24/solid";

const tasks = ref([]);

onMounted(() => {
  const savedTasks = localStorage.getItem("tasks");
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks);
  } else {
    tasks.value = [
      { id: 1, text: "Learn Polish", completed: false },
      { id: 2, text: "Go to the gym", completed: true },
      { id: 3, text: "Go to the swimming pool", completed: false },
    ];
  }
});

watch(
  tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
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

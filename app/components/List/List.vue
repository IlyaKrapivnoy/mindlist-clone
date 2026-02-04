<template>
  <div class="p-4 h-screen flex flex-col">
    <!--  header list -->
    <div class="mb-4">
      <div class="flex gap-2 items-center justify-between -mt-[4px]">
        <div class="flex gap-2 items-center">
          <TagIcon class="w-5 h-5 text-gray-600" />
          <h1 class="text-lg font-bold">{{ currentList.name }}</h1>
        </div>
        <button
          @click="showListMenuModal = true"
          class="p-1 rounded hover:bg-gray-100 transition-colors"
          title="List options"
        >
          <EllipsisVerticalIcon class="w-5 h-5 text-gray-500" />
        </button>
      </div>
      <div class="text-[11px] text-gray-500 pl-[30px]">
        {{ tasks.length }} {{ tasks.length === 1 ? "task" : "tasks" }}
      </div>
    </div>

    <!--  todo list -->
    <Suspense>
      <template #default>
        <div
          v-if="isLoading"
          class="flex-1 flex flex-col items-center justify-center"
        >
          <div
            class="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-500 mb-4"
          ></div>
          <p class="text-gray-500 text-sm">Loading tasks...</p>
        </div>

        <div
          v-else-if="sortedTasks.length === 0"
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
          <TransitionGroup name="task-list" tag="div" class="space-y-0">
            <li
              v-for="task in sortedTasks"
              :key="task.id"
              class="py-2 border-b border-gray-100 flex items-center group hover:bg-gray-50 px-4"
            >
              <label class="flex items-center flex-1 cursor-pointer">
                <div v-if="task.completed" class="mr-2">
                  <CheckIcon class="w-4 h-4 text-gray-600" />
                </div>
                <input
                  type="checkbox"
                  v-model="task.completed"
                  :class="[
                    'mr-2 appearance-none border rounded relative cursor-pointer',
                    task.completed ? 'hidden' : 'w-4 h-4',
                    task.priority === 'High'
                      ? 'border-red-500'
                      : task.priority === 'Medium'
                        ? 'border-yellow-500'
                        : task.priority === 'Low'
                          ? 'border-blue-500'
                          : 'border-gray-300',
                  ]"
                  style="accent-color: transparent"
                />
                <div class="ml-2 flex-1 flex flex-col">
                  <span
                    v-if="editingTask?.id !== task.id"
                    :class="{ 'line-through text-gray-400': task.completed }"
                    >{{ task.text }}</span
                  >
                  <div v-else class="relative">
                    <input
                      v-model="editingText"
                      @keyup.enter="saveEdit"
                      @keyup.esc="cancelEdit"
                      @blur="saveEdit"
                      class="w-[90%] text-sm border border-gray-300 rounded px-1 py-0.5 pr-12 focus:outline-none focus:ring-1 focus:ring-gray-500"
                      ref="editInput"
                    />
                    <button
                      @click.stop="saveEdit"
                      class="absolute right-1 top-1/2 transform -translate-y-1/2 p-0.5 rounded hover:bg-gray-200 transition-colors"
                      title="Save changes"
                    >
                      <CheckCircleIcon class="w-5 h-5 text-green-600" />
                    </button>
                  </div>
                  <span
                    class="text-[11px] mt-0.5"
                    :class="
                      highlightedTasks.has(task.id)
                        ? 'text-red-500'
                        : 'text-gray-400'
                    "
                  >
                    {{ formatTaskDate(task.createdAt) }}
                  </span>
                </div>
              </label>

              <div
                v-if="editingTask?.id !== task.id"
                class="flex items-center gap-1 opacity-0 group-hover:opacity-100 transition-opacity duration-150"
              >
                <button
                  v-if="!task.completed"
                  @click="startEdit(task)"
                  class="p-1 rounded hover:bg-gray-200 transition-colors"
                  title="Edit task"
                >
                  <PencilIcon class="w-4 h-4 text-gray-500" />
                </button>
                <button
                  v-if="!task.completed"
                  @click="setTaskPriority(task)"
                  class="p-1 rounded hover:bg-gray-200 transition-colors"
                  title="Set priority"
                >
                  <FlagIcon
                    :class="[
                      'w-4 h-4',
                      task.priority === 'High'
                        ? 'text-red-500'
                        : task.priority === 'Medium'
                          ? 'text-yellow-500'
                          : task.priority === 'Low'
                            ? 'text-blue-500'
                            : 'text-gray-500',
                    ]"
                  />
                </button>
                <button
                  v-if="!task.completed"
                  @click="toggleTaskHighlight(task.id)"
                  class="p-1 rounded hover:bg-gray-200 transition-colors"
                  title="Highlight date"
                >
                  <CalendarIcon
                    :class="[
                      'w-4 h-4',
                      highlightedTasks.has(task.id)
                        ? 'text-red-500'
                        : 'text-gray-500',
                    ]"
                  />
                </button>
                <button
                  @click="deleteTask(task.id)"
                  class="p-1 rounded hover:bg-gray-200 transition-colors"
                  title="Delete task"
                >
                  <TrashIcon class="w-4 h-4 text-gray-500" />
                </button>
              </div>
            </li>
          </TransitionGroup>
        </ul>
      </template>

      <template #fallback>
        <div class="flex-1 flex flex-col items-center justify-center">
          <div
            class="animate-spin rounded-full h-12 w-12 border-b-2 border-gray-500 mb-4"
          ></div>
          <p class="text-gray-500 text-sm">Loading tasks...</p>
        </div>
      </template>
    </Suspense>
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

    <!-- List Menu Modal -->
    <div
      v-if="showListMenuModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-6 w-96">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-lg font-semibold text-sm">List options</h2>
          <button
            @click="showListMenuModal = false"
            class="p-1 rounded hover:bg-gray-100 transition-colors"
          >
            <XMarkIcon class="w-5 h-5 text-gray-500" />
          </button>
        </div>

        <div class="space-y-1">
          <div
            v-if="currentList.id === 'main-list' && sortedTasks.length === 0"
            class="px-3 py-2 text-sm text-gray-500 text-center"
          >
            This is your starter list.<br />
            <span class="font-bold">Create your first task to begin.</span>
          </div>
          <template
            v-if="!(currentList.id === 'main-list' && sortedTasks.length === 0)"
          >
            <button
              @click="clearCompletedTasks"
              class="w-full text-left px-3 py-2 rounded hover:bg-gray-100 transition-colors text-sm"
            >
              Clear completed tasks
            </button>
            <button
              @click="deleteAllTasks"
              class="w-full text-left px-3 py-2 rounded hover:bg-gray-100 transition-colors text-sm"
            >
              Delete all tasks
            </button>
            <button
              @click="renameList"
              class="w-full text-left px-3 py-2 rounded hover:bg-gray-100 transition-colors text-sm"
            >
              Rename list
            </button>
            <button
              v-if="currentList.id !== 'main-list'"
              @click="deleteList"
              class="w-full text-left px-3 py-2 rounded hover:bg-red-50 text-red-600 hover:text-red-700 transition-colors text-sm"
            >
              Delete list
            </button>
          </template>
        </div>
      </div>
    </div>

    <!-- Priority Modal -->
    <div
      v-if="showPriorityModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-6 w-96">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-lg font-semibold text-sm flex-1 text-center">
            Set priority
          </h2>
          <button
            @click="showPriorityModal = false"
            class="p-1 rounded hover:bg-gray-100 transition-colors"
          >
            <XMarkIcon class="w-5 h-5 text-gray-500" />
          </button>
        </div>

        <div>
          <button
            v-for="priority in priorityOptions"
            :key="priority.value"
            @click="updateTaskPriority(priority.value)"
            class="w-full text-left px-3 py-2 rounded hover:bg-gray-100 transition-colors flex items-center gap-3"
          >
            <FlagIcon :class="['w-4 h-4', priority.color]" />
            <span>{{ priority.label }}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {
  PlusCircleIcon as PlusCircleIconSolid,
  TagIcon,
  FlagIcon,
  TrashIcon,
  XMarkIcon,
  CheckIcon,
  PencilIcon,
  CheckCircleIcon,
  EllipsisVerticalIcon,
  CalendarIcon,
} from "@heroicons/vue/24/outline";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();

const getInitialList = () => {
  const listId = route.params.id || "main-list";

  if (typeof window === "undefined") {
    if (listId === "main-list") {
      return { id: "main-list", name: "Main" };
    }
    return {
      id: listId,
      name: listId.replace(/-/g, " ").replace(/\b\w/g, (l) => l.toUpperCase()),
    };
  }

  const savedLists = localStorage.getItem("lists");

  if (savedLists) {
    const lists = JSON.parse(savedLists);
    const list = lists.find((l) => l.id === listId);
    if (list) return list;
  }

  if (listId === "main-list") {
    return { id: "main-list", name: "Main" };
  }

  return {
    id: listId,
    name: listId.replace(/-/g, " ").replace(/\b\w/g, (l) => l.toUpperCase()),
  };
};

const tasks = ref([]);
const currentList = ref(getInitialList());

useSeoMeta({
  title: () => `${currentList.value.name} - Task Manager`,
  description: () =>
    `Manage your tasks in ${currentList.value.name} list efficiently`,
});
const showPriorityModal = ref(false);
const selectedTask = ref(null);
const editingTask = ref(null);
const editingText = ref("");
const showListMenuModal = ref(false);
const highlightedTasks = ref(new Set());
const isLoading = ref(true);

const priorityOptions = [
  { value: null, label: "No priority", color: "text-gray-400" },
  { value: "Low", label: "Low", color: "text-blue-500" },
  { value: "Medium", label: "Medium", color: "text-yellow-500" },
  { value: "High", label: "High", color: "text-red-500" },
];

const sortedTasks = computed(() => {
  return [...tasks.value].sort((a, b) => {
    if (a.completed !== b.completed) {
      return a.completed ? 1 : -1;
    }
    if (a.completed && b.completed) {
      return (a.completedAt || 0) - (b.completedAt || 0);
    }
    if (!a.completed && !b.completed) {
      return b.createdAt - a.createdAt;
    }
    return 0;
  });
});

const loadTasksData = async () => {
  isLoading.value = true;
  const listId = route.params.id || "main-list";

  const savedLists = localStorage.getItem("lists");
  if (savedLists) {
    const lists = JSON.parse(savedLists);
    const list = lists.find((l) => l.id === listId);
    if (list) {
      currentList.value = list;
    } else if (listId === "main-list") {
      currentList.value = { id: "main-list", name: "Main" };
    } else {
      currentList.value = {
        id: listId,
        name: listId
          .replace(/-/g, " ")
          .replace(/\b\w/g, (l) => l.toUpperCase()),
      };
    }
  } else if (listId === "main-list") {
    currentList.value = { id: "main-list", name: "Main" };
  } else {
    currentList.value = {
      id: listId,
      name: listId.replace(/-/g, " ").replace(/\b\w/g, (l) => l.toUpperCase()),
    };
  }

  await new Promise((resolve) => setTimeout(resolve, 500));

  const savedHighlightedTasks = localStorage.getItem(
    `highlightedTasks_${currentList.value.id}`,
  );
  if (savedHighlightedTasks) {
    highlightedTasks.value = new Set(JSON.parse(savedHighlightedTasks));
  }

  const savedTasks = localStorage.getItem(`tasks_${currentList.value.id}`);
  if (savedTasks) {
    const parsedTasks = JSON.parse(savedTasks);
    tasks.value = (Array.isArray(parsedTasks) ? parsedTasks : []).map((t) => ({
      ...t,
      createdAt: t.createdAt ?? Date.now(),
    }));
  } else if (currentList.value.id === "main-list") {
    tasks.value = [
      { id: 1, text: "Learn Polish", completed: false, createdAt: Date.now() },
      { id: 2, text: "Go to the gym", completed: true, createdAt: Date.now() },
      {
        id: 3,
        text: "Go to the swimming pool",
        completed: false,
        createdAt: Date.now(),
      },
    ];
  } else {
    tasks.value = [];
  }

  isLoading.value = false;
  return { tasks: tasks.value, currentList: currentList.value };
};

onMounted(() => {
  loadTasksData();
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
      priority: null,
      createdAt: Date.now(),
    });
    newTask.value = "";
  }
};

const deleteTask = (taskId) => {
  tasks.value = tasks.value.filter((task) => task.id !== taskId);
};

const setTaskPriority = (task) => {
  selectedTask.value = task;
  showPriorityModal.value = true;
};

const updateTaskPriority = (priority) => {
  if (selectedTask.value) {
    selectedTask.value.priority = priority;
    showPriorityModal.value = false;
    selectedTask.value = null;
  }
};

const formatTaskDate = (timestamp) => {
  if (!timestamp) return "";

  const date = new Date(timestamp);
  const formatted = date.toLocaleDateString("en-US", {
    month: "short",
    day: "numeric",
    year: "numeric",
  });

  return formatted.toLowerCase();
};

const startEdit = (task) => {
  editingTask.value = task;
  editingText.value = task.text;
  nextTick(() => {
    const input = document.querySelector('input[v-model="editingText"]');
    if (input) {
      input.focus();
      input.select();
    }
  });
};

const saveEdit = () => {
  if (editingTask.value && editingText.value.trim() !== "") {
    editingTask.value.text = editingText.value.trim();
  }
  editingTask.value = null;
  editingText.value = "";
};

const cancelEdit = () => {
  editingTask.value = null;
  editingText.value = "";
};

const toggleTaskHighlight = (taskId) => {
  if (highlightedTasks.value.has(taskId)) {
    highlightedTasks.value.delete(taskId);
  } else {
    highlightedTasks.value.add(taskId);
  }
  localStorage.setItem(
    `highlightedTasks_${currentList.value.id}`,
    JSON.stringify([...highlightedTasks.value]),
  );
};

const clearCompletedTasks = () => {
  tasks.value = tasks.value.filter((task) => !task.completed);
  showListMenuModal.value = false;
};

const deleteAllTasks = () => {
  if (confirm("Are you sure you want to delete all tasks?")) {
    tasks.value = [];
    showListMenuModal.value = false;
  }
};

const renameList = () => {
  const newName = prompt("Enter new list name:", currentList.value.name);
  if (newName && newName.trim() !== "") {
    currentList.value.name = newName.trim();

    const savedLists = localStorage.getItem("lists");
    if (savedLists) {
      const lists = JSON.parse(savedLists);
      const listIndex = lists.findIndex((l) => l.id === currentList.value.id);
      if (listIndex !== -1) {
        lists[listIndex].name = newName.trim();
        localStorage.setItem("lists", JSON.stringify(lists));
      }
    }

    showListMenuModal.value = false;
  }
};

const deleteList = () => {
  if (currentList.value.id === "main-list") {
    alert("Cannot delete the default list");
    return;
  }

  if (confirm("Are you sure you want to delete this list?")) {
    localStorage.removeItem(`tasks_${currentList.value.id}`);

    const savedLists = localStorage.getItem("lists");
    if (savedLists) {
      const lists = JSON.parse(savedLists);
      const filteredLists = lists.filter((l) => l.id !== currentList.value.id);
      localStorage.setItem("lists", JSON.stringify(filteredLists));
    }

    router.push("/");
  }
};
</script>

<style scoped>
.task-list-enter-active {
  transition: all 0.3s ease-out;
}

.task-list-leave-active {
  transition: all 0.3s ease-in;
}

.task-list-move {
  transition: transform 0.3s ease;
}

.task-list-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.task-list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.suspense-loading-enter-active {
  transition: all 0.5s ease-out;
}

.suspense-loading-leave-active {
  transition: all 0.3s ease-in;
}

.suspense-loading-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.suspense-loading-leave-to {
  opacity: 0;
  transform: scale(1.1);
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

.suspense-content-enter-active {
  transition: all 0.5s ease-out;
}

.suspense-content-enter-from {
  opacity: 0;
  transform: translateY(20px);
}
</style>

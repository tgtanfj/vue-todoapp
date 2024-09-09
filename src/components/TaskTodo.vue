<template>
  <div class="container">
    <div class="task">
      <!-- title -->
      <div class="title">
        <h1>To Do List</h1>
      </div>
      <!-- form -->
      <div class="form relative">
        <input class="py-[15px] pl-[15px] pr-[45%]" type="text" placeholder="New Task" v-model="newTask" @keyup.enter="addTask" />
        <div class="flex flex-wrap justify-center gap-6 absolute top-5 right-0">
          <button class="flex gap-1 items-center">
            <div
              class="relative group button w-20 h-8 bg-transparent rounded-lg cursor-pointer select-none mt-2 flex items-center justify-center"
              :class="{
                'border-2 border-dashed bg-transparent hover:text-red-400 border-red-400 w-20':
                  newTag === 'urgent',
                'border-2 border-dashed bg-transparent hover:text-green-400 border-green-400 w-24':
                  newTag === 'not urgent',
                'border-2 border-dashed bg-transparent hover:text-yellow-400 border-yellow-400 w-24':
                  newTag === 'high priority',
                'hover:border-[#4ec5c1] bg-transparent border-dashed border-2 w-20': newTag === ''
              }"
            >
              <p v-if="newTag === ''" class="text-sm">add tags</p>
              <p v-if="newTag === 'high priority'" class="text-sm text-yellow-400">high priority</p>
              <p v-if="newTag === 'not urgent'" class="text-sm text-green-400">not urgent</p>
              <p v-if="newTag === 'urgent'" class="text-sm text-red-400">urgent</p>
              <div
                class="absolute top-full left-0 mt-[3px] w-[120px] bg-white shadow-lg opacity-0 rounded-lg hidden group-hover:block transition-opacity group-hover:opacity-100 duration-300 z-10"
              >
                <ul class="flex flex-col items-center justify-center gap-2 p-2">
                  <li
                    v-for="tag in tags"
                    :key="tag.title"
                    @click="selectTag(tag.title)"
                    class="p-1 text-sm cursor-pointer rounded-lg text-black"
                    :class="{
                      'border-2 border-dashed hover:text-red-400 border-red-400':
                        tag.title === 'urgent',
                      'border-2 border-dashed hover:text-green-400 border-green-400':
                        tag.title === 'not urgent',
                      'border-2 border-dashed hover:text-yellow-400 border-yellow-400':
                        tag.title === 'high priority'
                    }"
                  >
                    {{ tag.title }}
                  </li>
                </ul>
              </div>
            </div>
            <div
              @click="addTask"
              class="button w-14 h-6 bg-[#4ec5c1] rounded-lg cursor-pointer select-none active:translate-y-2 active:[box-shadow:0_0px_0_0_#48b5b2,0_0px_0_0_#d4fffd] active:border-b-[0px] transition-all duration-150 [box-shadow:0_10px_0_0_#48b5b2,0_15px_0_0_#d4fffd] border-b-[1px] border-[#57deda]"
            >
              <span
                class="flex flex-col justify-center items-center h-full text-white font-bold text-sm"
                >add</span
              >
            </div>
          </button>
        </div>
      </div>
      <!-- task lists -->
      <div class="taskItems">
        <p v-if="tasks.length === 0" class="text-center text-slate-400">
          The task list is empty, please add a task now
        </p>
        <div
          @mouseenter="hovering = true"
          @mouseleave="hovering = false"
          v-if="tasks.length === 0"
          class="group mt-4 w-full flex items-center justify-center"
        >
          <SmilePlus
            size="50"
            v-if="!hovering"
            class="text-center text-slate-400 group-hover:text-[#4ec5c1] group-hover:scale-110 transition-all"
          />
          <Laugh
            size="50"
            v-else
            class="text-center text-slate-400 group-hover:text-[#4ec5c1] group-hover:scale-110 transition-all"
          />
        </div>
        <ul v-if="tasks.length > 0" class="h-full max-h-[250px] overflow-y-scroll pr-4">
          <li
            v-for="(task, index) in tasks"
            :key="task.id"
            class="flex items-center justify-between gap-1"
          >
            <button
              @click="toggleTask(index)"
              v-if="task.isEditing === false"
              class="truncate flex-[0.9] text-left"
              :class="{ 'line-through text-gray-500': task.completed }"
            >
              {{ task.title }}
            </button>
            <input
              v-if="task.isEditing"
              v-model="task.tempTitle"
              @keyup.enter="saveEdit(task, index)"
              class="border-b-2 border-[#4ec5c1] outline-none focus:outline-none"
            />
            <div class="flex items-center gap-4 cursor-default">
              <div v-if="task.isEditing === false" class="h-[26px]">
                <p
                  v-if="task.tag === ''"
                  class="text-xs border-dashed border-2 rounded-lg p-1 self-end"
                >
                  no tag
                </p>
                <p
                  v-if="task.tag === 'urgent'"
                  class="text-xs border-dashed border-2 hover:text-red-400 border-red-400 rounded-lg p-1 self-end"
                >
                  urgent
                </p>
                <p
                  v-if="task.tag === 'high priority'"
                  class="text-xs border-dashed border-2 hover:text-yellow-400 border-yellow-400 rounded-lg p-1 self-end"
                >
                  high priority
                </p>
                <p
                  v-if="task.tag === 'not urgent'"
                  class="text-xs border-dashed border-2 hover:text-green-400 border-green-400 rounded-lg p-1 self-end"
                >
                  not urgent
                </p>
              </div>
              <div v-if="task.isEditing">
                <select
                  v-model="task.tempTag"
                  :class="{
                    'border-red-400 ': task.tempTag === 'urgent',
                    'border-green-400 ': task.tempTag === 'not urgent',
                    'border-yellow-400 ': task.tempTag === 'high priority',
                    'border-gray-300': task.tempTag === '' // border mặc định
                  }"
                  class="text-xs h-[26px] border-dashed border-2 outline-none focus:outline-none rounded-lg"
                >
                  <option v-for="tag in tags" :key="tag.title" :value="tag.title">
                    {{ tag.title }}
                  </option>
                </select>
              </div>
              <button @click="removeTask(index)"><i class="far fa-trash-alt"></i></button>
              <Pen
                v-if="task.isEditing === false"
                @click="editTask(task, index)"
                size="16px"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
              <Check
                @click="saveEdit(task, index)"
                v-if="task.isEditing === true"
                size="16"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
            </div>
          </li>
        </ul>
      </div>
      <div class="clearBtns">
        <button @click="clearCompleted">Clear completed</button>
        <button @click="clearAll">Clear all</button>
      </div>
      <div class="pendingTasks">
        <span>Pending Tasks: {{ pendingTasks }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import confetti from 'canvas-confetti'
import { Check, Laugh, Pen, SmilePlus } from 'lucide-vue-next'
import { computed, onMounted, ref } from 'vue'

const tags = ref([{ title: 'urgent' }, { title: 'not urgent' }, { title: 'high priority' }])
const tasks = ref([])
const newTask = ref('')
const newTag = ref('')

const hovering = ref(false)

onMounted(() => {
  if (localStorage.getItem('tasks')) {
    tasks.value = JSON.parse(localStorage.getItem('tasks'))
    return
  }
  return
})

const editTask = (task, index) => {
  tasks.value[index].tempTitle = task.title
  tasks.value[index].tempTag = task.tag
  tasks.value[index].isEditing = true
}

const saveEdit = (task, index) => {
  if (task.tempTitle.trim() !== '') {
    tasks.value[index].title = task.tempTitle.trim()
    localStorage.setItem('tasks', JSON.stringify(tasks.value))
  }
  tasks.value[index].tag = task.tempTag
  tasks.value[index].isEditing = false

  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}

const selectTag = (tag) => {
  newTag.value = tag
}

const toggleTask = (index) => {
  tasks.value[index].completed = !tasks.value[index].completed

  localStorage.setItem('tasks', JSON.stringify(tasks.value))

  if (allTasksCompleted.value) {
    fireConfetti();
  }
}

const addTask = () => {
  if (newTask.value.trim() === '') return

  tasks.value.push({
    id: tasks.value.length + 1,
    title: newTask.value.trim(),
    completed: false,
    tag: newTag.value !== '' ? newTag.value : '',
    isEditing: false
  })

  newTask.value = ''
  newTag.value = ''

  localStorage.setItem('tasks', JSON.stringify(tasks.value))

  if (allTasksCompleted.value) {
    fireConfetti();
  }
}

const removeTask = (index) => {
  tasks.value.splice(index, 1)

  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}

const clearCompleted = () => {
  tasks.value = tasks.value.filter((task) => !task.completed)

  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}

const clearAll = () => {
  tasks.value.splice(0, tasks.value.length)

  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}

const allTasksCompleted = computed(() => {
  return tasks.value.length > 0 && tasks.value.every(task => task.completed);
});

const pendingTasks = computed(() => tasks.value.filter((task) => !task.completed).length)

const fireConfetti = () => {
  confetti({
  particleCount: 400,
  spread: 400,
  origin: { y: 0.6 }
});
}
</script>

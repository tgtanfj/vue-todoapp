<template>
  <div class="container">
    <div class="task">
      <!-- title -->
      <div class="title">
        <h1>To Do List</h1>
      </div>
      <!-- form -->
      <div class="form relative">
        <input
          ref="inputRef"
          v-show="!isSearching"
          class="py-[15px] pl-[15px] pr-[45%]"
          type="text"
          placeholder="New Task (ctrl + b to search)"
          v-model="newTask"
          @keyup.enter="addTask"
        />
        <input
          ref="searchInputRef"
          v-show="isSearching"
          class="py-[15px] pl-[15px] pr-[35%]"
          type="text"
          placeholder="Search Task (ctrl + b to end search)"
          v-model="searchTask"
          @keyup.enter="addTask"
        />
        <div class="flex flex-wrap justify-center gap-6 absolute top-5 right-0">
          <button class="flex gap-1 items-center">
            <div
              v-if="!isSearching"
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
              v-if="!isSearching"
              class="button w-14 h-6 bg-[#4ec5c1] rounded-lg cursor-pointer select-none active:translate-y-2 active:[box-shadow:0_0px_0_0_#48b5b2,0_0px_0_0_#d4fffd] active:border-b-[0px] transition-all duration-150 [box-shadow:0_10px_0_0_#48b5b2,0_15px_0_0_#d4fffd] border-b-[1px] border-[#57deda]"
            >
              <span
                class="flex flex-col justify-center items-center h-full text-white font-bold text-sm"
                >add</span
              >
            </div>
            <div class="flex items-center gap-2">
              <div
                v-if="isSearching"
                @click="toggleSearch"
                class="button w-28 h-6 bg-[#4ec5c1] rounded-lg cursor-pointer select-none active:translate-y-2 active:[box-shadow:0_0px_0_0_#48b5b2,0_0px_0_0_#d4fffd] active:border-b-[0px] transition-all duration-150 [box-shadow:0_10px_0_0_#48b5b2,0_15px_0_0_#d4fffd] border-b-[1px] border-[#57deda]"
              >
                <span
                  class="flex flex-col justify-center items-center h-full text-white font-bold text-sm"
                  >close search</span
                >
              </div>
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
        <ul
          v-show="!isSearching && tasks.length > 0"
          class="h-full max-h-[250px] overflow-y-scroll pr-4"
        >
          <li v-for="task in tasks" :key="task.id" class="flex items-center justify-between gap-1">
            <button
              @click="toggleTask(task.id)"
              v-if="task.isEditing === false"
              class="truncate flex-[0.9] text-left"
              :class="{ 'line-through text-gray-500': task.completed }"
            >
              {{ task.title }}
            </button>
            <input
              v-if="task.isEditing"
              v-model="task.tempTitle"
              @keyup.enter="saveEdit(task.id)"
              class="border-b-2 border-[#4ec5c1] outline-none focus:outline-none flex-[0.9]"
            />
            <div class="flex items-center gap-3 cursor-default">
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
                    'border-gray-300': task.tempTag === ''
                  }"
                  class="text-xs h-[26px] border-dashed border-2 outline-none focus:outline-none rounded-lg"
                >
                  <option v-for="tag in tags" :key="tag.title" :value="tag.title">
                    {{ tag.title }}
                  </option>
                </select>
              </div>
              <button @click="removeTask(task.id)"><i class="far fa-trash-alt"></i></button>
              <Pen
                v-if="task.isEditing === false"
                @click="editTask(task.id)"
                size="16px"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
              <Check
                @click="saveEdit(task.id)"
                v-if="task.isEditing === true"
                size="18"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
              <Star
                v-if="task.isStarred === false"
                @click="toggleStar(task.id)"
                size="18"
                class="hover:text-yellow-300 text-black hover:fill-[#f8df56] transition-all cursor-pointer"
                :class="{'star-effect': effectTaskId === task.id}"
              />
              <Star
                v-if="task.isStarred === true"
                @click="toggleStar(task.id)"
                size="18"
                class="text-yellow-300 fill-[#f8df56] transition-all cursor-pointer"
                :class="{'star-effect': effectTaskId === task.id}"
              />
              <Dialog>
                <DialogTrigger
                  ><Info
                    size="16"
                    class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
                /></DialogTrigger>
                <DialogContent>
                  <DialogHeader>
                    <DialogTitle class="text-[#4ec5c1]">Task Information</DialogTitle>
                    <DialogDescription>Information of a specific task</DialogDescription>
                  </DialogHeader>
                  <p class="w-full truncate flex items-center gap-2">
                    <TypeOutline size="18" class="hover:text-[#4ec5c1]" />
                    <span class="truncate">Task Title: {{ task.title }}</span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Tag size="16" class="hover:text-[#4ec5c1]" />Task Tag:
                    <span
                      :class="{
                        'border-red-400 hover:text-red-400': task.tag === 'urgent',
                        'border-green-400 hover:text-green-400': task.tag === 'not urgent',
                        'border-yellow-400 hover:text-yellow-400': task.tag === 'high priority',
                        'border-gray-300': task.tag === ''
                      }"
                      class="p-1 border-dashed border-2 outline-none focus:outline-none rounded-lg"
                    >
                      {{ task.tag === '' ? 'no tag' : task.tag }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <ListTodo size="18" class="hover:text-[#4ec5c1]" />Task Status:
                    <span
                      :class="{
                        'text-green-400': task.completed === true,
                        'text-red-400': task.completed === false
                      }"
                    >
                      {{ task.completed === true ? 'Completed' : 'Not Completed' }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Bookmark size="18" class="hover:text-[#4ec5c1]" />Starred:
                    <Star v-if="task.isStarred === false" size="18" class="text-black" :class="{'star-effect': effectTaskId === task.id}"/>
                    <Star
                      v-if="task.isStarred === true"
                      size="18"
                      class="text-yellow-300 fill-[#f8df56] transition-all"
                      :class="{'star-effect': effectTaskId === task.id}"
                    />
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Clock size="18" class="hover:text-[#4ec5c1]" />Created At:
                    <span>
                      {{ formatDate(task.createdAt) }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <CalendarClock size="18" class="hover:text-[#4ec5c1]" />Updated At:
                    <span>
                      {{
                        task.updatedAt !== null
                          ? formatDate(task.updatedAt)
                          : 'Task has not been updated yet'
                      }}
                    </span>
                  </p>
                  <DialogFooter>
                    <DialogClose class="hover:text-[#4ec5c1]">Close</DialogClose>
                  </DialogFooter>
                </DialogContent>
              </Dialog>
            </div>
          </li>
        </ul>
        <ul
          v-show="isSearching && searchedTasks.length > 0"
          class="h-full max-h-[250px] overflow-y-scroll pr-4"
        >
          <li
            v-for="task in searchedTasks"
            :key="task.id"
            class="flex items-center justify-between gap-1"
          >
            <button
              @click="toggleTask(task.id)"
              v-if="task.isEditing === false"
              class="truncate flex-[0.9] text-left"
              :class="{ 'line-through text-gray-500': task.completed }"
            >
              {{ task.title }}
            </button>
            <input
              v-if="task.isEditing"
              v-model="task.tempTitle"
              @keyup.enter="saveEdit(task.id)"
              class="border-b-2 border-[#4ec5c1] outline-none focus:outline-none flex-[0.9]"
            />
            <div class="flex items-center gap-3 cursor-default">
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
                    'border-gray-300': task.tempTag === ''
                  }"
                  class="text-xs h-[26px] border-dashed border-2 outline-none focus:outline-none rounded-lg"
                >
                  <option v-for="tag in tags" :key="tag.title" :value="tag.title">
                    {{ tag.title }}
                  </option>
                </select>
              </div>
              <button @click="removeTask(task.id)"><i class="far fa-trash-alt"></i></button>
              <Pen
                v-if="task.isEditing === false"
                @click="editTask(task.id)"
                size="16px"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
              <Check
                @click="saveEdit(task.id)"
                v-if="task.isEditing === true"
                size="18"
                class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
              />
              <Star
                v-if="task.isStarred === false"
                @click="toggleStar(task.id)"
                size="18"
                class="hover:text-yellow-300 text-black hover:fill-[#f8df56] transition-all cursor-pointer"
                :class="{'star-effect': effectTaskId === task.id}"
              />
              <Star
                v-if="task.isStarred === true"
                @click="toggleStar(task.id)"
                size="18"
                class="text-yellow-300 fill-[#f8df56] transition-all cursor-pointer"
                :class="{'star-effect': effectTaskId === task.id}"
              />
              <Dialog>
                <DialogTrigger
                  ><Info
                    size="16"
                    class="hover:text-[#4ec5c1] transition-all duration-300 cursor-pointer"
                /></DialogTrigger>
                <DialogContent>
                  <DialogHeader>
                    <DialogTitle class="text-[#4ec5c1]">Task Information</DialogTitle>
                    <DialogDescription>Information of a specific task</DialogDescription>
                  </DialogHeader>
                  <p class="w-full truncate flex items-center gap-2">
                    <TypeOutline size="18" class="hover:text-[#4ec5c1]" />
                    <span class="truncate">Task Title: {{ task.title }}</span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Tag size="16" class="hover:text-[#4ec5c1]" />Task Tag:
                    <span
                      :class="{
                        'border-red-400 hover:text-red-400': task.tag === 'urgent',
                        'border-green-400 hover:text-green-400': task.tag === 'not urgent',
                        'border-yellow-400 hover:text-yellow-400': task.tag === 'high priority',
                        'border-gray-300': task.tag === ''
                      }"
                      class="p-1 border-dashed border-2 outline-none focus:outline-none rounded-lg"
                    >
                      {{ task.tag === '' ? 'no tag' : task.tag }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <ListTodo size="18" class="hover:text-[#4ec5c1]" />Task Status:
                    <span
                      :class="{
                        'text-green-400': task.completed === true,
                        'text-red-400': task.completed === false
                      }"
                    >
                      {{ task.completed === true ? 'Completed' : 'Not Completed' }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Bookmark size="18" class="hover:text-[#4ec5c1]" />Starred:
                    <Star v-if="task.isStarred === false" size="18" class="text-black" 
                    :class="{'star-effect': effectTaskId === task.id}"/>
                    <Star
                      v-if="task.isStarred === true"
                      size="18"
                      class="text-yellow-300 fill-[#f8df56] transition-all"
                      :class="{'star-effect': effectTaskId === task.id}"
                    />
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <Clock size="18" class="hover:text-[#4ec5c1]" />Created At:
                    <span>
                      {{ formatDate(task.createdAt) }}
                    </span>
                  </p>
                  <p class="cursor-default flex items-center gap-2">
                    <CalendarClock size="18" class="hover:text-[#4ec5c1]" />Updated At:
                    <span>
                      {{
                        task.updatedAt !== null
                          ? formatDate(task.updatedAt)
                          : 'Task has not been updated yet'
                      }}
                    </span>
                  </p>
                  <DialogFooter>
                    <DialogClose class="hover:text-[#4ec5c1]">Close</DialogClose>
                  </DialogFooter>
                </DialogContent>
              </Dialog>
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
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger
} from '@/components/ui/dialog'
import confetti from 'canvas-confetti'
import {
  Bookmark,
  CalendarClock,
  Check,
  Clock,
  Info,
  Laugh,
  ListTodo,
  Pen,
  SmilePlus,
  Star,
  Tag,
  TypeOutline
} from 'lucide-vue-next'
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue'
import DialogClose from './ui/dialog/DialogClose.vue'
import moment from 'moment'
import debounce from 'lodash/debounce'
import { v4 as uuidv4 } from 'uuid'

const tags = ref([{ title: 'urgent' }, { title: 'not urgent' }, { title: 'high priority' }])
const tasks = ref([])
const searchedTasks = ref([])
const newTask = ref('')
const searchTask = ref('')
const newTag = ref('')
const isSearching = ref(false)
const hovering = ref(false)

const searchInputRef = ref(null)
const inputRef = ref(null)

onMounted(() => {
  if (localStorage.getItem('tasks')) {
    tasks.value = JSON.parse(localStorage.getItem('tasks'))
    return
  }
  return
})

const handleKeyPress = (event) => {
  if (event.ctrlKey && event.key === 'b') {
    event.preventDefault()
    toggleSearch()
  }
}

const toggleSearch = () => {
  isSearching.value = !isSearching.value
  if (!isSearching.value) {
    searchedTasks.value = tasks.value
    nextTick(() => {
      inputRef.value.focus()
    })
  } else {
    searchTasks()
    nextTick(() => {
      searchInputRef.value.focus()
    })
  }
  // searchTask.value = ''
}

const searchTasks = () => {
  if (searchTask.value.trim() === '') {
    searchedTasks.value = tasks.value
  } else {
    searchedTasks.value = tasks.value.filter((task) =>
      task.title.toLowerCase().includes(searchTask.value.toLowerCase())
    )
  }
}

const debouncedSearch = debounce(searchTasks, 300)

watch(searchTask, () => {
  debouncedSearch()
})

onMounted(() => {
  searchedTasks.value = tasks.value
})

onMounted(() => {
  window.addEventListener('keydown', handleKeyPress)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyPress)
})
const effectTaskId = ref(null);
const toggleStar = (taskId) => {
  const taskIndex = tasks.value.findIndex((task) => task.id === taskId)

  effectTaskId.value = taskId;

// Remove effect class after 1 second
setTimeout(() => {
  effectTaskId.value = null;
}, 1000);

  if (taskIndex !== -1) {
    tasks.value[taskIndex].isStarred = !tasks.value[taskIndex].isStarred
    tasks.value[taskIndex].updatedAt = new Date()
    tasks.value.sort((a, b) => b.isStarred - a.isStarred)

    localStorage.setItem('tasks', JSON.stringify(tasks.value))
  }
}

const editTask = (taskId) => {
  const task = tasks.value.find((task) => task.id === taskId)
  if (task) {
    task.tempTitle = task.title
    task.tempTag = task.tag
    task.isEditing = true
  }
}

function formatDate(date) {
  return moment(date).format('LLLL')
}

const saveEdit = (taskId) => {
  const task = tasks.value.find((task) => task.id === taskId)
  if (task) {
    if (task.tempTitle.trim() !== '') {
      task.title = task.tempTitle.trim()
    }
    task.tag = task.tempTag
    task.isEditing = false
    task.updatedAt = new Date()

    localStorage.setItem('tasks', JSON.stringify(tasks.value))
  }
}

const selectTag = (tag) => {
  newTag.value = tag
}

const toggleTask = (id) => {
  const task = tasks.value.find((task) => task.id === id)

  if (task) {
    task.completed = !task.completed
    task.updatedAt = new Date()

    localStorage.setItem('tasks', JSON.stringify(tasks.value))

    if (allTasksCompleted.value) {
      fireConfetti()
    }
  }
}

const addTask = () => {
  if (newTask.value.trim() === '') return

  tasks.value.push({
    id: uuidv4(),
    title: newTask.value.trim(),
    completed: false,
    tag: newTag.value !== '' ? newTag.value : '',
    isEditing: false,
    createdAt: new Date(),
    updatedAt: null,
    isStarred: false
  })

  newTask.value = ''
  newTag.value = ''

  localStorage.setItem('tasks', JSON.stringify(tasks.value))

  if (allTasksCompleted.value) {
    fireConfetti()
  }
}

const removeTask = (id) => {
  const index = tasks.value.findIndex((task) => task.id === id)
  const index2 = searchedTasks.value.findIndex((task) => task.id === id)

  if (index !== -1) {
    tasks.value.splice(index, 1)
    searchedTasks.value.splice(index2, 1)

    localStorage.setItem('tasks', JSON.stringify(tasks.value))
  }
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
  return tasks.value.length > 0 && tasks.value.every((task) => task.completed)
})

const pendingTasks = computed(() => tasks.value.filter((task) => !task.completed).length)

const fireConfetti = () => {
  confetti({
    particleCount: 400,
    spread: 400,
    origin: { y: 0.6 }
  })
}
</script>

<style scoped>
@keyframes star-pulse {
  0% {
    transform: scale(1);
    filter: brightness(1) drop-shadow(0 0 0 rgba(255, 215, 0, 0));
  }
  50% {
    transform: scale(1.2);
    filter: brightness(1.5) drop-shadow(0 0 20px rgba(255, 215, 0, 0.7));
  }
  100% {
    transform: scale(1);
    filter: brightness(1) drop-shadow(0 0 0 rgba(255, 215, 0, 0));
  }
}

.star-effect {
  animation: star-pulse 0.6s ease-out;
}
</style>
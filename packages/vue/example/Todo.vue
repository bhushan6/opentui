<script setup lang="ts">
import { ref, computed, onUnmounted } from "vue"
import { getKeyHandler, type InputRenderable, type ParsedKey } from "@opentui/core"

interface Todo {
  id: number
  text: string
  completed: boolean
}

const todos = ref<Todo[]>([
  { id: 1, text: "Learn OpenTUI", completed: true },
  { id: 2, text: "Build a cool TUI", completed: false },
  { id: 3, text: "Profit!", completed: false },
])

const newTodoText = ref("")
const selectedIndex = ref(0)
const focusedElement = ref<"input" | "list">("input")
const editingTodo = ref<Todo | null>(null)
const editingTodoText = ref("")

const todoOptions = computed(() => {
  return todos.value.map((todo) => {
    const prefix = todo.completed ? "[x]" : "[ ]"
    return {
      name: `${prefix} ${todo.text}`,
      value: todo.id,
    }
  })
})

const onInput = (text: string) => {
  newTodoText.value = text
}

const addTodo = () => {
  if (newTodoText.value.trim() === "") return
  todos.value.push({
    id: Date.now(),
    text: newTodoText.value,
    completed: false,
  })
  newTodoText.value = ""
}

const onHighlight = (index: number) => {
  selectedIndex.value = index
}

const saveEdit = () => {
  if (editingTodo.value) {
    if (editingTodoText.value.trim() === "") {
      const index = todos.value.findIndex((todo) => todo.id === editingTodo.value!.id)
      if (index !== -1) {
        todos.value.splice(index, 1)
      }
    } else {
      editingTodo.value.text = editingTodoText.value
    }
    editingTodo.value = null
  }
}

const handleKeyPress = (key: ParsedKey) => {
  if (editingTodo.value) {
    if (key.name === "escape") {
      editingTodo.value = null
    }
    return
  }

  if (key.name === "tab") {
    focusedElement.value = focusedElement.value === "input" ? "list" : "input"
    return
  }

  if (focusedElement.value === "list") {
    const selectedTodo = todos.value[selectedIndex.value]
    if (!selectedTodo) return

    if (key.name === "return") {
      editingTodo.value = selectedTodo
      editingTodoText.value = selectedTodo.text
    }

    if (key.name === "space") {
      selectedTodo.completed = !selectedTodo.completed
    }

    if (key.name === "backspace" || key.name === "delete") {
      todos.value.splice(selectedIndex.value, 1)
      if (selectedIndex.value >= todos.value.length) {
        selectedIndex.value = todos.value.length - 1
      }
    }
  }
}

getKeyHandler().on("keypress", handleKeyPress)

onUnmounted(() => {
  getKeyHandler().off("keypress", handleKeyPress)
})

const boxStyles = {
  width: "100%",
  flexGrow: 1,
}

const inputStyles = {
  flexGrow: 1,
}
</script>

<template>
  <boxRenderable title="Todo App" :style="{ flexDirection: 'column', flexGrow: 1 }">
    <boxRenderable :style="{ height: 3, flexDirection: 'row' }">
      <inputRenderable
        placeholder="Enter a new todo..."
        :value="newTodoText"
        :onInput="onInput"
        :onSubmit="addTodo"
        :style="inputStyles"
        :focused="focusedElement === 'input'"
      />
    </boxRenderable>

    <boxRenderable v-if="editingTodo" :style="{ height: 3, flexDirection: 'row' }">
      <inputRenderable
        :value="editingTodoText"
        :onInput="(text) => (editingTodoText = text)"
        :onSubmit="saveEdit"
        :style="inputStyles"
        :focused="true"
      />
    </boxRenderable>

    <selectRenderable
      v-else
      :options="todoOptions"
      :onChange="onHighlight"
      :style="boxStyles"
      :focused="focusedElement === 'list'"
      showScrollIndicator
    />

    <boxRenderable :style="{ height: 2, paddingTop: 1 }">
      <textRenderable content="[space] to toggle, [delete] to remove" />
    </boxRenderable>
  </boxRenderable>
</template>

<template>
    <div class="todo-list">
        <div class="add-todo">
            <input v-model="newTodo" @keyup.enter="addTodo" placeholder="添加新任务..." type="text" />
            <button @click="addTodo">添加</button>
        </div>

        <div v-if="todos.length === 0" class="empty-state">
            还没有任务，添加一个吧！
        </div>

        <div v-else class="todos">
            <todo-item v-for="todo in todos" :key="todo.id" :todo="todo" @toggle-complete="toggleComplete"
                @toggle-timer="toggleTimer" @delete-todo="deleteTodo" />
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import TodoItem from './TodoItem.vue'

// 任务列表状态
const todos = ref([])
const newTodo = ref('')

// 从localStorage加载数据
onMounted(() => {
    const savedTodos = localStorage.getItem('time-tracking-todos')
    if (savedTodos) {
        todos.value = JSON.parse(savedTodos)
    }
})

// 保存到localStorage
const saveTodos = () => {
    localStorage.setItem('time-tracking-todos', JSON.stringify(todos.value))
}

// 添加新任务
const addTodo = () => {
    if (newTodo.value.trim()) {
        const todo = {
            id: Date.now(),
            text: newTodo.value,
            completed: false,
            timeRecords: [],
            isTracking: false,
            startTime: null,
            totalTime: 0 // 总计时间（毫秒）
        }
        todos.value.push(todo)
        newTodo.value = ''
        saveTodos()
    }
}

// 切换任务完成状态
const toggleComplete = (id) => {
    const todo = todos.value.find(t => t.id === id)
    if (todo) {
        // 如果任务正在计时且设为完成，停止计时
        if (todo.isTracking && !todo.completed) {
            toggleTimer(id)
        }
        todo.completed = !todo.completed
        saveTodos()
    }
}

// 切换计时状态
const toggleTimer = (id) => {
    const todo = todos.value.find(t => t.id === id)
    if (!todo) return

    if (todo.isTracking) {
        // 停止计时
        const endTime = new Date()
        const duration = endTime - todo.startTime

        // 添加一条计时记录
        todo.timeRecords.push({
            start: todo.startTime,
            end: endTime,
            duration
        })

        // 更新总时间
        todo.totalTime += duration
        todo.isTracking = false
        todo.startTime = null
    } else {
        // 开始计时
        todo.isTracking = true
        todo.startTime = new Date()
    }

    saveTodos()
}

// 删除任务
const deleteTodo = (id) => {
    todos.value = todos.value.filter(t => t.id !== id)
    saveTodos()
}
</script>

<style scoped>
.todo-list {
    margin-top: 20px;
}

.add-todo {
    display: flex;
    margin-bottom: 20px;
}

.add-todo input {
    flex: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-right: 10px;
}

.add-todo button {
    padding: 10px 15px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.empty-state {
    text-align: center;
    color: #666;
    padding: 20px;
    border: 1px dashed #ddd;
    border-radius: 4px;
}

.todos {
    margin-top: 20px;
}
</style>
<template>
    <div class="todo-list">
        <div class="add-todo">
            <input v-model="newTodo" @keyup.enter="addTodo" placeholder="添加新任务..." type="text" />
            <button @click="addTodo">添加</button>
        </div>

        <task-stats :todos="todos" />

        <div class="todo-filters">
            <button @click="filter = 'all'" :class="{ active: filter === 'all' }">
                全部
            </button>
            <button @click="filter = 'active'" :class="{ active: filter === 'active' }">
                进行中
            </button>
            <button @click="filter = 'completed'" :class="{ active: filter === 'completed' }">
                已完成
            </button>
        </div>

        <div v-if="filteredTodos.length === 0" class="empty-state">
            暂无任务显示
        </div>

        <div v-else class="todos">
            <todo-item v-for="todo in filteredTodos" :key="todo.id" :todo="todo" @toggle-complete="toggleComplete"
                @toggle-timer="toggleTimer" @delete-todo="deleteTodo" />
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import TodoItem from './TodoItem.vue'
import TaskStats from './TaskStats.vue'

// 任务列表状态
const todos = ref([])
const newTodo = ref('')
const filter = ref('all') // 过滤器：'all', 'active', 'completed'

// 过滤后的任务列表
const filteredTodos = computed(() => {
    switch (filter.value) {
        case 'active':
            return todos.value.filter(todo => !todo.completed)
        case 'completed':
            return todos.value.filter(todo => todo.completed)
        default:
            return todos.value
    }
})

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

.todo-filters {
    display: flex;
    margin-bottom: 15px;
}

.todo-filters button {
    flex: 1;
    padding: 8px;
    background-color: #f1f1f1;
    border: 1px solid #ddd;
    cursor: pointer;
}

.todo-filters button:first-child {
    border-radius: 4px 0 0 4px;
}

.todo-filters button:last-child {
    border-radius: 0 4px 4px 0;
}

.todo-filters button.active {
    background-color: #2196F3;
    color: white;
    border-color: #2196F3;
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
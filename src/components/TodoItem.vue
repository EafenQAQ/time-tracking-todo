<template>
    <div class="todo-item" :class="{ completed: todo.completed, tracking: todo.isTracking }">
        <div class="todo-content">
            <input type="checkbox" :checked="todo.completed" @change="$emit('toggle-complete', todo.id)" />
            <span class="todo-text">{{ todo.text }}</span>
        </div>

        <div class="todo-actions">
            <div class="timer-display">
                {{ formatTime(todo.totalTime + (todo.isTracking ? Date.now() - todo.startTime : 0)) }}
            </div>

            <button @click="$emit('toggle-timer', todo.id)" :class="['timer-btn', { 'timer-running': todo.isTracking }]"
                :disabled="todo.completed">
                {{ todo.isTracking ? '停止' : '开始' }}
            </button>

            <button class="delete-btn" @click="$emit('delete-todo', todo.id)">
                删除
            </button>
        </div>
    </div>

    <div v-if="showDetails" class="time-records">
        <h4>时间记录</h4>
        <div v-if="todo.timeRecords.length === 0">
            还没有记录
        </div>
        <ul v-else>
            <li v-for="(record, index) in todo.timeRecords" :key="index">
                <div>开始: {{ formatDateTime(record.start) }}</div>
                <div>结束: {{ formatDateTime(record.end) }}</div>
                <div>持续: {{ formatTime(record.duration) }}</div>
            </li>
        </ul>
    </div>
</template>

<script setup>
import { defineProps, defineEmits, ref } from 'vue'

const props = defineProps({
    todo: {
        type: Object,
        required: true
    }
})

defineEmits(['toggle-complete', 'toggle-timer', 'delete-todo'])

const showDetails = ref(false)

// 格式化时间显示 (毫秒 -> HH:MM:SS)
const formatTime = (ms) => {
    if (!ms) return '00:00:00'

    const totalSeconds = Math.floor(ms / 1000)
    const hours = Math.floor(totalSeconds / 3600)
    const minutes = Math.floor((totalSeconds % 3600) / 60)
    const seconds = totalSeconds % 60

    return [hours, minutes, seconds]
        .map(v => v < 10 ? '0' + v : v)
        .join(':')
}

// 格式化日期时间
const formatDateTime = (date) => {
    if (!date) return ''

    const d = new Date(date)
    return `${d.toLocaleDateString()} ${d.toLocaleTimeString()}`
}
</script>

<style scoped>
.todo-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-bottom: 10px;
    background-color: #f9f9f9;
}

.todo-item.completed {
    background-color: #e8e8e8;
    opacity: 0.8;
}

.todo-item.tracking {
    border-color: #4CAF50;
    background-color: #e8f5e9;
}

.todo-content {
    display: flex;
    align-items: center;
}

.todo-text {
    margin-left: 10px;
    font-size: 16px;
}

.completed .todo-text {
    text-decoration: line-through;
    color: #999;
}

.todo-actions {
    display: flex;
    align-items: center;
}

.timer-display {
    font-family: monospace;
    font-size: 16px;
    margin-right: 10px;
    min-width: 80px;
    text-align: right;
}

.timer-btn {
    padding: 8px 12px;
    background-color: #2196F3;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 10px;
}

.timer-btn:disabled {
    background-color: #bbdefb;
    cursor: not-allowed;
}

.timer-running {
    background-color: #f44336;
}

.delete-btn {
    padding: 8px 12px;
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.time-records {
    background-color: #f5f5f5;
    padding: 10px;
    margin-top: -10px;
    margin-bottom: 15px;
    border: 1px solid #ddd;
    border-top: none;
    border-radius: 0 0 4px 4px;
}

.time-records h4 {
    margin-top: 0;
    margin-bottom: 10px;
}

.time-records ul {
    list-style: none;
    padding: 0;
}

.time-records li {
    border-bottom: 1px solid #eee;
    padding: 8px 0;
}
</style>
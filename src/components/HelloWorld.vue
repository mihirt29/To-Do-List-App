<template>
  <div class="app">
    <h1>To-Do List</h1>
    <div class="input-container">
      <input type="text" v-model="task" @keyup.enter="addTask" placeholder="Enter a new task..." class="task-input">
      <button class="add-button" @click="addTask">Add Task</button>
    </div>
    <div class="open-tasks">
      <h2>Open Tasks</h2>
      <ul>
        <li v-for="todo in openTasks" :key="todo.id" class="task">
          <div class="task-info">
            <span :style="{ color: taskColor(todo), fontSize: '1.2em', fontFamily: 'Arial, sans-serif' }">{{ todo.task }}</span>
            <span class="created-info">{{ formatDate(todo.createdAt) }}</span>
          </div>
          <div class="task-buttons">
            <input type="checkbox" v-model="todo.done" @change="toggleDone(todo)" class="tick-box">
            <button class="edit-button" @click="editTask(todo)">Edit</button>
            <button class="delete-button" @click="deleteTask(todo)">Delete</button>
          </div>
        </li>
      </ul>
    </div>
    <div class="completed-tasks">
      <h2>Completed Tasks</h2>
      <ul>
        <li v-for="todo in completedTasks" :key="todo.id" class="completed-task">
          <del>{{ todo.task }}</del>
          <span class="created-info">{{ formatDate(todo.createdAt) }}</span>
          <button class="delete-button" @click="deleteTask(todo)">Delete</button>
        </li>
      </ul>
    </div>
    <div v-if="editingTodo" class="edit-task-container">
      <h2>Edit Task</h2>
      <input type="text" v-model="editedTask" placeholder="Enter updated task..." class="task-input">
      <button class="update-button" @click="updateTask">Update Task</button>
      <button class="cancel-button" @click="cancelEdit">Cancel</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'HelloWorld',
  data() {
    return {
      task: '',
      editedTask: '',
      todos: [],
      editingTodo: null
    };
  },
  computed: {
    openTasks() {
      return this.todos.filter(todo => !todo.done);
    },
    completedTasks() {
      return this.todos.filter(todo => todo.done);
    }
  },
  created() {
    this.fetchTodos();
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await axios.get('http://localhost:3000/todos');
        this.todos = response.data;
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    },
    async addTask() {
      const trimmedTask = this.task.trim();
      if (!trimmedTask) return;
      try {
        const response = await axios.post('http://localhost:3000/todos', {
          task: trimmedTask,
          createdAt: new Date().toLocaleString(),
          done: false
        });
        this.todos.push(response.data);
        this.task = '';
      } catch (error) {
        console.error('Error adding task:', error);
      }
    },
    async toggleDone(todo) {
      try {
        await axios.patch(`http://localhost:3000/todos/${todo.id}`, {
          done: todo.done
        });
      } catch (error) {
        console.error('Error toggling done status:', error);
      }
    },
    async deleteTask(todo) {
      if (!confirm('Are you sure you want to delete this task?')) return;
      try {
        await axios.delete(`http://localhost:3000/todos/${todo.id}`);
        this.todos = this.todos.filter(t => t.id !== todo.id);
      } catch (error) {
        console.error('Error deleting task:', error);
      }
    },
    editTask(todo) {
      this.editingTodo = todo.id;
      this.editedTask = todo.task;
    },
    async updateTask() {
      const trimmedTask = this.editedTask.trim();
      if (!trimmedTask) {
        this.cancelEdit();
        return;
      }
      try {
        const index = this.todos.findIndex(t => t.id === this.editingTodo);
        this.todos[index].task = trimmedTask;
        this.editingTodo = null;
        this.editedTask = '';
      } catch (error) {
        console.error('Error updating task:', error);
      }
    },
    cancelEdit() {
      this.editingTodo = null;
      this.editedTask = '';
    },
    taskColor(todo) {
      return todo.done ? '#777' : '#000'; // Completed tasks will have darker text color
    },
    formatDate(dateString) {
      return new Date(dateString).toLocaleString();
    }
  }
};
</script>
<style scoped>
.app {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background-color: #b1a7d5; /* Light sea green background */
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  color: #333; /* Dark text color */
}

h1, h2 {
  text-align: center;
  color: #222; /* Darker text color */
}

.input-container {
  display: flex;
  align-items: center;
  justify-content: center; /* Center align input and button */
  margin-bottom: 20px;
}

.task-input {
  padding: 5px;
  margin-right: 10px;
  border-radius: 5px;
  border: none;
  font-size: 1.1em; /* Slightly bigger font size */
}

button {
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 15px;
  font-family: 'Arial', sans-serif;
}

.add-button {
  background-color: #76a158; /* Green */
  color: black;
}

.done-button, .edit-button, .delete-button, .update-button, .cancel-button {
  background-color: #76a158; /* Yellow */
  color: black; /* Darker text color */
}

.tick-box {
  margin-right: 10px;
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.tick-box:checked {
  background-color: #4caf50; /* Green */
}

ul {
  list-style: none;
  padding: 0;
}

.task, .completed-task {
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.task-info, .completed-task {
  display: flex;
  align-items: baseline; /* Align elements at their baselines */
}

.created-info {
  margin-left: 10px;
  font-size: 0.8em; /* Smaller font size */
}

.completed-task del {
  color: black;
}

.completed-task span {
  color: black;
}

.edit-task-container {
  margin-top: 20px;
}

.edit-task-container h2 {
  text-align: center;
}
</style>

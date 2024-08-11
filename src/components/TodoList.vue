<template>
    <div class="todo-container">
      <h1 class="title">Todo List</h1>
      <button @click="showAddModal = true" class="add-btn">Add Todo</button>
  
      <table class="todo-table">
        <thead>
          <tr>
            <th>No</th>
            <th>Todo</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="todo.id">
            <td>{{ index + 1 }}</td>
            <td>{{ todo.title }}</td>
            <td>
              <button class="edit-btn" @click="openEditModal(todo)">Edit</button>
              <button class="delete-btn" @click="deleteTodo(todo.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <button class="load-more-btn" @click="loadMore">Load More</button>
  
      <div v-if="showAddModal" class="modal-overlay" @click.self="closeAddModal">
        <div class="modal-content">
          <h2>Add Todo</h2>
          <TodoForm :todo="newTodo" :buttonText="'Add Todo'" @submit="addTodo" />
          <button class="close-btn" @click="closeAddModal">Close</button>
        </div>
      </div>
  
      <div v-if="showEditModal" class="modal-overlay" @click.self="closeEditModal">
        <div class="modal-content">
          <h2>Edit Todo</h2>
          <TodoForm :todo="newTodo" :buttonText="'Update Todo'" @submit="updateTodo" />
          <button class="close-btn" @click="closeEditModal">Close</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import TodoForm from './TodoForm.vue';
  
  export default {
    name: 'TodoList',
    components: {
      TodoForm
    },
    data() {
      return {
        todos: [],
        newTodo: { title: '' },
        page: 1,
        limit: 10,
        editIndex: -1,
        showAddModal: false,
        showEditModal: false,
      };
    },
    methods: {
      fetchTodos() {
        axios.get(`https://jsonplaceholder.typicode.com/todos?_page=${this.page}&_limit=${this.limit}`)
          .then(response => {
            this.todos = [...this.todos, ...response.data];
          });
      },
      addTodo(todo) {
        if (!todo || !todo.title || !todo.title.trim()) {
          return;
        }
        axios.post('https://jsonplaceholder.typicode.com/todos', todo)
          .then(response => {
            this.todos.unshift(response.data);
            this.newTodo = { title: '' };
            this.showAddModal = false; 
          })
          .catch(error => {
            console.error('Error during creation:', error.response || error.message);
            alert('Failed to create todo. Please try again.');
          });
      },
      openEditModal(todo) {
        this.newTodo = { ...todo };
        this.editIndex = this.todos.indexOf(todo);
        this.showEditModal = true;
      },
      updateTodo(todo) {
        if (!todo || !todo.id || !todo.title.trim()) {
          return;
        }
  
        axios.delete(`https://jsonplaceholder.typicode.com/todos/${todo.id}`)
          .then(() => {
            return axios.post('https://jsonplaceholder.typicode.com/todos', todo);
          })
          .then(response => {
            this.todos.splice(this.editIndex, 1, response.data);
            this.newTodo = { title: '' };
            this.editIndex = -1;
            this.showEditModal = false; 
          })
          .catch(error => {
            console.error('Error during update:', error.response || error.message);
            alert('Failed to update todo. Please try again.');
          });
      },
      deleteTodo(id) {
        axios.delete(`https://jsonplaceholder.typicode.com/todos/${id}`)
          .then(() => {
            this.todos = this.todos.filter(todo => todo.id !== id);
          })
          .catch(error => {
            console.error('Error during deletion:', error.response || error.message);
            alert('Failed to delete todo. Please try again.');
          });
      },
      loadMore() {
        this.page++;
        this.fetchTodos();
      },
      closeAddModal() {
        this.showAddModal = false;
        this.newTodo = { title: '' }; 
      },
      closeEditModal() {
        this.showEditModal = false;
        this.newTodo = { title: '' }; 
        this.editIndex = -1;
      }
    },
    mounted() {
      this.fetchTodos();
    }
  };
  </script>
  <style scoped>
  .todo-container {
    width: 80%;
    max-width: 900px;
    margin: 20px auto;
    background-color: #ffffff;
    padding: 20px;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  }
  
  .title {
    text-align: center;
    font-size: 2.5em;
    color: #2c3e50;
    margin-bottom: 30px;
    font-family: 'Poppins', sans-serif;
  }
  
  .todo-table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0 15px;
    margin-bottom: 20px;
  }
  
  .todo-table th, .todo-table td {
    padding: 15px;
    border: none;
    text-align: center;
    font-family: 'Roboto', sans-serif;
  }
  
  .todo-table th {
    background-color: #ecf0f1;
    color: #2c3e50;
    font-weight: 600;
    text-transform: uppercase;
  }
  
  .todo-table td {
    background-color: #f7f7f7;
    color: #34495e;
    border-radius: 12px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
  }
  
  .add-btn, .edit-btn, .delete-btn, .close-btn {
    padding: 10px 15px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 1em;
    transition: background-color 0.3s ease, color 0.3s ease;
  }
  
  .add-btn {
    background-color: #27ae60;
    color: white;
    margin-bottom: 20px;
    font-family: 'Poppins', sans-serif;
  }
  
  .edit-btn {
    background-color: #2980b9;
    color: white;
  }
  
  .delete-btn {
    background-color: #e74c3c;
    color: white;
  }
  
  .add-btn:hover, .edit-btn:hover, .delete-btn:hover, .close-btn:hover {
    opacity: 0.9;
  }
  
  .load-more-btn {
    display: block;
    width: 100%;
    padding: 12px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 1.2em;
    cursor: pointer;
    margin-top: 20px;
    font-family: 'Poppins', sans-serif;
  }
  
  .load-more-btn:hover {
    background-color: #2980b9;
  }
  
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .modal-content {
    background: white;
    padding: 25px;
    border-radius: 12px;
    width: 90%;
    max-width: 600px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  }
  
  .close-btn {
    background-color: #e74c3c;
    color: white;
    margin-top: 15px;
    display: block;
    width: 100%;
    padding: 12px;
    font-family: 'Poppins', sans-serif;
  }
  </style>
  
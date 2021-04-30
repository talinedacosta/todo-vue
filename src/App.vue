<template>
  <div id="app">
    <h1 class="title">Lista de tarefas</h1>

    <section class="todo">
      <!-- adiciona tarefa e atualiza tarefa -->
      <div class="todo-add-edit">
        <input
          type="text"
          name="todo"
          id="todo"
          v-model="newTodo"
          placeholder="O que você pretende fazer hoje?"
          required
        />
        <input
          v-if="!isEditing"
          type="submit"
          value="Adicionar"
          @click="addTodo"
        />
        <input v-else type="submit" value="Atualizar" @click="updateTodo" />
      </div>

      <!-- filtros -->
      <div class="todo-filters">
        <span>Filtrar tarefas:</span>
        <ul class="todo-filters-list">
          <li>
            <button
              @click="visibility = 'all'"
              :class="{ selected: visibility == 'all' }"
            >
              Todas
            </button>
            <button
              @click="visibility = 'active'"
              :class="{ selected: visibility == 'active' }"
            >
              Ativas
            </button>
            <button
              @click="visibility = 'completed'"
              :class="{ selected: visibility == 'completed' }"
            >
              Completadas
            </button>
          </li>
        </ul>
      </div>

      <!-- lista de tarefa -->
      <ul class="todo-list">
        <li
          class="todo-item"
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{ completed: todo.completed, editing: todo == editedTodo }"
        >
          <input
            type="checkbox"
            name="completed"
            class="todo-complete-checkbox todo-checkbox"
            v-model="todo.completed"
            @click="completeTodo(todo)"
            :disabled="editedTodo"
            title="Completar"
          />
          <p>{{ todo.text }}</p>
          <div class="todo-item-edit-delete">
            <button
              class="todo-edit-button todo-button"
              @click="editTodo(todo)"
              :disabled="todo.completed"
              :class="{ completed: todo.completed }"
            >
              Editar
            </button>
            <button
              class="todo-delete-button todo-button"
              @click="deleteTodo(todo)"
              :disabled="editedTodo"
            >
              Deletar
            </button>
          </div>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
const STORAGE_KEY = "todo-storage";

export default {
  name: "App",
  data: function () {
    return {
      isEditing: false,
      editedTodo: null,
      visibility: "all",
      newTodo: "",
      todos: [],
    };
  },
  created() {
    this.todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    this.updateTodoId();
  },
  computed: {
    filteredTodos() {
      //all
      if (this.visibility === "all") {
        return this.todos;
      }
      //active
      else if (this.visibility === "active") {
        return this.todos.filter((todo) => {
          return !todo.completed;
        });
        //completed
      } else {
        return this.todos.filter((todo) => {
          return todo.completed;
        });
      }
    },
  },
  methods: {
    findIndexTodo(id) {
      const indexTodo = this.todos.findIndex((item) => {
        return item.id === id;
      });
      return indexTodo;
    },
    addTodo() {
      if (this.newTodo.length > 0) {
        this.todos.push({
          text: this.newTodo,
          completed: false,
          id: this.todos.length,
        });
        this.newTodo = "";
        this.updateLocalStorage();
      }
    },
    editTodo(todo) {
      this.editedTodo = todo;
      this.newTodo = todo.text;
      this.isEditing = true;
    },
    updateTodo() {
      if (!this.editedTodo) {
        this.isEditing = false;
        return;
      }
      this.todos[this.editedTodo.id].text = this.newTodo;
      this.isEditing = false;
      this.newTodo = "";
      this.editedTodo = null;
      this.updateLocalStorage();
    },
    completeTodo(todo) {
      this.todos[todo.id].completed = !this.todos[todo.id].completed;
      this.updateLocalStorage();
    },
    deleteTodo(todo) {
      this.todos.splice(this.findIndexTodo(todo.id), 1);
      this.updateTodoId();
      this.updateLocalStorage();
    },
    updateLocalStorage() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.todos));
    },
    updateTodoId() {
      this.todos.map((item, index) => {
        return (item.id = index);
      });
    },
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Patrick+Hand&display=swap");

#app {
  font-family: "Patrick Hand", cursive;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.title {
  font-size: 3rem;
  color: tomato;
}

.todo {
  margin: 0 auto;
  max-width: 500px;
}

.todo-list {
  margin-top: 10px;
}
.todo-add-edit {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
  align-items: center;
}
.todo-add-edit input[type="text"] {
  border: 1px solid #efefef;
  padding: 10px;
  flex: 1 0;
}

.todo-add-edit input[type="submit"] {
  padding: 10px;
  cursor: pointer;
  border: 1px solid #efefef;
}

.todo-add-edit input[type="submit"]:hover {
  background-color: tomato;
  border-color: tomato;
}

.todo-list li {
  padding: 10px;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
  align-items: center;
}

.todo-list li:nth-of-type(even) {
  background-color: #efefef;
}
.todo-list .todo-item {
  position: relative;
}

.todo-item .todo-button:disabled {
  color: grey;
  cursor: default;
}

.todo-item .todo-button:disabled:hover {
  color: grey;
}

.todo-item-edit-delete button {
  margin: 0 5px;
}

.todo-list .todo-item p::first-letter {
  text-transform: uppercase;
}

.todo-list .todo-item.completed p {
  text-decoration: line-through;
  color: #c9c9c9;
}

.todo-list .todo-item.editing p {
  color: tomato;
}

.todo-list .todo-item.editing::before {
  content: "";
  position: absolute;
  left: 0;
  border-right: 5px solid tomato;
  height: 100%;
}

.todo-list .todo-item .todo-button.completed {
  color: #c9c9c9;
  cursor: default;
}

.todo-item .todo-complete-checkbox {
  cursor: pointer;
}

.todo-item .todo-item-edit-delete button:hover {
  color: tomato;
}

.todo-filters {
  margin-top: 10px;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
  align-items: center;
}

.todo-filters span {
  display: block;
  font-size: 13px;
}

.todo-filters .todo-filters-list li button {
  padding: 0 5px;
}

.todo-filters .todo-filters-list li button:hover {
  color: tomato;
}

.todo-filters .todo-filters-list li button.selected {
  background-color: #efefef;
}
</style>

<template>
  <div>
      <input type="text" class="todo-input" placeholder="Enter todos" 
      v-model="newTodo" @keyup.enter="addTodo">
      <transition-group name="fade" enter-active-class="animated fadeInUp"
      leave-active-class="animated fadeOutDown">
      <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
        <div class="todo-item-left">
            <input class="checkbox" type="checkbox" v-model="todo.completed">
            <div v-if="!todo.editing" @dblclick="editTodo(todo)" class="todo-item-label"
            :class="{ completed : todo.completed }"> {{ todo.title }}</div>
            <input v-else class="todo-item-edit" type="text" v-model="todo.title"
            @blur="doneEditing(todo)" @keyup.enter="doneEditing(todo)" v-focus>
        </div>
        <div class="remove-item" @click="removeTodo(index)">
            &times;
        </div>
      </div>
      </transition-group>

      <div class="extra-container">
          <div><label><input type="checkbox" :checked="!anyRemaining"
          @change="checkAllTodos"> Check All</label></div>
          <div>{{ remaining }} todos left</div>
      </div>

      <div class="extra-container">
          <div>
              <button :class="{ active: filter == 'all' }" 
              @click="filter = 'all'">All</button>
              <button :class="{ active: filter == 'active' }"
              @click="filter = 'active'">Active</button>
              <button :class="{ active: filter == 'completed' }"
              @click="filter = 'completed'">Completed</button>
          </div>

          <div>
              <transition name="fade">
                <button v-if="showClearCompletedButton" 
                @click="clearCompleted">Clear Completed</button>
              </transition>
          </div>

      </div>
  </div>
</template>

<script>

export default {
  name: 'todo-list',
  data () {
    return {
      newTodo: '',
      idForTodo: 3,
      filter: 'all',
      todos: [
          {
              'id': 1,
              'title': 'Crush coding challenge',
              'completed': false,
              'editing': false,
          },
          {
              'id': 2, 
              'title': 'Take over the world',
              'completed': false,
              'editing': false,
          }
      ]
    }
  },
  computed: {
      remaining() {
          return this.todos.filter(todo => !todo.completed).length
      },
      anyRemaining() {
          return this.remaining !== 0
      },
      todosFiltered() {
          if (this.filter == 'all') {
              return this.todos
          } else if (this.filter == 'active') {
              return this.todos.filter(todo => !todo.completed)
          } else if (this.filter == 'completed') {
              return this.todos.filter(todo => todo.completed)
          }

          return this.todos
      },
      showClearCompletedButton() {
          return this.todos.filter(todo => todo.completed).length > 0
      },
  },
  directives: {
      focus: {
          inserted: function(el) {
              el.focus()
          }
      }
  },
  mounted() {
    if (localStorage.getItem('todos')) {
      try {
        this.todos = JSON.parse(localStorage.getItem('todos'));
      } catch(e) {
        localStorage.removeItem('todos');
      }
    }
  },
  methods: {
      addTodo() {
          // To make sure we cant enter blank todos
          if (this.newTodo.trim().length == 0) {
              return
          }
        
        // To push new todos to array
          this.todos.push({
              id: this.idForTodo,
              title: this.newTodo,
              completed: false,
          })

          this.newTodo = ""
          this.idForTodo++
          this.saveTodo()
      },
      removeTodo(index) {
      this.todos.splice(index, 1)
      this.saveTodo()
    },
    editTodo(todo) {
        todo.editing = true
        this.saveTodo()
    },
    doneEditing(todo) {
        todo.editing = false
        this.saveTodo()
    },
    checkAllTodos() {
        this.todos.forEach((todo) => todo.completed = 
        event.target.checked)
    },
    clearCompleted() {
        this.todos = this.todos.filter(todo => !todo.completed)
        this.saveTodo()
    },
    saveTodo() {
      const parsed = JSON.stringify(this.todos);
      localStorage.setItem('todos', parsed);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
@import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css");

.todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    &:focus {
        outline: 0;
    }
}

.todo-item {
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    animation-duration: 0.3s;
}

.todo-item-left {
    display: flex;
}

.remove-item {
    cursor: pointer;
    margin-left: 14px;
    &:hover {
        color: red; 
    }
}

.todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;

    &:focus {
        outline: none;
    }
}

.completed {
    text-decoration: line-through;
    color: grey;
}

.checkbox {
    padding-right: 20px;
}

.extra-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 16px;
    border-top: 1px solid lightgrey;
    padding-top: 14px;
    margin-bottom: 14px;
}

button {
    font-size: 14px;
    background: transparent;
    appearance: none;
    border: 1px solid black;
    border-radius: 5px;

    &:hover {
        background: #ed1c7a;
    }

    &:focus {
        outline: none;
    }
}

    .active {
        background: #ed1c7a;
    }

    // Transitions
    .fade-enter-active, .fade-leave-active {
        transition: opacity .2s;
    }

    .fade-enter, .fade-leave-to {
        opacity: 0;
    }

</style>

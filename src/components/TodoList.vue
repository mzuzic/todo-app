<template>
  <div>
    <input class="todoInput" type="text" placeholder="Press enter after specifying the title" v-model="newTodo" 
    @keyup.enter="addToDo">
    <div v-for="todo in orderedTodos" :key="todo.id">

      <div>
        <div v-if="editing[todo.id]">
          <input type="text" v-model="changedTitle">
          <select v-model="changedPriority">
            <option value="low">low</option>
            <option value="medium">medium</option>
            <option value="high">high</option>
          </select>
          <button v-on:click="updateTodoAndSend(todo, changedTitle, changedPriority)">Change</button>
          <button v-on:click="setEditing(todo)"> Cancel</button>
        </div>
        <div v-else>
          <div class="inline" v-bind:class="{ 'done' : todo.done }">
            {{ todo.title }}
            <span class="priority">{{ todo.priority }}</span>
          </div>
          
          <div class="todo-item inline" title="modify" v-on:click="updateTodo(todo)">
            &spades;
          </div>
          <div class="todo-item inline" title="mark as done" v-on:click="markAsDone(todo)">
            &diams;
          </div>
          <div class="todo-item inline" title="delete" v-on:click="deleteTodo(todo)">
            &times;
          </div>
        </div>
      </div>
      <br>
    </div>
    
  </div>
</template>

<script>
import axios from 'axios';
import * as _ from 'lodash';

export default {
  name: 'TodoList',
  data () {
    return {
      newTodo: null,
      prioritySettable: 0,
      todos : [],
      editing: {},
      changedTitle : "",
      changedPriority : "low",
    }
  }, 
  methods: {
      addToDo() {
          var titleValue = this.newTodo;
          var self = this;
          axios.post('http://localhost:9005/api/createTodo', {
            title: titleValue,
          })
          .then(function (response) {
            self.todos.push(response.data);
            console.log(response);
          })
          .catch(function (error) {
            console.log(error);
          });
          this.newTodo = "";
      },
      deleteTodo(todo) {
          var self = this;
          axios.delete('http://localhost:9005/api/deleteTodo/' + todo.id)
              .then(function (response) {
                self.$delete(self.todos, self.todos.indexOf(todo));
                console.info(response.data);
              })
              .catch(function(error) {
                console.error(error);
              })
      },
      updateTodo(todo) {
        this.changedTitle = todo.title;
        this.$set(this.editing, todo.id, true);
      },
      updateTodoAndSend(todo, changedTitle, changedPriority) {
        todo.title = changedTitle;
        todo.priority = changedPriority;
        axios.put('http://localhost:9005/api/updateTodo', {
          id : todo.id,
          title : changedTitle,
          priority : changedPriority,
        })
          .then(function (response) {
            console.log(response.data);
          })
          .catch(function (error) {
            console.error(error);
          })
          .then(function () {
        // always executed
          })

        this.$set(this.editing, todo.id, false);
      },
      setEditing(todo) {
        this.$set(this.editing, todo.id, false);
      },
      markAsDone(todo) {
        todo.done = true;

        axios.put('http://localhost:9005/api/markAsDone/' + todo.id)
          .then(function (response) {
            console.log(response.data);
          })
          .catch(function (error) {
            console.error(error);
          })
          .then(function () {
        // always executed
          })
      },
  },
  computed: {
    orderedTodos: function() {
      return _.orderBy(this.todos, ['done', function (todo) { 
        if(todo.priority === "high") {
          return 1;
        } else if(todo.priority === "medium") {
          return 2;
        } else {
          return 3;
        }
      }]);
    }
  },
  created: function() {
      var self = this;
      axios.get('http://localhost:9005/api/getTodos')
          .then(function (response) {
            self.todos = response.data;
          })
          .catch(function (error) {
            console.error(error);
          })
          .then(function () {
        // always executed
          })
    }
  }

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.done {
  color: #42b983;
}
.todoInput {
    width: 35%;
}
.todo-item {
  cursor: pointer;
  margin-left: 14px;
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
div.inline { 
  float:left;  
}
.priority {
  font-size: 9pt;
  color: gray;
}
</style>

<template>
  <div class="hello">
    <b-container class="bv-example-row">
      <b-row>
        <b-col md="7" class="p-3 bg-info">
          <b-form-group
            id="fieldset-horizontal"
            label-cols-sm="4"
            label-cols-lg="3"
            label="Switch Your Project"
            label-for="input-horizontal"
          >
            <b-form-select v-model="selectedProject" :options="projects"></b-form-select>
          </b-form-group>
          </b-col>
      </b-row>
      <b-row class="fillter">
        <b-col>
          <div class = "todolist-text">
            <h2>{{this.selectedProject}} : To-Do-List</h2>
          </div>
        </b-col>
        <b-col>
          <div class="addButton">
            <b-button variant = "primary" v-b-modal.modal-prevent-closing>Add Task</b-button>
          </div>
        </b-col>
      </b-row>
      <b-row class="fillter">
        <b-form-select v-model="selectedCompleted" :options="filterOptions"></b-form-select>
      </b-row>
      <b-row>
        <b-table striped hover :items="fiterledItems" :fields="fields">
          <template #cell(edit)="data">
            <b-button variant="secondary" v-b-modal.editModal v-on:click="clickEdit(data.item.id)">Edit</b-button>
          </template>
          <template #cell(complete)="data">
            <div v-if="data.item.complete">
              <b-button variant="success" v-on:click="clickIncomplete(data.item.id)">Complete</b-button>
            </div>
            <div v-else>
              <b-button variant="info" v-on:click="clickComplete(data.item.id)">Incomplete</b-button>
            </div>
          </template>
          <template #cell(delete)="data">
            <b-button variant="danger" v-on:click="clickDelete(data.item.id)">Delete</b-button>
          </template>
        </b-table>
      </b-row>
      <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Add new task to " 
        @ok="handleOk"
        @show="resetModal"
        @hidden="resetModal"
      >
        <form ref="form" @submit.stop.prevent="addTodo">
          <b-form-group
            label="Name"
            label-for="name-input"
            invalid-feedback="Name is required"
          >
            <b-form-input
              id="name-input"
              v-model="newTodoName"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            label="Status"
            label-for="status-input"
            invalid-feedback="Status is required"
          >
            <b-form-select
              id="status-input"
              v-model="newTodoStatus"
              :options="options"
              required
            ></b-form-select>
          </b-form-group>
          <b-form-group
            label="Date"
            label-for="date-input"
            invalid-feedback="Date is required"
          >
            <b-form-datepicker 
                id="example-datepicker" 
                v-model="newTodoDate" 
                class="mb-2"
              ></b-form-datepicker>
          </b-form-group>
        </form>
      </b-modal>
      <b-modal
        :active.sync="isEditModalActive"
        id="editModal"
        ref="modal"
        title="Edit task"
        @ok="handleOkEdit"
        @hidden="resetModal"
      >
        <form ref="form" @submit.stop.prevent="editTask">
          <b-form-group
            label="Name"
            label-for="name-input"
            invalid-feedback="Name is required"
          >
            <b-form-input
              id="name-input"
              v-model="newTodoName"
              required
            ></b-form-input>
            
          </b-form-group>
          <b-form-group
            label="Status"
            label-for="status-input"
            invalid-feedback="Status is required"
          >
            <b-form-select
              id="status-input"
              v-model="newTodoStatus"
              :options="options"
              required
            ></b-form-select>
          </b-form-group>
          
          <b-form-group
            label="Date"
            label-for="date-input"
            invalid-feedback="Date is required"
          >
            <b-form-datepicker 
                id="example-datepicker" 
                v-model="newTodoDate" 
                class="mb-2"
              ></b-form-datepicker>
          </b-form-group>
        </form>
      </b-modal>
    </b-container>
  </div>
</template>

<script>
  export default {
    name: 'ToDoList',
    props: {
      msg: String
    },
    methods: {
      // Action when user click ok in edit modal
      handleOkEdit(bvModalEvt) {
        bvModalEvt.preventDefault()
        this.editTask()
      },
      // Action when user click ok in add modal
      handleOk(bvModalEvt) {
        bvModalEvt.preventDefault()
        this.addTodo()
      },
      // Action when user click complete button
      // It will set the task to complte
      clickComplete(id) {
        let obj = this.todos.find(todo => todo.id === id);
        var index = this.todos.indexOf(obj)
        this.todos[index].complete = true
      },
      // Action when user click incomplete button
      // It will set the task to incomplete
      clickIncomplete(id) {
        let obj = this.todos.find(todo => todo.id === id);
        var index = this.todos.indexOf(obj)
        this.todos[index].complete = false
      },
      // Action when user click delete button
      // It will delete the task
      clickDelete(id) {
        let obj = this.todos.find(todo => todo.id === id);
        var index = this.todos.indexOf(obj)
        if (index > -1) {
          this.todos.splice(index, 1);
        }
      },
      // Method to collect a new todo task.
      addTodo() {
        this.generatedId = this.generatedId+1
        this.todos.push({ 
          id: this.generatedId, 
          name: this.newTodoName, 
          status: this.newTodoStatus, 
          date: this.newTodoDate, 
          complete: false,
          project: this.selectedProject
        })
        
        this.resetModal()

        this.$nextTick(() => {
          this.$bvModal.hide('modal-prevent-closing')
        })
      },
      
      editTask() {
        let obj = this.todos.find(todo => todo.id === this.currentId);
        var index = this.todos.indexOf(obj)

        this.todos[index].name = this.newTodoName
        this.todos[index].status = this.newTodoStatus
        this.todos[index].date = this.newTodoDate

        this.resetModal()

        this.$nextTick(() => {
          this.$bvModal.hide('editModal')
        })
      },
      // Method to edit a todo task
      clickEdit(id) {
        this.currentId = id
        let obj = this.todos.find(todo => todo.id === this.currentId);
        var index = this.todos.indexOf(obj)

        this.newTodoName = this.todos[index].name
        this.newTodoStatus = this.todos[index].status
        this.newTodoDate = this.todos[index].date
      },
      resetModal() {
        this.newTodoName = ''
        this.newTodoStatus = null
        this.newTodoDate = ''
        this.currentId = -1
      }
    },
    data() {
      return {
        generatedId: 0,
        currentId: -1,
        isEditModalActive: false,
        selected: null,
        newTodoName: "",
        newTodoDate: "",
        newTodoStatus: null,
        selectedCompleted: 'all',
        selectedProject: 'ProjectA',
        projects:['ProjectA', 'ProjectB'],
        fields: [
          {
            key: 'id',
            sortable: true
          },
          {
            key: 'name',
            sortable: true
          },
          {
            key: 'status',
            sortable: true
          },
          {
            key: 'date',
            sortable: true
          },
          {
            key: 'edit',
          },
          {
            key: 'complete',
          },
          {
            key: 'delete',
          }
        ],
        todos: [],
        options: [{ text: 'Select One', value: null }, 'Ready', 'Pending', 'Complete', 'Paused'],
        filterOptions: [ 'all', 'complete', 'incomplete']
      }
    },
    computed: {
      fiterledItems() {
        var items = []
        if(this.selectedProject === 'ProjectA') {
          items = this.todos.filter(item => {
            return item.project === 'ProjectA'
          })
        }else{
          items = this.todos.filter(item => {
            return item.project === 'ProjectB'
          })
        }
        if(this.selectedCompleted === 'all') {
          return items
        } else if (this.selectedCompleted === 'complete'){
          return items.filter(item => {
            return item.complete
          })
        }
        return items.filter(item => {
          return !item.complete
        })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.addButton  {
  float: right;
  right: 0;
}

.todolist-text  {
  float: left;
}

.fillter {
  margin-top: 20px;
}
</style>

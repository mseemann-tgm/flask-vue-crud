<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>TODO</h1>
        <hr><br><br>
        <alert :message=message v-if="showMessage"></alert>
        <button type="button" class="btn btn-success btn-sm" v-b-modal.todo-modal>Add todo</button>
        <br><br>

        <!-- todos table -->
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">todo</th>
              <th scope="col">assignee</th>
              <th scope="col">done</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(todo, index) in todos" :key="index">
              <td>{{ todo.todo }}</td>
              <td>{{ todo.assignee }}</td>
              <td>
                <span v-if="todo.done">Yes</span>
                <span v-else>No</span>
              </td>

              <td>
                <button type="button"
                        class="btn btn-warning btn-sm"
                        v-b-modal.todo-update-modal
                        @click="edittodo(todo)">
                    Update
                </button>
                <button type="button"
                        class="btn btn-danger btn-sm"
                        @click="onDeletetodo(todo)">
                    Delete
                </button>
                <router-link :to="`/order/${todo.id}`"
                             class="btn btn-primary btn-sm">
                    Purchase
                </router-link>
              </td>
            </tr>
          </tbody>
        </table>

      </div>
    </div>

    <!-- add todo modal -->
    <b-modal ref="addtodoModal"
             id="todo-modal"
            todo="Add a new todo"
            hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-todo-group"
                      label="todo:"
                      label-for="form-todo-input">
            <b-form-input id="form-todo-input"
                          type="text"
                          v-model="addtodoForm.todo"
                          required
                          placeholder="Enter todo">
            </b-form-input>
        </b-form-group>
        <b-form-group id="form-assignee-group"
                      label="assignee:"
                      label-for="form-assignee-input">
          <b-form-input id="form-assignee-input"
                        type="text"
                        v-model="addtodoForm.assignee"
                        required
                        placeholder="Enter assignee">
          </b-form-input>

        <b-form-group id="form-done-group">
            <b-form-checkbox-group v-model="addtodoForm.done" id="form-checks">
              <b-form-checkbox value="true">done?</b-form-checkbox>
            </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Submit</b-button>
        <b-button type="reset" variant="danger">Reset</b-button>
      </b-form>
    </b-modal>

    <b-modal ref="edittodoModal"
             id="todo-update-modal"
             todo="Update"
             hide-footer>
      <b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
        <b-form-group id="form-todo-edit-group"
                      label="todo:"
                      label-for="form-todo-edit-input">
          <b-form-input id="form-todo-edit-input"
                        type="text"
                        v-model="editForm.todo"
                        required
                        placeholder="Enter todo">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-assignee-edit-group"
                      label="assignee:"
                      label-for="form-assignee-edit-input">
          <b-form-input id="form-assignee-edit-input"
                        type="text"
                        v-model="editForm.assignee"
                        required
                        placeholder="Enter assignee">
          </b-form-input>
        </b-form-group>

        <b-form-group id="form-done-edit-group">
          <b-form-checkbox-group v-model="editForm.done" id="form-checks">
            <b-form-checkbox value="true">done?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Update</b-button>
        <b-button type="reset" variant="danger">Cancel</b-button>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import axios from 'axios';
import Alert from './Alert';

export default {
  data() {
    return {
      todos: [],
      addtodoForm: {
        todo: '',
        assignee: '',
        done: [],

      },
      editForm: {
        id: '',
        todo: '',
        assignee: '',
        done: [],

      },
      message: '',
      showMessage: false,
    };
  },
  components: {
    alert: Alert,
  },
  methods: {
    gettodos() {
      const path = 'http://localhost:5000/todos';
      axios.get(path)
        .then((res) => {
          this.todos = res.data.todos;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
    addtodo(payload) {
      const path = 'http://localhost:5000/todos';
      axios.post(path, payload)
        .then(() => {
          this.gettodos();
          this.message = 'todo added!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.gettodos();
        });
    },
    updatetodo(payload, todoID) {
      const path = `http://localhost:5000/todos/${todoID}`;
      axios.put(path, payload)
        .then(() => {
          this.gettodos();
          this.message = 'todo updated!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.gettodos();
        });
    },
    removetodo(todoID) {
      const path = `http://localhost:5000/todos/${todoID}`;
      axios.delete(path)
        .then(() => {
          this.gettodos();
          this.message = 'todo removed!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.gettodos();
        });
    },
    initForm() {
      this.addtodoForm.todo = '';
      this.addtodoForm.assignee = '';
      this.addtodoForm.done = [];

      this.editForm.id = '';
      this.editForm.todo = '';
      this.editForm.assignee = '';
      this.editForm.done = [];
      this.editForm.id = '';
    },
    onSubmit(evt) {
      evt.preventDefault();
      this.$refs.addtodoModal.hide();
      let done = false;
      if (this.addtodoForm.done[0]) done = true;
      const payload = {
        todo: this.addtodoForm.todo,
        assignee: this.addtodoForm.assignee,
        done, // property shorthand

      };
      this.addtodo(payload);
      this.initForm();
    },
    onSubmitUpdate(evt) {
      evt.preventDefault();
      this.$refs.edittodoModal.hide();
      let done = false;
      if (this.editForm.done[0]) done = true;
      const payload = {
        todo: this.editForm.todo,
        assignee: this.editForm.assignee,
        done, // property shorthand

      };
      this.updatetodo(payload, this.editForm.id);
    },
    onReset(evt) {
      evt.preventDefault();
      this.$refs.addtodoModal.hide();
      this.initForm();
    },
    onResetUpdate(evt) {
      evt.preventDefault();
      this.$refs.edittodoModal.hide();
      this.initForm();
      this.gettodos(); // why?
    },
    onDeletetodo(todo) {
      this.removetodo(todo.id);
    },
    edittodo(todo) {
      this.editForm = todo;
    },
  },
  created() {
    this.gettodos();
  },
};
</script>

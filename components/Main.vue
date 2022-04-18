<template>
  <div class="main-page">
    <div class="main-page__wrapper wrapper">
      <div class="wrapper__logo"></div>
      <div class="wrapper__title">
        Today I need to
      </div>
      <div
        class="wrapper__body body"
        :class="{
          'body_error': isShowedError,
        }"
      >
        <div
          class="body__input input"
          :class="{
                    'input_expanded': todo.trim().length,
                  }"
        >
          <input
            v-model="todo"
            placeholder="Add new todo..."
            @focusin="focusInHandler"
            @keyup.enter="submit"
          >
          <div
            class="input__button"
            @click="validateBeforeSubmit"
          >
            Submit
          </div>
        </div>
        <div
          class="body__list"
          :class="{
            'body__list_under-editing': isEditMode,
          }"
        >
          <draggable
            :list="todos"
          >
            <TodoRow
              v-for="(todo, todoIndex) in filteredTodosList"
              :key="todo.name"
              :item="todo"
              @deleteTodo="deleteTodo(todo, todoIndex)"
              @editTodo="editTodo(todo, todoIndex)"
              @change="change"
            />
          </draggable>
        </div>
        <div class="body__counters">
          <TodosCounter
            title="Completed"
            :current-tasks-count="completedTodosList.length"
            :total-tasks-count="todos.length"
            color="#5D5FEF"
          />
          <TodosCounter
            title="To be finished"
            :current-tasks-count="activeTodosList.length"
            :total-tasks-count="todos.length"
            color="#EF5DA8"
          />
        </div>
      </div>
      <div class="wrapper__footer footer">
        <template v-if="!todos.length">
          <div class="footer__message">
            <div class="footer__message__icon"></div>
            Congrat, you have no more tasks to do
          </div>
        </template>
        <template v-else>
          <div
            v-if="this.activeTodosList.length && this.activeModeIndex !== 2"
            class="footer__button"
            @click="changeTodoActivity(true)"
          >
            Check all
          </div>
          <div class="footer__list-mode-switcher switcher">
            <div
              class="switcher__button"
              :class="{
                'switcher__button_active': activeModeIndex === 0,
              }"
              @click="switchMode(0)"
            >
              All
            </div>
            <div
              v-if="this.todos.length !== this.activeTodosList.length && this.todos.length !==
              this.completedTodosList.length"
              class="switcher__button"
              :class="{
                'switcher__button_active': activeModeIndex === 1,
              }"
              @click="switchMode(1)"
            >
              Active
            </div>
            <div
              v-if="this.todos.length !== this.activeTodosList.length && this.todos.length !==
              this.completedTodosList.length"
              class="switcher__button"
              :class="{
                'switcher__button_active': activeModeIndex === 2,
              }"
              @click="switchMode(2)"
            >
              Completed
            </div>
          </div>
          <div
            v-if="this.completedTodosList.length && this.activeModeIndex !== 1"
            class="footer__button"
            @click="changeTodoActivity(false)"
          >
            Clear completed
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
  import TodosCounter from './Main/TodosCounter';
  import TodoRow from './Main/TodoRow';
  import draggable from 'vuedraggable';

  export default {
    name: 'Main',
    components: {TodoRow, TodosCounter, draggable},
    props: {
      msg: String,
    },
    data() {
      return {
        activeModeIndex: 0,
        isEditMode: false,
        editingTodo: {},
        todo: '',
        todos: [],
        isShowedError: false,
      };
    },
    computed: {
      filteredTodosList() {
        return this.todos.filter((todo) => this.filterByCondition(todo));
      },
      activeTodosList() {
        return this.todos.filter((todo) => !todo.isCompleted);
      },
      completedTodosList() {
        return this.todos.filter((todo) => todo.isCompleted);
      },
    },
    created() {
      this.getTodos();
    },
    methods: {
      getTodos() {
        if (this.getDataFromCache()) {
          this.todos = this.getDataFromCache();
        }
      },
      focusInHandler() {
        if (this.isShowedError) {
          this.isShowedError = false;
        }
      },
      filterByCondition(todo) {
        let condition;
        switch (this.activeModeIndex) {
          case 1: {
            condition = !todo.isCompleted;
            break;
          }
          case 2: {
            condition = todo.isCompleted;
            break;
          }
          default: {
            condition = true;
            break;
          }
        }
        return condition;
      },
      validateBeforeSubmit() {
        const sameNameTodo = this.todos.find((todo) => todo.name === this.todo.trim());
        if (!sameNameTodo) {
          this.submit();
        } else {
          this.isShowedError = true;
        }
      },
      submit() {
        const newTodo = {
          name: this.todo,
          isCompleted: this.isEditMode ? this.editingTodo.isCompleted : false,
        };
        if (!this.isEditMode) {
          this.todos.push(newTodo);
        } else {
          const targetTodoIndex = this.todos.findIndex((todo) => todo.name === this.editingTodo.name);
          this.todos.splice(targetTodoIndex, 1, newTodo);
          this.isEditMode = false;
          this.editingTodo = {};
        }
        this.todo = '';
        this.setDataInCache(this.todos);
      },
      switchMode(index) {
        this.this.activeModeIndex = index;
      },
      deleteTodo(todo, todoIndex) {
        this.todos.splice(todoIndex, 1);
        this.setDataInCache(this.todos);
      },
      editTodo(todo) {
        this.isEditMode = true;
        this.editingTodo = JSON.parse(JSON.stringify(todo));
        this.todo = todo.name;
        this.setDataInCache(this.todos);
      },
      changeTodoActivity(isCheckActive) {
        if (isCheckActive) {
          this.activeTodosList.forEach((item) => {
            item.isCompleted = true;
          });
        } else {
          this.completedTodosList.forEach((item) => {
            const targetTodoIndex = this.todos.findIndex((todo) => todo.name = item.name);
            this.todos.splice(targetTodoIndex, 1);
          });
        }
        this.setDataInCache(this.todos);
      },
      setDataInCache(data, key = 'todos') {
        if (process.browser) {
          const storage = window.localStorage;
          storage.setItem(key, JSON.stringify(data));
        }
      },
      getDataFromCache(key = 'todos') {
        if (process.browser) {
          const storage = window.localStorage;
          return JSON.parse(storage.getItem(key));
        }
      },
      change() {
        this.setDataInCache(this.todos);
      },
    },
  };
</script>

<style
  scoped
  lang="scss"
>
  .main-page {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100vw;
    height: 100vh;
    background: linear-gradient(303.79deg, #F0C3FC -53.74%, #D7BEFC -4.64%, #BCB1FC 40.96%, #A1AEFC 83.05%, #94BDFC 127.47%, #8EC5FC 170.73%), linear-gradient(0deg, #84FAB0 0%, #85F9BC 21.87%, #88F8D7 42.19%, #8BF6F5 60.94%, #8EDDF5 80.73%, #8FD3F4 100%);

    .wrapper {
      display: flex;
      flex-direction: column;
      align-items: center;
      height: calc(100% - 180px);
      width: 730px;
      background-color: white;
      border-radius: 24px;
      padding: 32px 30px;

      &__logo {
        width: 180px;
        height: 180px;
        background-repeat: no-repeat;
        background-position: center;
        background-image: url("../static/Illustration.svg");
        margin-bottom: 50px;
      }

      &__title {
        font-weight: 700;
        font-size: 24px;
        line-height: 29px;
        margin-bottom: 50px;
      }

      .body {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: center;
        height: calc(100% - 372px);
        flex: 1;
        margin-bottom: 30px;

        .input {
          display: flex;
          transition: all .3s;
          width: 322px;
          overflow: hidden;

          input {
            display: flex;
            align-items: center;
            padding-left: 12px;
            height: 40px;
            border-radius: 8px;
            font-weight: 400;
            font-size: 14px;
            line-height: 16px;
            border: 1px solid #C7CCD1;
            outline: none;
            min-width: calc(317px - 12px);
            width: calc(317px - 12px);

            &::placeholder {
              color: #8F99A3;
            }
          }

          &__button {
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 0 16px;
            background-color: #2578F4;
            transition: all .3s;
            border-radius: 8px;
            cursor: pointer;
            margin-left: 16px;
            color: white;

            &:hover {
              box-shadow: 4px 4px 8px 0 rgba(34, 60, 80, 0.2);
            }
          }

          &_expanded {
            width: 422px;
          }
        }

        &_error {
          position: relative;

          &::before {
            content: 'Todo with such name is already exist';
            color: red;
            position: absolute;
            top: -18px;
            left: 25px;
          }

          .body__input input {
            border: 1px solid red;
          }
        }

        &__list {
          width: 450px;
          min-height: 78px;
          height: calc(100% - 44px - 85px - 40px);
          overflow-y: auto;
          margin: 10px 0;
          transition: all .3s;

          &_under-editing {
            opacity: .5;
            pointer-events: none;
          }
        }

        &__counters {
          display: grid;
          grid-template-columns: 1fr 1fr;
          grid-column-gap: 30px;
        }
      }

      .footer {
        display: flex;

        &__message {
          display: flex;
          align-items: center;
          color: #8F99A3;

          &__icon {
            margin-right: 12px;
            width: 24px;
            height: 24px;
            background-repeat: no-repeat;
            background-position: center;
            background-image: url("../static/mark.svg");
          }
        }

        &__button {
          font-weight: 600;
          font-size: 13px;
          line-height: 16px;
          cursor: pointer;
          padding: 8px 12px;
          background-color: white;
          border-radius: 8px;
          transition: all .3s;

          &:hover {
            box-shadow: 4px 4px 8px 0 rgba(34, 60, 80, 0.2);
          }
        }

        .switcher {
          display: flex;
          margin: 0 10px;
          min-width: 185px;

          &__button {
            font-weight: 600;
            font-size: 13px;
            line-height: 16px;
            cursor: pointer;
            padding: 8px 12px;
            background-color: white;
            border-radius: 8px;
            transition: all .3s;

            &_active {
              color: white;
              background-color: #2578F4;;
            }

            &:hover {
              //box-shadow: 4px 4px 8px 0 rgba(34, 60, 80, 0.2);
            }
          }
        }
      }

      @media (max-height: 900px) {
        &__logo {
          margin-bottom: 20px;
        }
        &__title {
          margin-bottom: 20px;
        }
      }
    }
  }
</style>

<template>
    <div
      class="row"
      :class="{
        'row_completed': item.isCompleted,
      }"
    >
      <input
        v-model="item.isCompleted"
        type="checkbox"
        class="row__checkbox"
        @click="changeTodoActivity"
      >
      <div class="row__title">
        {{ item.name }}
      </div>
      <div
        class="row__icon row__icon_edit"
        @click="editTodo"
      ></div>
      <div
        class="row__icon row__icon_delete"
        @click="deleteTodo"
      ></div>
    </div>
</template>

<script>

  export default {
    name: 'TodoRow',
    props: {
      title: {
        type: String,
        default: '',
      },
      item: {
        type: Object,
        default: () => ({}),
      },
    },
    methods: {
      editTodo() {
        this.$emit('editTodo');
      },
      deleteTodo() {
        this.$emit('deleteTodo');
      },
      changeTodoActivity() {
        this.$emit('change');
      },
    },
  };
</script>

<style
  scoped
  lang="scss"
>
  .row {
    height: 20px;
    width: calc(100% - 20px);
    display: grid;
    grid-template-columns: 14px 1fr 16px 16px;
    grid-column-gap: 10px;
    padding: 3px 10px;
    align-items: center;

    &__checkbox {
      cursor: pointer;
    }

    &__icon {
      width: 16px;
      height: 16px;
      background-repeat: no-repeat;
      background-position: center;
      cursor: pointer;

      &_edit {
        background-image: url("../../static/pencil.svg");
      }

      &_delete {
        background-image: url("../../static/thrash.svg");
      }
    }

    &:hover {
      background-color: #DCDEE2;
    }

    &_completed {
      .row__title {
        opacity: .5;
      }
    }
  }
</style>

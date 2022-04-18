<template>
  <div class="counter">
    <div class="counter__total">
      {{ `${currentTasksCount} tasks` }}
    </div>
    <div class="counter__title">
      {{ title }}
    </div>
    <div class="counter__progress-bar progress-bar">
      <div
        :style="{
          width: `${currentTasksCount / totalTasksCount * 100}%`,
          backgroundColor: color,
        }"
        class="progress-bar__actual"></div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'TodosCounter',
    props: {
      title: {
        type: String,
        default: '',
      },
      color: {
        type: String,
        default: '',
      },
      currentTasksCount: {
        type: Number,
        default: 0,
      },
      totalTasksCount: {
        type: Number,
        default: 0,
      },
    },
  };
</script>

<style
  scoped
  lang="scss"
>
  .counter {
    display: flex;
    flex-direction: column;
    width: 158px;
    height: 53px;
    padding: 16px;
    background-color: rgba(32, 36, 39, 0.05);
    border-radius: 8px;

    &__total {
      font-weight: 600;
      font-size: 12px;
      line-height: 16px;
      color: #202427;
      opacity: .5;
      margin-bottom: 4px;
    }

    &__title {
      font-weight: 700;
      font-size: 14px;
      line-height: 17px;
      color: #202427;
      margin-bottom: 12px;
    }

    .progress-bar {
      width: 158px;
      height: 4px;
      background: #20242707;
      border-radius: 8px;
      position: relative;
      &__actual {
        position: absolute;
        top: 0;
        left: 0;
        background-color: black;
        height: 4px;
        border-radius: 8px;
      }
    }
  }
</style>

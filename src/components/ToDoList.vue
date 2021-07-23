<template>
  <div class="toDoList">
    <div class="task-container" v-for="task in tasks" v-bind:key="task.text">
      <div class="current-task">
        <p>{{ task.text }}</p>
        <input type="checkbox" name="" id="" v-bind:checked="task.status" />
      </div>
      <div
        class="subtasks"
        v-for="subtask in task.subtasks"
        v-bind:key="subtask.text"
      >
        <div class="subtask">
          <p>{{ subtask.text }}</p>
          <input
            type="checkbox"
            name=""
            id=""
            v-bind:checked="subtask.status"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ToDoList",
  props: {
    tasks: {
      type: Object,
      required: true,
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.task-container {
  background-color: #787878;
}

.current-task {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: center;
  background-color: red;
}

.subtask {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: center;
  background-color: green;
}
</style>

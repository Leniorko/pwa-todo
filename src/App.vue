<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <to-do-list v-bind:tasks="this.tasks" />
</template>

<script>
import ToDoList from "./components/ToDoList.vue";

export default {
  name: "App",
  components: {
    ToDoList,
  },
  data() {
    return {
      db: false,
      tasks: {},
    };
  },
  beforeCreate() {
    let databaseRequest = window.indexedDB.open("pwaTodo", 3);

    const mockupToDo = [
      {
        text: "Wash the dishes",
        status: false,
        subtasks: [
          { text: "Wash plastes", status: false },
          { text: "Wash kaddle", status: false },
        ],
      },
      {
        text: "Create ToDo app",
        status: false,
        subtasks: [
          { text: "Connect local db", status: true },
          { text: "Create mockup data", status: true },
          { text: "Populate db with date", status: false },
          { text: "Show data from db", status: false },
          { text: "Add interaction", status: false },
          { text: "Add styles", status: false },
        ],
      },
    ];

    databaseRequest.onerror = (event) => {
      console.error("Database didn't opened. " + event.target.errorCode);
    };

    databaseRequest.onsuccess = (event) => {
      console.log("Database connected");
      this.db = event.target.result;

      // It will work because onupgradeneeded goes before onsuccess
      let testTrans = this.db.transaction(["tasks"]);
      let tasksObjectStore = testTrans.objectStore("tasks");

      let allData = tasksObjectStore.getAll();

      allData.onsuccess = () => {
        console.log(allData.result);
        this.tasks = allData.result;
      };
    };

    // COMPLETED: Check if old data are whiped with this method.
    //            No, it's not get whiped if you not specify so
    // TODO: Change objectStore delete to simple migration of all previous data smh if it is possible.
    databaseRequest.onupgradeneeded = (event) => {
      console.log("Need upgrade. Upgrading");
      this.db = event.target.result;

      let objectStore;

      try {
        objectStore = this.db.createObjectStore("tasks", {
          autoIncrement: true,
        });
      } catch (DOMException) {
        this.db.deleteObjectStore("tasks");
        objectStore = this.db.createObjectStore("tasks", {
          autoIncrement: true,
        });
      }

      mockupToDo.forEach((task) => {
        objectStore.add(task);
      });
    };
  },
};
</script>

<style>
*,
*::before,
*::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

<template>
  <div class="toDoList">
    <div
      class="task-container"
      v-for="(task, index) in tasks"
      v-bind:key="task.text"
    >
      <div class="current-task">
        <p>{{ task.text }}</p>
        <input type="checkbox" name="" id="" v-bind:checked="task.status" />
        <div class="new-task" v-if="task === tasks[tasks.length - 1]">
          <input
            type="text"
            name=""
            id=""
            v-model="taskAddFieldValue"
            v-on:keyup.enter="addTask()"
          />
          <p class="new-task__button" v-on:click="addTask()">Add new task</p>
        </div>
      </div>
      <div class="subtask-wap" v-if="task.subtasks.length !== 0">
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
            <div
              class="new-task"
              v-if="subtask === task.subtasks[task.subtasks.length - 1]"
            >
              <input
                type="text"
                name=""
                id=""
                v-model="subtaskAddFieldValue[index]"
                v-on:keyup.enter="addTask(task, index)"
              />
              <p class="new-task__button" v-on:click="addTask(task, index)">
                Add new sub task
              </p>
            </div>
          </div>
        </div>
      </div>
      <div class="subtask-wrapper" v-else>
        <div
          class="new-task"
          v-if="subtask === task.subtasks[task.subtasks.length - 1]"
        >
          <input
            type="text"
            name=""
            id=""
            v-model="subtaskAddFieldValue[index]"
            v-on:keyup.enter="addTask(task, index)"
          />
          <p class="new-task__button" v-on:click="addTask(task, index)">
            Add new sub task
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ToDoList",
  data: function () {
    return {
      db: false,
      tasks: false,
      taskAddFieldValue: "",
      subtaskAddFieldValue: {},
    };
  },
  methods: {
    addTask(task = false, subtaskFieldId = false) {
      if (task) {
        console.log("lmao");
        console.log(task);
        const addSubtaskCursor = this.db.transaction(["tasks"], "readwrite");
        const requestTaskFromDb = addSubtaskCursor
          .objectStore("tasks")
          .get(task.id);

        requestTaskFromDb.onsuccess = (event) => {
          const taskFromDb = event.target.result;
          console.log(this.subtaskAddFieldValue);
          console.log(subtaskFieldId);
          taskFromDb.subtasks.push({
            text: this.subtaskAddFieldValue[subtaskFieldId],
            status: false,
          });

          const requetUpdate = addSubtaskCursor
            .objectStore("tasks")
            .put(taskFromDb);

          requetUpdate.onerror = (event) => {
            console.log("Error during value update " + event.target.errorCode);
          };

          requetUpdate.onsuccess = () => {
            console.log("Updatet sucsessfully");
            this.updateTasksList();
            this.subtaskAddFieldValue[subtaskFieldId] = "";
          };
        };
        return;
      }

      const addTaskCursor = this.db.transaction(["tasks"], "readwrite");
      const taskValue = this.taskAddFieldValue;
      if (taskValue.split(" ").join("").length === 0) {
        alert("Enter not empty task value.");
        return;
      }

      addTaskCursor.objectStore("tasks").add({
        text: taskValue,
        status: false,
        subtasks: [],
      });

      addTaskCursor.oncomplete = () => {
        console.log("Added task succsessfully");
        this.updateTasksList();
        this.taskAddFieldValue = "";
      };

      addTaskCursor.onerror = (event) => {
        console.log(`Not done: ${event.target.errorCode}`);
      };
    },

    updateTasksList() {
      let updatingTasksCursor = this.db.transaction(["tasks"]);
      let updatedData = updatingTasksCursor.objectStore("tasks").getAll();
      updatingTasksCursor.oncomplete = () => {
        this.tasks = updatedData.result;
      };
    },
  },
  beforeCreate() {
    let databaseRequest = window.indexedDB.open("pwaTodo", 5);

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
          keyPath: "id",
        });
      } catch (DOMException) {
        this.db.deleteObjectStore("tasks");
        objectStore = this.db.createObjectStore("tasks", {
          autoIncrement: true,
          keyPath: "id",
        });
      }

      mockupToDo.forEach((task) => {
        objectStore.add(task);
      });
    };
  },
};
</script>

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
  color: white;
}

.new-task {
  cursor: pointer;
}
</style>

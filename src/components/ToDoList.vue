<template>
  <div class="todolist">
    <div class="todolist__item" v-for="task in tasks" v-bind:key="task.text">
      <div class="todolist__task">
        <p class="todolist__text">{{ task.text }}</p>
        <label class="todolist__custom-checkbox-container">
          <input
            type="checkbox"
            name=""
            id="task-status"
            class="todolist__checkbox"
            v-bind:checked="task.status"
            v-on:click="changeTaskState(task)"
          />
          <div class="todolist__custom-checkbox"></div>
        </label>
        <img
          src="../assets/icons/delete_forever_black_24dp.svg"
          alt=""
          srcset=""
          class="todolist__delete"
          v-on:click="deleteTask(task)"
        />
      </div>
    </div>
    <div class="todolist__new">
      <input
        type="text"
        name=""
        id="task-text"
        placeholder="Enter task text..."
        v-model="taskAddFieldValue"
        class="todolist__new-input"
        v-on:keyup.enter="addTask()"
      />
      <p class="todolist__new-button" v-on:click="addTask()">Add new task</p>
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
    addTask() {
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

    changeTaskState(task) {
      let tasksObjectStore = this.db
        .transaction(["tasks"], "readwrite")
        .objectStore("tasks");

      let ourTaskProcess = tasksObjectStore.get(task.id);
      ourTaskProcess.onsuccess = (event) => {
        let ourTask = event.target.result;

        ourTask.status = !ourTask.status;

        let ourTaskUpdateRequest = tasksObjectStore.put(ourTask);

        ourTaskUpdateRequest.onsuccess = () => {
          console.log("Status changed");
        };
      };
    },

    deleteTask(task) {
      let tasksObjectStore = this.db
        .transaction(["tasks"], "readwrite")
        .objectStore("tasks");
      let ourTaskProcess = tasksObjectStore.delete(task.id);
      ourTaskProcess.onsuccess = () => {
        console.log("Deleted");
        this.updateTasksList();
      };
    },
  },
  beforeCreate() {
    let databaseRequest = window.indexedDB.open("pwaTodo", 5);

    const mockupToDo = [
      {
        text: "Wash the dishes",
        status: false,
      },
      {
        text: "Create ToDo app",
        status: false,
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
.todolist {
  max-width: 400px;
  margin: auto;
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
  padding: 0 16px;
}

.todolist__item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding-bottom: 6px;
  border-bottom: 1px solid #787878;
}

.todolist__text {
  overflow-wrap: anywhere;
}

.todolist__task {
  display: flex;
  gap: 16px;
  width: 100%;
}

.todolist__custom-checkbox-container {
  margin-left: auto;
  display: flex;
  align-items: center;
}

.todolist__custom-checkbox {
  width: 24px;
  height: 24px;
  border: 1px solid black;
  border-radius: 6px;
  cursor: pointer;
}

.todolist__custom-checkbox:hover {
  background-color: rgb(0, 153, 255);
}

.todolist__checkbox {
  position: absolute;
  z-index: -1;
  opacity: 0;
}

.todolist__checkbox:checked ~ .todolist__custom-checkbox {
  background-color: rgb(0, 110, 255);
  background-image: url("../assets/icons/done_black_24dp.svg");
}

.todolist__delete {
  cursor: pointer;
}

.todolist__new {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.todolist__new-input {
  width: 100%;
  font-size: 18px;
  padding: 16px 16px 0 0;
  border: none;
  outline: none;
  border-bottom: 1px solid black;
}

.todolist__new-button {
  background: rgb(0, 191, 255);
  width: fit-content;
  padding: 8px;
  color: white;
  cursor: pointer;
  border-radius: 16px;
}

.todolist__new-button:hover {
  background: rgb(0, 140, 255);
}
</style>

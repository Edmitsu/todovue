<template>
  <div class="container">
    <header class="header">
      <h1 class="header__title" >{{ title }}</h1>
      <input type="text" v-model="title" @input="changeTitle" />
    </header>
    <main class="main">
      <TaskForm>
        
      </TaskForm>
      <ul class="task-list">
        <li v-for="task in tasks" :key="task.id" class="task-list__item">
          <label class="container__checkbox">
            <input class="task-list__checkbox" type="checkbox" v-model="task.completed" />
            <span class="checkmark"></span>
          </label>

          <span :class="{'task-list__task--completed': task.completed}">{{ task.name }}</span>
          <span class="task-list__date" v-if="task.dueDate"> - {{ this.getTimeRemaining(task.dueDate) }} restante</span>
          <button class="task-list__edit-task-btn" @click="showModal = true; selectedTask = task">Editar</button>
          <button class="task-list__edit-task-btn" @click="deleteTask(task.id)">Deletar</button>
        </li>
      </ul>
    </main>
    <footer class="footer">

    </footer>
    <dialog class="modal_container" open v-if="showModal" @close="showModal = false">
      <div class="modal">
      <h2 class="modal__title">{{ selectedTask ? 'Editar Tarefa' : 'Nova Tarefa' }}</h2>
      <form class="modal__form">
        <label class="modal__label">Nome da Tarefa</label>
        <input placeholder="Insira o nome da tarefa" class="modal__input" v-model="form.name" />
        <label class="modal__label">Data de Vencimento</label>
        <input class="modal__input" type="date" v-model="form.dueDate" />
      </form>
      <div class="modal__actions">
        <button class="modal__save-btn" @click="saveTask">{{ selectedTask ? 'Salvar' : 'Adicionar' }}</button>
        <button class="modal__cancel-btn" @click="cancelTask">Cancelar</button>
      </div>
      </div>
    </dialog> 
  </div>
</template>


<script>
import TaskForm from "./components/TaskForm";

export default {
  name: "App",
  data() {
    return {
      tasks: [],
      title: "",
      showModal: false,
      selectedTask: null,
      form: {
        name: "",
        dueDate: null,
      },
    };
  },
  components: {
    TaskForm,
  },
  created() {
    this.getTasks();
    this.getTitle();
  },
  mounted(){
    this.getTimeRemaining();
  },
  methods: {
    async getTasks() {
      const response = await fetch("http://localhost:3000/tarefas");
      const data = await response.json();

      const today = new Date();

      data.tasks.forEach((task) => {
        const dueDate = new Date(task.dueDate);
        const diffInTime = dueDate.getTime() - today.getTime();
        const diffInDays = Math.floor(diffInTime / (1000 * 3600 * 24));
        const diffInHours = Math.floor((diffInTime % (1000 * 3600 * 24)) / (1000 * 3600));
        const diffInMinutes = Math.floor((diffInTime % (1000 * 3600)) / (1000 * 60));
        const diffInSeconds = Math.floor((diffInTime % (1000 * 60)) / 1000);

        task.timeRemaining = `${diffInDays} days, ${diffInHours} hours, ${diffInMinutes} minutes, ${diffInSeconds} seconds`;
      });

      this.tasks = data.tasks;
    },

    async getTitle() {
      const req = await fetch("http://localhost:3000/titulos");
      const data = await req.json();
      this.title = data.title;
    },
    async saveTask() {
      const response = await fetch("http://localhost:3000/tarefas");
      const existingData = await response.json();

      if (this.selectedTask) {
        const existingTask = existingData.tasks.find(
          (task) => task.id === this.selectedTask.id
        );
        if (existingTask) {
          existingTask.name = this.form.name;
          existingTask.dueDate = this.form.dueDate || existingTask.dueDate;
        }
      } else {
        const newId = existingData.tasks.length + 1;
        const newTask = {
          id: newId,
          name: this.form.name,
          completed: false,
          dueDate: this.form.dueDate,
        };
        existingData.tasks.push(newTask);
      }

      const dataJson = JSON.stringify(existingData);
      const req = await fetch("http://localhost:3000/tarefas", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: dataJson,
      });

      const res = await req.json();
      console.log(res);

      this.form.name = "";
      this.form.dueDate = null;
      this.showModal = false;
      this.selectedTask = null;
      this.getTasks();
    },
    cancelTask() {
      this.showModal = false;
      this.selectedTask = null;
      this.form.name = "";
      this.form.dueDate = null;
    },
    async deleteTask(id) {
      const response = await fetch("http://localhost:3000/tarefas");
      const existingData = await response.json();

      const index = existingData.tasks.findIndex((task) => task.id === id);
      if (index !== -1) {
        existingData.tasks.splice(index, 1);

        const dataJson = JSON.stringify(existingData);
        const req = await fetch("http://localhost:3000/tarefas", {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: dataJson,
        });

        const res = await req.json();
        console.log(res);

        this.getTasks();
      }
    },
      openModal(task = null) {
        this.showModal = true;
        this.form.name = task ? task.name : "";
        this.form.dueDate = task ? task.dueDate: null;
        this.selectedTask = task;
        },
        getTimeRemaining(dueDate) {
         const dueDateTime = new Date(dueDate).getTime();
         const now = new Date().getTime();
         const timeRemaining = dueDateTime - now;
 
         const seconds = Math.floor((timeRemaining / 1000) % 60);
         const minutes = Math.floor((timeRemaining / 1000 / 60) % 60);
         const hours = Math.floor((timeRemaining / 1000 / 60 / 60) % 24);
         const days = Math.floor(timeRemaining / (1000 * 60 * 60 * 24));
 
         return `${days}d ${hours}h ${minutes}m ${seconds}s`;
       }
      },
    };
</script>


<style>
  *{
    box-sizing: border-box;
    padding: 0;
    margin: 0;
  }
  body { 
    min-height: 100vh; 
    background: linear-gradient(#355C7D, white);
    display:flex; flex-direction:column;
  }
  .container{
    display: flex;
    flex-direction: column;
    background-size: cover;
  }
  .header{
    display: flex;
    flex-flow: column wrap;
    align-items: flex-start;
    font-weight: bold;
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    padding: 1.5em 0.5em 1.5em 1.5em;
    background-color: #355C7D;
    width: 100%;
    color: white;
  }
  .footer{
    background-color: #355C7D;
    width: 100%;
    min-height: 10%;
    padding: 2.5em;
    color: white;
  }
  .header__title{
    font-size: 1.5em;
    background-color: transparent;
  }
  .add-task-btn{
    font-size: 1.5em;
    border: 0px;
    margin: 0.5em 0 0 0;
    padding: 0.5em;
    cursor: pointer;  
    background-color: #355C7D;
    width: 80%;
    color: white;
    border-radius: 1.5em;
  }
  .main{
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-size: cover;
    color: white;  
    min-height: 100vh; 
  }
  .task-list{
    list-style-type: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    padding: 1.4em;
  }

  .container__checkbox {
  display: block;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 22px;
  }

  .task-list__checkbox {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    height: 0;
    width: 0;
  }

  .checkmark {
    display: flex;   
    top: 0;
    left: 0;
    height: 1em;
    width: 1em;
    background-color: #eee;
    border-radius: 2px;
    margin: 0em 0.5em 0em 0em;
  }

  .container__checkbox:hover input ~ .checkmark {
    background-color: #ccc;
  }

  .container input:checked ~ .checkmark {
    background-color: #68a3d6;
  }

  .task-list__item{
    display: flex;
    align-items: center;
    align-self: flex-start;
    padding: 1.2em;
    font-size: 1.4em;
    color: white;
  }

  .task-list__task--completed {
    color: #333;
    font-size: 1.2em;
    text-decoration: line-through;
  }

  .task-list__date{
    display: flex;
    flex-direction: row;
    margin-left: 10px;
    font-size: 0.7em;
    background: transparent; 
    color: rgb(255, 255, 255);
    border: none;
    border-radius: 1.5em;  
  }

  .task-list__edit-task-btn{
    display: flex;
    flex-direction: row;
    margin-left: 10px;
    background: transparent; 
    color: rgb(255, 255, 255);
    border: none;
    border-radius: 1.5em;    
  }

  .modal_container{
    display: flex;
    flex-direction: column;
    justify-content: center;
    background-color: rgb(0 0 0 / .3);
    padding: 1em;
    min-width: 100vw;
    min-height: 100vh;
  }

  .modal__title{
    display: flex;
    flex-direction: column;
    align-items: center;
    font-weight: bold;
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    padding: 25px 0 25px 0;
    background-color: #355C7D;
    width: 100%;
    color: rgb(255, 255, 255);
    font-size: 2em;
  }

  .modal__form{
    display: flex;
    flex-direction: column;
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    padding: 1.5em;
    background-color:#355C7D;
    color: rgb(255, 255, 255);
  }
  .modal__label{
    background-color: transparent;
  }
  .modal__input{
    padding: 0.5em;
    margin: 1em;
    border-radius: 1em;
    background-color: white;
    color: #355C7D;
    border: none;
    
  }
  .modal__actions{
    display: flex;
    padding: 1em;
    justify-content: flex-end;
    background-color:white;   
    color: white;  
  }
  .modal__save-btn{
    background-color: #355C7D;
    border-radius: 1em;
    color: rgb(255, 255, 255); 
    padding:0.5em;
    margin: 0.5em;
  }
  .modal__cancel-btn{
    background-color: #355C7D;
    border-radius: 1em;
    color: rgb(255, 255, 255); 
    padding:0.5em;
    margin: 0.5em;
  }
</style>

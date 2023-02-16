<template>
  <div class="container">
    <header class="header">
      <h1 class="header__title" >{{ title }}</h1>
    </header>
    <main class="main">
      <button class="add-task-btn" @click="showModal = true">Adicionar Tarefa</button>
      <ul class="task-list">
        <li v-for="task in tasksdata" :key="task.id" value="task.name" class="task-list__item">
          <input type="checkbox" v-model="task.completed" />
          <span class="task-list__task--completed">{{ task.name }}</span>
          <button class="task-list__edit-task-btn" @click="showModal = true; selectedTask = task">Editar</button>
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
        <input class="modal__input" v-model="form.name" />
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
export default {
  data() {
    return {
      tasks: [],
      tasksdata: null,
      name: null,
      completed: false,
      title: null,
      showModal: false,
      selectedTask: null,
      form: {
        name: '',
        completed: false
      }, 
    };
  },
  methods: {
    async getTasks() {
      const req = await fetch('http://localhost:3000/tarefas')
      const data = await req.json()
      this.tasksdata = data.tasks
    },
    async getTitle() {
      const req = await fetch('http://localhost:3000/titulos')
      const data = await req.json()
      this.title = data.title.name
    },
    async saveTask() {  
      const response = await fetch("http://localhost:3000/tarefas");
      const existingData = await response.json();

      if (this.selectedTask) {
        const existingTask = existingData.tasks.find(task => task.id === this.selectedTask.id);
        if (existingTask) {
          existingTask.name = this.form.name;
          existingTask.completed = this.form.completed = this.selectedTask.completed;
        }
      } else {
        const newId = existingData.tasks.length + 1;
        const newTask = {
          id: newId,
          name: this.form.name,
          completed: this.form.completed
        };
        existingData.tasks.push(newTask);
      }

      const dataJson = JSON.stringify(existingData);
      const req = await fetch("http://localhost:3000/tarefas", {
        method: "POST",
        headers: { "Content-Type" : "application/json" },
        body: dataJson
      });

      const res = await req.json();
      console.log(res);

      this.form.name = '';
      this.form.completed = false;
      this.showModal = false;
      this.selectedTask = null;
      this.getTasks();
    },
    cancelTask() {
      this.showModal = false;
      this.selectedTask = null;
    }
  },
  mounted () {
    this.getTasks(),
    this.getTitle()
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
    background: linear-gradient(#F8B195, #C06C84);
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
    background-color: #C06C84;
    box-shadow: 0px 0px 2px 2px #F67280;
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

  .task-list__item{
    display: flex;
    align-items: center;
    align-self: flex-start;
    padding: 1.2em;
    font-size: 1.2em;
    color: white;
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
    max-width: 50vh;
    min-height: 100vh;
  }

  .modal{
    border-radius: 5rem;;
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
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    padding: 1.5em;
    background: linear-gradient(#F8B195, #C06C84);
    color: rgb(255, 255, 255);
  }
  .modal__label{
    background-color: transparent;
  }
  .modal__input{
    padding: 0.5em;
    margin: 1em;
    border-radius: 1em;
    background-color: #C06C84;
    color: rgb(255, 255, 255);
    border: none;
    
  }
  .modal__actions{
    display: flex;
    padding: 1em;
    justify-content: flex-end;
    background: linear-gradient(#C06C84, #6C5B7B);   
    color: white;  
  }
  .modal__save-btn{
    background-color: #C06C84;
    border-radius: 1em;
    color: rgb(255, 255, 255); 
    padding:0.5em;
    margin: 0.5em;
  }
  .modal__cancel-btn{
    background-color: #C06C84;
    border-radius: 1em;
    color: rgb(255, 255, 255); 
    padding:0.5em;
    margin: 0.5em;
  }
</style>

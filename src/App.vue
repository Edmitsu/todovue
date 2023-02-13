<template>
  <div class="container">
    <header class="header">
      <h1 class="header__title">Minhas Tarefas</h1>
      <button class="header__add-task-btn" @click="showModal = true">Adicionar Tarefa</button>
    </header>
    <main class="main">
      <ul class="task-list">
        <li v-for="task in tasks" :key="task.id" class="task-list__item">
          <input type="checkbox" v-model="task.completed" />
          <span :class="{ 'task-list__task--completed': task.completed }">{{ task.name }}</span>
          <button class="task-list__edit-task-btn" @click="showModal = true; selectedTask = task">Editar</button>
        </li>
      </ul>
    </main>
    <modal v-if="showModal" @close="showModal = false">
      <h2 class="modal__title">{{ selectedTask ? 'Editar Tarefa' : 'Nova Tarefa' }}</h2>
      <form class="modal__form">
        <label class="modal__label">Nome da Tarefa</label>
        <input class="modal__input" v-model="form.name" />
      </form>
      <div class="modal__actions">
        <button class="modal__cancel-btn" @click="showModal = false">Cancelar</button>
        <button class="modal__save-btn" @click="saveTask">{{ selectedTask ? 'Salvar' : 'Adicionar' }}</button>
      </div>
    </modal>
  </div>
</template>


<script>
export default {
  data() {
    return {
      tasks: [
        { id: 1, name: 'Fazer compras', completed: false },
        { id: 2, name: 'Lavar roupa', completed: false },
        { id: 3, name: 'Limpar a casa', completed: true }
      ],
      showModal: false,
      selectedTask: null,
      form: {
        name: ''
      }
    };
  },
  methods: {
    addTask() {
      this.modalTitle = 'Add Task';
      this.currentTask = {};
      this.showModal = true;
    },
    editTask(id) {
      this.modalTitle = 'Edit Task';
      this.currentTask = this.tasks.find(task => task.id === id);
      this.showModal = true;
    },
    deleteTask(id) {
      this.tasks = this.tasks.filter(task => task.id !== id);
    },
    saveTask() {
      if (this.selectedTask) {
        this.selectedTask.name = this.form.name;
        this.selectedTask = null;
      } else {
        this.tasks.push({
          id: Math.max(...this.tasks.map(task => task.id)) + 1,
          name: this.form.name,
          completed: false
        });
      }
      this.form.name = '';
      this.showModal = false;
    }
  },
};
</script>







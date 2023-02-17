<template>
  <button class="add-task-btn" type="button" @click="showModal = true">Adicionar Tarefas iniciais</button>
  <dialog open class="modal_container" v-if="showModal" @close="showModal = false" >
    <div class="task-form-container">
      <form @submit.prevent="handleSubmit" class="modal__form">
        <div class="task-form-row" v-for="(task, index) in tasks" :key="index" >
          <label class="modal__label modal__label_name" :for="'name' + index">Nome da tarefa:</label>
          <input class="modal__input" type="text" :id="'name' + index" v-model="task.name" />
          <label class="modal__label" :for="'completed' + index">Completado:</label>
          <input type="checkbox" :id="'completed' + index" v-model="task.completed" />
          <label class="modal__label" :for="'dueDate' + index">Data de vencimento:</label>
          <input type="date" :id="'dueDate' + index" v-model="task.dueDate" />
        </div>
        <button type="button" class="task-form-button" @click="addTask">Adicionar Tarefa</button>
        <button type="submit" class="task-form-button">{{ tasks.length > 1 ? 'Adicionar Tarefas' : 'Salvar Tarefas' }}</button>
      </form>
    </div>
  </dialog>
</template>

<script>
export default {
  name: 'TaskForm',
  data() {
    return {
      tasks: [{ name: '', completed: false, dueDate: '' }],
      showModal: false,
    };
  },
  methods: {
    async handleSubmit() {
      const response = await fetch('http://localhost:3000/tarefas');
      const existingData = await response.json();

      // Generate new task IDs based on existing tasks
      const newIds = Array.from({ length: this.tasks.length }, (_, i) => existingData.tasks.length + 1 + i);

      // Create new task objects with IDs
      const newTasks = this.tasks.map((task, i) => ({
        id: newIds[i],
        name: task.name,
        completed: task.completed,
        dueDate: task.dueDate
      }));

      // Add new tasks to existing data
      if (existingData.tasks.length > 0) {
        existingData.tasks.push(...newTasks);
      } else {
        existingData.tasks = newTasks;
      }

      // Update JSON file with new data
      const dataJson = JSON.stringify(existingData);
      const req = await fetch('http://localhost:3000/tarefas', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: dataJson,
      });

      const res = await req.json();
      console.log(res);

      // Reset form data and close modal
      this.tasks = [{ name: '', completed: false, dueDate: '' }];
      this.showModal = false;

      // Emit event to update task list
      this.$emit('task-added');

      // Reload the page to show updated content
      window.location.reload();
    },

    addTask() {
      this.tasks.push({ name: '', completed: false, dueDate: '' });
    },
  },
};
</script>


<style scoped>
.task-form-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 2rem;
  font-family:Verdana, Geneva, Tahoma, sans-serif;
}

.task-form-row {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 1rem;
  padding: 1em;
  flex-wrap: wrap;
}

label {
  font-size: 1em;
  font-weight: bold;
  padding: 1em;
}

input{
  padding: 1em;
  border-radius: 1.5em;
  font-size: 0.9em;
  font-weight: bold;
}

.task-form-button {
  padding: 1em;
  border-radius: 1.5em;
  font-size: 0.9em;
  font-weight: bold;
}

.modal_container{
    display: flex;
    flex-direction: column;
    justify-content: center;
    background-color: rgb(0 0 0 / .3);
    padding: 1em;
    max-width: 50vh;
    min-height: 100vh;
    font-family:Verdana, Geneva, Tahoma, sans-serif;
    
}
.modal__label_name{
  font-size: 1.3em;
}
</style>
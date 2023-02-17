<template>
  <form @submit.prevent="handleSubmit">
    <div v-for="(task, index) in tasks" :key="index">
      <label :for="'name' + index">Nome da tarefa:</label>
      <input type="text" :id="'name' + index" v-model="task.name" />
      <label :for="'completed' + index">Completado:</label>
      <input type="checkbox" :id="'completed' + index" v-model="task.completed" />
    </div>
    <button type="button" @click="addTask">Adicionar Tarefa</button>
    <button type="submit">{{ tasks.length > 1 ? 'Adicionar Tarefas' : ' Salvar Tarefas' }}</button>
  </form>
</template>

<script>
export default {
  name: 'TaskForm',
  data() {
    return {
      tasks: [{ name: '', completed: false }],
    };
  },
  methods: {
    async handleSubmit() {
      const response = await fetch('http://localhost:3000/tarefas');
      const existingData = await response.json();

      // Generate new task IDs based on existing tasks
      const newIds = Array.from({ length: this.tasks.length }, (_, i) => existingData.tasks.length + 1 + i);

      // Create new task objects with IDs
      const newTasks = this.tasks.map((task, i) => ({ id: newIds[i], name: task.name, completed: task.completed }));

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
      this.tasks = [{ name: '', completed: false }];
      this.showModal = false;

      // Emit event to update task list
      this.$emit('task-added');

      // Reload the page to show updated content
      window.location.reload();
    },

    addTask() {
      this.tasks.push({ name: '', completed: false });
    },
  },
};
</script>

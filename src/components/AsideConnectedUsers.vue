<template>
  <div class="aside">
    <h2>Usuarios Conectados</h2>
    <p v-if="users.length === 0">No hay usuarios conectados</p>
    <ul v-else>
      <li v-for="(user, index) in users" :key="user.id || index">
        {{ user.nickname || user.slice(0, 5) }}
      </li>
    </ul>
  </div>

</template>

<script setup>
defineProps({
  users: {
    type: Array,
    required: true,
    default: () => ([])
  }
});
</script>

<style scoped>
.aside {
  padding: 0.5rem;
  border: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;height: 100%;
}

h2 {
  color: #2196f3;
  margin-bottom: 1rem;
  font-size: 1.2rem;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 0.5rem;
  margin-bottom: 0.3rem;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}
</style>
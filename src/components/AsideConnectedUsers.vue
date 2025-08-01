<template>
  <div class="aside">
    <h2>Usuarios Conectados</h2>
    <p v-if="users.length === 0">No hay usuarios conectados</p>
    <ul v-else>
      <li 
        v-for="(user, index) in users" 
        :key="user.id || index"
        @click="selectUser(user.id)"
        class="user-item"
      >
        {{ displayName(user) }}
      </li>
    </ul>
  </div>
</template>

<script setup>
const emit = defineEmits(['user-selected']);

const props = defineProps({
  users: {
    type: Array,
    required: true,
    default: () => ([])
  }
});

// Función para mostrar el nombre adecuado (truncado si es necesario)
const displayName = (user) => {
  if (user.nickname) {
    return user.nickname.length > 10 
      ? `${user.nickname.substring(0, 10)}...`
      : user.nickname;
  }
  return user.id.slice(0, 5);
};

// Función para seleccionar usuario (envía nickname si es diferente al ID)
const selectUser = (userId) => {
  const user = props.users.find(u => u.id === userId);
  const nameToSend = user?.nickname !== user?.id ? user?.nickname : userId;
  emit('user-selected', nameToSend || userId);
};
</script>

<style scoped>
.aside {
  padding: 0.5rem;
  border: 4mm ridge rgb(226, 94, 41);  padding: 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  margin: 0.5rem;
  border-radius: 14px;


}

h2 {
  color: #e91e63;
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
  cursor: pointer;
  background-color: #222;
  color: #f7420b;

}

li:hover {
  background-color: #f5f5f515;
}
</style>
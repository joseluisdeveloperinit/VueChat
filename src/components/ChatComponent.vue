<template>
  <div class="chat-container">    
    <div class="chat-main">
      <h1>Chat Anónimo</h1>
      <div class="message-inputs">
        <label for="nickname">Change nickname</label>
        <input
          v-model="nickname"
          @keyup.enter="sendNickname"
          placeholder="Escribe tu nickname"
        />
        <button @click="sendNickname">Guardar</button>
        <input 
          v-model="message" 
          @keyup.enter="sendPublicMessage" 
          placeholder="Mensaje público" 
        />
        <input 
          v-model="privateMessage" 
          @keyup.enter="sendPrivateMessage" 
          placeholder="Mensaje privado" 
        />
        <input 
          v-model="targetUser" 
          placeholder="ID del destinatario" 
        />
      </div>
      
      
      <div class="messages-panel">
        <div v-for="(msg, index) in messages" :key="index" class="message">
          <strong>{{ (msg.nickname || msg.from).slice(0, 5) }}</strong>: {{ msg.message }}
          <span v-if="msg.type === 'private'" class="private-tag">(Privado)</span>
        </div>
      </div>
    </div>

    
  </div>  

  
</template>

<script setup>
import { ref, onMounted } from 'vue';

const emit = defineEmits(['users-updated']);
const ws = ref(null);
const message = ref('');
const privateMessage = ref('');
const targetUser = ref('');
const messages = ref([]);
const users = ref([]);
const clientId = ref('');
const nickname = ref('');

// Conexión WebSocket
const connectWebSocket = () => {
  ws.value = new WebSocket('ws://localhost:8080/chat');

  ws.value.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'id') {
      clientId.value = data.message;
    } else {
      messages.value.push(data);
    }
  };

  ws.value.onclose = () => {
    console.log('Conexión cerrada');
  };
};

// Obtener usuarios conectados
const fetchConnectedUsers = async () => {
  try {
    const response = await fetch('http://localhost:8080/api/chat/users');
    users.value = await response.json();    
    emit('users-updated', users.value); // <-- Añade esta línea
  } catch (error) {
    console.error('Error fetching users:', error);
  }
};

// Enviar mensajes
const sendPublicMessage = () => {
  if (ws.value && message.value.trim()) {
    ws.value.send(JSON.stringify({
      type: 'public',
      message: message.value,
    }));
    message.value = '';
  }
};

const sendPrivateMessage = () => {
  if (ws.value && privateMessage.value.trim() && targetUser.value.trim()) {
    ws.value.send(JSON.stringify({
      type: 'private',
      to: targetUser.value,
      message: privateMessage.value,
    }));
    privateMessage.value = '';
  }
};

const sendNickname = () => {
  if (ws.value && nickname.value.trim()) {
    console.log(`el nickname ` + nickname.value);
    ws.value.send(JSON.stringify({
      type: 'nickname',
      nickname: nickname.value  // <-- aquí
    }));
  }
};


// Inicialización
onMounted(() => {
  connectWebSocket();
  fetchConnectedUsers();
  setInterval(fetchConnectedUsers, 5000);
});
</script>

<style scoped>
.chat-container {
  display: flex;
  height: 100vh;
}

.chat-main {
  flex: 1;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.message-inputs {
  display: flex;
  gap: 0.5rem;
}

.messages-panel {
  flex: 1;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 1rem;
  border-radius: 4px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start; /* Añade esto */
  height: 100%; /* Asegura altura completa */
  max-height: calc(100vh - 200px); /* Ajusta según tu layout */
}

.message {
  border-radius: 18px;
  word-wrap: break-word;
  text-align: left;
  margin: 0; /* Elimina márgenes por defecto */
}

.private-tag {
  color: #ff5722;
  font-size: 0.8rem;
  margin-left: 0.5rem;
}

h1 {
  color: #e91e63;
  margin-bottom: 1rem;
}
</style>
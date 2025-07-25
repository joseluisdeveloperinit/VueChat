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
          <strong>
            {{
              msg.nickname 
                ? (msg.nickname === msg.from ? msg.nickname.slice(0, 5) : msg.nickname)
                : (msg.from ? msg.from.slice(0, 5) : 'Anónimo')
            }}
          </strong>: {{ msg.message }}
          <span v-if="msg.type === 'private'" class="private-tag">(Privado)</span>
        </div>
      </div>
    </div>    
  </div>  
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const emit = defineEmits(['users-updated']);
const ws = ref(null);
const message = ref('');
const privateMessage = ref('');
const targetUser = ref('');
const messages = ref([]);
const users = ref([]);
const clientId = ref('');
const nickname = ref('');

const props = defineProps({
  selectedUserId: {
    type: String,
    default: ''
  }
});

// Watcher para actualizar targetUser cuando cambia selectedUserId
watch(() => props.selectedUserId, (newId) => {
  if (newId) {
    targetUser.value = newId;
  }
});

// Resto del código (connectWebSocket, fetchConnectedUsers, sendPublicMessage, etc.) permanece igual
const connectWebSocket = () => {
  ws.value = new WebSocket('ws://localhost:8080/chat');

  ws.value.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'id') {
      clientId.value = data.message;
    } else if (data.type === 'users-updated') {
      try {
        users.value = typeof data.message === 'string' 
          ? JSON.parse(data.message) 
          : data.message;
        emit('users-updated', users.value);
      } catch (e) {
        console.error('Error parsing users:', e);
      }
    } else {
      messages.value.push(data);
    }
  };

  ws.value.onerror = (error) => {
    console.error('WebSocket error:', error);
  };

  ws.value.onclose = () => {
    console.log('WebSocket connection closed');
  };
};

const fetchConnectedUsers = async () => {
  try {
    const response = await fetch('http://localhost:8080/api/chat/users');
    users.value = await response.json();    
    emit('users-updated', users.value);
  } catch (error) {
    console.error('Error fetching users:', error);
  }
};

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
    ws.value.send(JSON.stringify({
      type: 'nickname',
      nickname: nickname.value
    }));
    nickname.value = '';
    setTimeout(fetchConnectedUsers, 300);

  }
};

onMounted(() => {
  connectWebSocket();
  fetchConnectedUsers();
  setInterval(fetchConnectedUsers, 5000);
});
</script>

<style scoped>
/* Mantener el CSS existente sin cambios */
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
  justify-content: flex-start;
  height: 100%;
  max-height: calc(100vh - 200px);
}

.message {
  border-radius: 18px;
  word-wrap: break-word;
  text-align: left;
  margin: 0;
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
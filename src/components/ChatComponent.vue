<template>
  <div class="chat-container">    
    <div class="chat-main">
      <div class="border">
        <!-- <h1>Chat Anónimo</h1> -->
      </div>     
      <hr class="divider" />

      
      <!-- Agrega esta capa con clase .messages-panel -->
      <div class="messages-panel">
        <div v-for="(msg, index) in messages" :key="index" class="message">
          <strong class="message-sender">
            {{ formatDisplayName(msg) }}
          </strong>: {{ msg.message }}
          <span v-if="msg.type === 'private'" class="private-tag">(Privado)</span>
        </div>
      </div>
      <hr class="divider" />


        <div class="message-inputs">       
          <input 
            v-model="message" 
            @keyup.enter="sendPublicMessage" 
            placeholder="Public Message" 
          />
          <input 
            v-model="privateMessage" 
            @keyup.enter="sendPrivateMessage" 
            placeholder="Private Message" 
          />
          <input 
            v-model="targetUser" 
            placeholder="User Id" 
          />

          <input
            v-model="nickname"
            @keyup.enter="sendNickname"
            placeholder="Change your nickname"
          />
          <button @click="sendNickname">Save</button>
        </div>
        <div class="contact">
            <a href="mailto:contacto@empresa.com">Contacta con nosotros</a>
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

const formatDisplayName = (msg) => {
  if (msg.nickname) {
    const nameToDisplay = msg.nickname === msg.from ? msg.nickname.slice(0, 6) : msg.nickname;
    return nameToDisplay.length > 10 
      ? `${nameToDisplay.substring(0, 10)}...`
      : nameToDisplay;
  }
  return msg.from ? msg.from.slice(0, 6) : 'Anónimo';
};

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
  ws.value = new WebSocket('ws://192.168.1.200:8080/chat');

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
    const response = await fetch('http://192.168.1.200:8080/api/chat/users');
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
    setTimeout(fetchConnectedUsers, 100);

  }
};

const fetchMessageHistory = async () => {
  try {
    const response = await fetch('http://192.168.1.200:8080/api/chat/messages');
    const history = await response.json();
    messages.value = history; // Agrega los mensajes al estado
  } catch (error) {
    console.error('Error al cargar el historial:', error);
  }
};

onMounted(() => {
  connectWebSocket();
  fetchConnectedUsers();
  fetchMessageHistory(); 
  setInterval(fetchConnectedUsers, 5000);
});
</script>

<style scoped>
/* Mantener el CSS existente sin cambios */
.chat-container {
  display:flex;
  align-items: flex-start;
  justify-content: flex-start;
  padding: 0.5rem;
  height: 100vh;

  

}

.chat-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  border: 4mm ridge rgb(226, 94, 41);  padding: 0.5rem;
  padding: 0.5rem;
  overflow: hidden; /* ❗️ IMPORTANTE: evita que crezca la página */
  height: 100%;
    border-radius: 14px;

}

.message-inputs {
  display: flex;
  gap: 0.5rem;
  padding: 0.5rem;
  
}

.messages-panel {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  border-radius: 14px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  height: 100%;
  max-height: calc(100vh - 200px);

}

.message {
  text-align: left;
  background-color: #222;
  border-radius: 12px;
  padding: 0.5rem 1rem;
  margin-bottom: 0.5rem;
  font-size: 1rem;
  color: #ffffff;
  line-height: 1.4;
  max-width: 100%;
  word-break: break-word;
  overflow-wrap: break-word;
  box-sizing: border-box;
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.message-sender {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
  white-space: nowrap;    /* no permitir salto de línea */
  text-overflow: ellipsis;/* añade puntos suspensivos */
  font-weight: bold;
  color: #ff5722;
}


.private-tag {
  color: #ff5722;
  font-size: 0.8rem;
  margin-left: 0.5rem;
}

h1 {
  color: #e91e63;
}

input{

    border-radius: 18px;
    border-color: #ff5722;
    background-color: #181818;
    color: #7e837d;
}

button {
  border-radius: 18px;
  border: 2px solid #ff5722;
  background-color: #ff5722; /* Naranja como el borde */
  color: rgb(255, 255, 255);
  padding: 0.5rem 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  font-size: 0.8rem;
}

button:hover {
  background-color: #e64a19;
  transform: translateY(-1px);
}

input {
  border-radius: 18px;
  border: 1px solid #e64a19;
  background-color: #181818;
  color: #7e837d;
  padding: 0.5rem 1rem;
}

.border{
  display:  flex;
  padding: 1rem;

}
.divider {
  height: 1px;
  color: #ff5722; /* o cualquier color que uses */
  margin: 1rem 0;
}



</style>
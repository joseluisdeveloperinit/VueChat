<template>
  <div class="chat-container">    

    <div class="chat-main">
      <div class="border"></div> 
            <TabComponent
    :tabs="formattedTabs"
    :active-tab="activeTab"
    @tab-selected="changeTab"
    @tab-closed="closeTab"
  />    
      <hr class="divider" />

      <!-- Panel de mensajes -->
      <div class="messages-panel">
        <div v-for="(msg, index) in messages" :key="index" class="message">
          <strong class="message-sender">{{ formatDisplayName(msg) }}</strong>: 
          <span v-if="msg.isImage" class="image-message">
            [Imagen: {{ msg.imageName }}]
          </span>
          <span v-else>{{ msg.message }}</span>
          <span v-if="msg.type === 'private'" class="private-tag">(Privado)</span>
        </div>
      </div>
      <hr class="divider" />

      <!-- Inputs para mensajes -->
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
          placeholder="User ID" 
        />

        <!-- Input oculto para imágenes -->
        <input
          type="file"
          id="imageInput"
          ref="imageInput"
          accept="image/*"
          @change="handleImageSelect"
          style="display: none;"
        />
        <button @click="triggerImageInput">📷 Send Image</button>

        <input
          v-model="nickname"
          @keyup.enter="sendNickname"
          placeholder="Change nickname"
        />
        <button @click="sendNickname">Save</button>
      </div>

      <!-- Indicador de archivo seleccionado -->
      <div v-if="imageFile" class="file-indicator">
        <span class="file-name">{{ imageFile.name }}</span>
        <button @click="sendImageMessage" class="small-btn">Send</button>
        <button @click="cancelImage" class="small-btn cancel">×</button>
      </div>
      
      <div class="contact">
        <a href="mailto:contacto@empresa.com">Contacto</a>
      </div>
    </div>    
  </div>  
</template>
<script setup>
import { ref, onMounted, watch } from 'vue';
import { BASE_URL } from '@/constans/Base_url';
import TabComponent from './TabComponent.vue';
import { computed } from 'vue';

const emit = defineEmits(['users-updated']);
const ws = ref(null);
const message = ref('');
const privateMessage = ref('');
const targetUser = ref('');
const messages = ref([]);
const users = ref([]);
const clientId = ref('');
const nickname = ref('');
const imageInput = ref(null);
const selectedImage = ref(null);
const imageFile = ref(null);
const activeTab = ref('public');

const formatDisplayName = (msg) => {
  if (msg.nickname) {
    const nameToDisplay = msg.nickname === msg.from ? msg.nickname.slice(0, 6) : msg.nickname;
    return nameToDisplay.length > 10 
      ? `${nameToDisplay.substring(0, 10)}...`
      : nameToDisplay;
  }
  return msg.from ? msg.from.slice(0, 6) : 'Anónimo';
};

const formattedTabs = computed(() => {
  return [
    {
      id: 'public',
      title: 'Chat Público',
      unread: conversations.value.public.unread || 0,
      isOnline: true
    },
    ...Object.entries(conversations.value)
      .filter(([id]) => id !== 'public')
      .map(([id, conv]) => ({
        id,
        title: conv.nickname || id.slice(0, 6),
        unread: conv.unread || 0,
        isOnline: users.value.some(user => user.id === id)
      }))
  ];
});

// Estructura de conversaciones
const conversations = ref({
  public: {
    messages: [],
    unread: 0,
    nickname: 'Chat Público'
  }
});


const changeTab = (tabId) => {
  activeTab.value = tabId;
  // Resetear contador de no leídos
  if (conversations.value[tabId]) {
    conversations.value[tabId].unread = 0;
  }
};

const closeTab = (tabId) => {
  if (tabId === 'public') return;
  delete conversations.value[tabId];
  if (activeTab.value === tabId) {
    activeTab.value = 'public';
  }
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
  ws.value = new WebSocket('ws://127.0.0.1:8080/chat');

  ws.value.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    if (data.type === 'id') {
      clientId.value = data.message;
    } 
    else if (data.type === 'users-updated') {
      try {
        users.value = typeof data.message === 'string' 
          ? JSON.parse(data.message) 
          : data.message;
        emit('users-updated', users.value);
      } catch (e) {
        console.error('Error parsing users:', e);
      }
    }
    else {
      const conversationId = data.type === 'private' ? data.from : 'public';
      
      if (!conversations.value[conversationId]) {
        conversations.value[conversationId] = {
          messages: [],
          unread: 0,
          nickname: data.nickname || data.from.slice(0, 6)
        };
      }
      
      conversations.value[conversationId].messages.push({
        ...data,
        timestamp: new Date().toLocaleTimeString(),
        isImage: data.type === 'image',
        imageName: data.type === 'image' ? `image.${data.imageType}` : null
      });
      
      if (activeTab.value !== conversationId) {
        conversations.value[conversationId].unread++;
      }
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
    const response = await fetch( BASE_URL+'/chat/users');
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
    const response = await fetch(BASE_URL+'/chat/messages');
    const history = await response.json();
    messages.value = history; // Agrega los mensajes al estado
  } catch (error) {
    console.error('Error al cargar el historial:', error);
  }
};

const triggerImageInput = () => {
  if (!targetUser.value) {
    alert("Selecciona un usuario destino primero");
    return;
  }
  imageInput.value.click();
};

const handleImageSelect = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  // Validar tipo de archivo
  if (!file.type.match('image.*')) {
    alert("Solo se permiten archivos de imagen");
    return;
  }

  // Validar tamaño (ej. 1MB máximo)
  if (file.size > 1024 * 1024) {
    alert("La imagen es demasiado grande (máximo 1MB)");
    return;
  }

  // Crear preview
  const reader = new FileReader();
  reader.onload = (e) => {
    selectedImage.value = e.target.result;
    imageFile.value = file;
  };
  reader.readAsDataURL(file);
};

const sendImageMessage = async () => {
  if (!selectedImage.value || !targetUser.value || !ws.value) return;

  try {
    // Convertir imagen a base64
    const base64Image = await toBase64(imageFile.value);
    
    ws.value.send(JSON.stringify({
      type: "image",
      to: targetUser.value,
      imageData: base64Image.split(',')[1], // Quitar el prefijo
      imageType: imageFile.value.type.split('/')[1] // png, jpeg, etc.
    }));
    
    // Resetear después de enviar
    cancelImage();
    
  } catch (error) {
    console.error("Error al enviar imagen:", error);
    alert("Error al enviar la imagen");
  }
};


// Función auxiliar para convertir File a Base64
const toBase64 = (file) => new Promise((resolve, reject) => {
  const reader = new FileReader();
  reader.readAsDataURL(file);
  reader.onload = () => resolve(reader.result);
  reader.onerror = error => reject(error);
});






const cancelImage = () => {
  selectedImage.value = null;
  imageFile.value = null;
  if (imageInput.value) {
    imageInput.value.value = '';
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

.file-indicator {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem;
  background-color: #333;
  border-radius: 18px;
  margin-top: 0.5rem;
}

.file-name {
  flex-grow: 1;
  font-size: 0.9rem;
  color: #ccc;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.small-btn {
  padding: 0.25rem 0.5rem;
  font-size: 0.8rem;
  border-radius: 12px;
  background-color: #ff5722;
  border: none;
  cursor: pointer;
}

.small-btn.cancel {
  background-color: #ff3333;
  padding: 0.25rem 0.6rem;
}

.image-message {
  color: #4fc3f7;
  font-style: italic;
}

</style>
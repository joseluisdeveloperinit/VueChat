<template>
    <div>
        <AsideConnectedUsers :users="connectedUsers"></AsideConnectedUsers>
        <ChatComponent @users-updated="handleUsersUpdate"></ChatComponent>
        
    </div>
</template>
<script setup>
import { ref } from 'vue';
import AsideConnectedUsers from '@/components/AsideConnectedUsers.vue';
import ChatComponent from '@/components/ChatComponent.vue';

const connectedUsers = ref({});

const handleUsersUpdate = (users) => {
  connectedUsers.value = users;
};
</script>


<style>
div{
    display: flex;
    justify-content: center;
    
}

    
</style>
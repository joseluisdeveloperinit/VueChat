<template>
    <div>
      <Advertise></Advertise>
        <AsideConnectedUsers 
          :users="connectedUsers"
          @user-selected="handleUserSelected"
        ></AsideConnectedUsers>
        <ChatComponent 
          @users-updated="handleUsersUpdate"
          :selected-user-id="selectedUserId"
        ></ChatComponent>
        <Advertise></Advertise>

        
    </div>
</template>

<script setup>
import { ref } from 'vue';
import AsideConnectedUsers from '@/components/AsideConnectedUsers.vue';
import ChatComponent from '@/components/ChatComponent.vue';
import Advertise from '@/components/Advertise.vue';

const connectedUsers = ref([]);
const selectedUserId = ref('');

const handleUsersUpdate = (users) => {
  connectedUsers.value = users;
};

const handleUserSelected = (userId) => {
  selectedUserId.value = userId;
};
</script>

<style>
div {
    display: flex;
    justify-content: center;
}
</style>
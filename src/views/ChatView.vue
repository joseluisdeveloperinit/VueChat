<template>
    <div>
        <AsideConnectedUsers 
          :users="connectedUsers"
          @user-selected="handleUserSelected"
        ></AsideConnectedUsers>
        <ChatComponent 
          @users-updated="handleUsersUpdate"
          :selected-user-id="selectedUserId"
        ></ChatComponent>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import AsideConnectedUsers from '@/components/AsideConnectedUsers.vue';
import ChatComponent from '@/components/ChatComponent.vue';

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
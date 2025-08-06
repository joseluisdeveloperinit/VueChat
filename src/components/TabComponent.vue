<template>
  <div class="tabs-container">
    <div class="tabs-scroll">
      <div
        v-for="tab in tabs"
        :key="tab.id"
        class="tab"
        :class="{ 
          'active': activeTab === tab.id,
          'has-unread': tab.unread > 0,
          'is-private': tab.id !== 'public'
        }"
        @click="selectTab(tab.id)"
      >
        <span class="tab-label">
          {{ tab.title }}
          <span v-if="tab.id !== 'public'" class="user-status" :class="{'online': tab.isOnline}"></span>
        </span>
        
        <span v-if="tab.unread > 0" class="unread-count">
          {{ tab.unread > 9 ? '9+' : tab.unread }}
        </span>
        
        <button 
          v-if="tab.id !== 'public'" 
          class="close-tab"
          @click.stop="closeTab(tab.id)"
        >
          &times;
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  tabs: {
    type: Array,
    required: true,
    default: () => [
      {
        id: 'public',
        title: 'Chat Público',
        unread: 0,
        isOnline: true
      }
    ]
  },
  activeTab: {
    type: String,
    default: 'public'
  }
});

const emit = defineEmits(['tab-selected', 'tab-closed']);

const selectTab = (tabId) => {
  emit('tab-selected', tabId);
};

const closeTab = (tabId) => {
  emit('tab-closed', tabId);
};
</script>

<style scoped>
.tabs-container {
  background-color: #2d2d2d;
  border-bottom: 1px solid #444;
  padding: 0 0.5rem;
}

.tabs-scroll {
  display: flex;
  overflow-x: auto;
  scrollbar-width: thin;
  scrollbar-color: #ff5722 #444;
}

.tabs-scroll::-webkit-scrollbar {
  height: 6px;
}

.tabs-scroll::-webkit-scrollbar-track {
  background: #444;
}

.tabs-scroll::-webkit-scrollbar-thumb {
  background-color: #ff5722;
  border-radius: 3px;
}

.tab {
  position: relative;
  padding: 0.75rem 1.25rem;
  cursor: pointer;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  border-right: 1px solid #444;
  color: #ccc;
  font-size: 0.9rem;
  transition: all 0.2s ease;
}

.tab:hover {
  background-color: #3a3a3a;
}

.tab.active {
  background-color: #444;
  color: #ff5722;
  font-weight: bold;
}

.tab.is-private {
  padding-right: 2rem;
}

.tab-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.user-status {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #777;
}

.user-status.online {
  background-color: #4caf50;
}

.unread-count {
  background-color: #ff5722;
  color: white;
  border-radius: 10px;
  padding: 0.1rem 0.4rem;
  font-size: 0.7rem;
  font-weight: bold;
}

.close-tab {
  position: absolute;
  right: 0.5rem;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  font-size: 1.1rem;
  line-height: 1;
  padding: 0.2rem;
  border-radius: 50%;
  width: 1.2rem;
  height: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-tab:hover {
  background-color: #ff5722;
  color: white;
}

.has-unread {
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 87, 34, 0.4); }
  70% { box-shadow: 0 0 0 5px rgba(255, 87, 34, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 87, 34, 0); }
}
</style>
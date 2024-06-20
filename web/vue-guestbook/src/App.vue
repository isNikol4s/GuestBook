<template>
  <div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-4">Guestbook</h1>
    <form @submit.prevent="isEditing ? submitEditedMessage() : submitMessage()" class="mb-4">
      <div class="input-form">
        <input v-model="name" type="text" id="name" name="name" placeholder="BOOK'S NAME" class="border p-2 w-full">
      </div>
      <div class="input-form">
        <textarea v-model="text" id="text" name="text" placeholder="BOOK SYNOPSIS" class="border p-2 w-full"></textarea>
      </div>
      <div class="button-container">
        <button type="submit" class="submit bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">{{ isEditing ? 'Update' : 'To send' }}</button>
        <button type="button" @click="clearHistory" class="reset bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded">Reset History</button>
      </div>
    </form>
    <div v-if="messages.length > 0">
      <div v-for="message in messages" :key="message._id" class="mb-4 border-b pb-4">
        <p><strong>{{ message.name }}</strong> was read on {{ formatDate(message.date) }}:</p>
        <p>{{ message.text }}</p>
        <button @click="editMessage(message)" class="edit bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded">Edit</button>
      </div>
    </div>
    <div v-else>
      <p>No messages found.</p>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue';
import axios from 'axios';
axios.defaults.baseURL = 'http://localhost:3009/';

interface Message {
  _id: string;
  name: string;
  text: string;
  date: string;
}

export default defineComponent({
  name: 'Guestbook',
  setup() {
    const messages = ref<Message[]>([]);
    const name = ref<string>('');
    const text = ref<string>('');
    const isEditing = ref<boolean>(false);
    const editingMessageId = ref<string | null>(null);

    const loadMessages = async () => {
      try {
        const response = await axios.get<Message[]>('/api/messages');
        messages.value = response.data;
      } catch (error) {
        console.error('Erro ao carregar mensagens:', error);
      }
    };

    const submitMessage = async () => {
      if (name.value && text.value) {
        try {
          const response = await axios.post<Message>('/api/message/submit', {
            name: name.value,
            text: text.value
          });
          messages.value.unshift(response.data);
          name.value = '';
          text.value = '';
        } catch (error) {
          console.error('Erro ao enviar mensagem:', error);
        }
      }
    };

    const editMessage = (message: Message) => {
      name.value = message.name;
      text.value = message.text;
      isEditing.value = true;
      editingMessageId.value = message._id;
    };

    const submitEditedMessage = async () => {
      if (name.value && text.value && editingMessageId.value) {
        try {
          const response = await axios.put<Message>(`/api/message/${editingMessageId.value}`, {
            name: name.value,
            text: text.value
          });
          const index = messages.value.findIndex(msg => msg._id === editingMessageId.value);
          if (index !== -1) {
            messages.value[index] = response.data;
          }
          name.value = '';
          text.value = '';
          isEditing.value = false;
          editingMessageId.value = null;
        } catch (error) {
          console.error('Erro ao editar mensagem:', error);
        }
      }
    };

    const clearHistory = async () => {
      try {
        await axios.delete('/api/messages/clear');
        messages.value = [];
      } catch (error) {
        console.error('Erro ao limpar o histórico:', error);
      }
    };

    const formatDate = (dateStr: string) => {
      const date = new Date(dateStr);
      if (isNaN(date.getTime())) {
        return 'Data inválida';
      }
      const day = date.getDate();
      const month = date.getMonth() + 1;
      const year = date.getFullYear();
      return `${day}/${month}/${year}`;
    };

    onMounted(loadMessages);

    return {
      messages,
      name,
      text,
      isEditing,
      editingMessageId,
      submitMessage,
      submitEditedMessage,
      editMessage,
      clearHistory,
      formatDate
    };
  }
});
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
}

.input-form {
  margin-bottom: 1rem;
}

.button-container {
  display: flex;
  gap: 1rem;
}
</style>
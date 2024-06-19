<template>
  <div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-4">Guestbook</h1>
    <form @submit.prevent="submitMessage" class="mb-4">
      <div class="input-form">
        <input v-model="name" type="text" id="name" name="name" placeholder="BOOK'S NAME" class="border p-2 w-full">
      </div>
      <div class="input-form">
        <textarea v-model="text" id="text" name="text" placeholder="BOOK SYNOPSIS" class="border p-2 w-full"></textarea>
      </div>

      <div class="button-container">
        <button type="submit" class="submit bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">To send</button>
        <button type="button" @click="clearHistory" class="reset bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded">Reset History</button>
      </div>
    </form>
    <div v-if="messages.length > 0">
      <div v-for="message in messages" :key="message._id" class="mb-4 border-b pb-4">
        <p><strong>{{ message.name }}</strong> was read on {{ formatDate(message.date) }}:</p>
        <p>{{ message.text }}</p>
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
axios.defaults.baseURL = 'http://localhost:3008/'

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
      submitMessage,
      clearHistory,
      formatDate
    };
  }
});
</script>

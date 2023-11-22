<template>
    <div>
      <h1>{{ title }}</h1>
      <ul>
        <li v-for="item in items" :key="item.id">{{ item.username }}</li>
      </ul>
      <button @click="sendData">Send Data</button>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  // Define reactive variables
  const title = ref('Fetching Data Example');
  const items = ref([]);
  
  // Define a function to send a POST request with application/x-www-form-urlencoded content type
  const sendData = async () => {
    try {
      const postData = {
          created_date: "2023-11-16T23:15:31.328Z",
          name: "Marsha Feil",
          address: "9238 Annie Light",
          dob: "1980-10-31T11:31:22.444Z",
      }
     
  
      const response = await fetch('https://65571769bd4bcef8b6120891.mockapi.io/api/v1/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(postData),
      });
  
      if (response.ok) {
        console.log('Data sent successfully');
      } else {
        console.error('Failed to send data');
      }
    } catch (error) {
      console.error('An error occurred:', error);
    }
  };
  
  // Call fetchData when the component is mounted
  // onMounted(sendData);
  </script>
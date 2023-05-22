<template>
    <v-card elevation="2" >
        <form @submit.prevent="createUser">
            <label for="name">Name:</label>
            <input type="text" id="name" v-model="newUser.name" required>
            <button type="submit">Create</button>
        </form>
    </v-card>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newUser: {
          name: '',
          email: '',
          password: ''
        }
      };
    },
    methods: {
        createUser() {
        // Perform the create operation using API or other logic
        // Example: this.$http.post('/api/items', this.newUser)
        // Reset the form after successful creation
        this.newUser.name = '';
      }
    }
  };
  </script>
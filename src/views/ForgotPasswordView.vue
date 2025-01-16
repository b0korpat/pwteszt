<template>
  <div class="password-reset">
    <h2>Jelszó helyreállítás</h2>

    <form @submit.prevent="handleSubmit">
      <label for="email">Email:</label>
      <input v-model="emailInput" type="email" id="email" placeholder="Kérlek írd be az email címed" required />

      <button type="submit" :disabled="isLoading">
        {{ isLoading ? 'Küldés...' : 'Helyreállító küldése' }}
      </button>

      <div v-if="error" class="error">{{ error }}</div>
      <div v-if="success" class="success">Email sikeresen elküldve!</div>
    </form>
  </div>
</template>

<script>
import { supabase } from '../supabase';

export default {
  data() {
    return {
      emailInput: '',
      isLoading: false,
      error: null,
      success: false,
    };
  },
  methods: {
    async handleSubmit() {
      this.isLoading = true;
      this.error = null;
      this.success = false;

      const { error } = await supabase.auth.resetPasswordForEmail(this.emailInput, {
        redirectTo: `${window.location.origin}/reset-password`,
      });

      this.isLoading = false;

      if (error) {
        this.error = 'Could not authenticate user';
      } else {
        this.success = true;
      }
    },
  },
};
</script>

<style scoped>
.password-reset {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  text-align: center;
  background-color: #ffffff;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  font-family: 'Roboto', sans-serif;
}

h2 {
  margin-bottom: 20px;
  font-size: 1.8rem;
  color: #333;
  font-weight: 600;
}

label {
  display: block;
  margin-bottom: 8px;
  text-align: left;
  font-weight: 500;
  color: #555;
}

input[type="email"] {
  width: 400px; /* Change this to a fixed width */
  padding: 12px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
  transition: border-color 0.3s;
}

input[type="email"]:focus {
  border-color: #8746ef;
  outline: none;
  box-shadow: 0 0 0 3px rgba(135, 70, 239, 0.2);
}

button {
  width: 100%; /* Change this to a fixed width */
  max-width: 300px; /* Add a max-width */
  padding: 12px;
  background-color: #8746ef;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  transition: background-color 0.3s, transform 0.2s;
}

button:disabled {
  background-color: #b3a0e6;
  cursor: not-allowed;
}

button:not(:disabled):hover {
  background-color: #6f39c8;
}

button:not(:disabled):active {
  transform: scale(0.98);
}

.error {
  color: #e63946;
  margin-top: 10px;
  font-size: 0.9rem;
}

.success {
  color: #43aa8b;
  margin-top: 10px;
  font-size: 0.9rem;
}
</style>
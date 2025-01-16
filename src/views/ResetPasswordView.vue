<template>
  <div class="reset-password">
    <h1>Új jelszó</h1>
    <form @submit.prevent="updatePassword">
      <label for="password">Új jelszó:</label>
      <input type="password" id="password" v-model="password" required placeholder="Kérlek adj meg egy új jelszót" />

      <label for="confirm-password">Jelszó megerősítése:</label>
      <input type="password" id="confirm-password" v-model="confirmPassword" required placeholder="Erősítsd meg az új jelszót" />

      <button type="submit">Jelszó frissítése</button>
    </form>
    <div v-if="message" :class="['message', messageClass]">{{ message }}</div>
  </div>
</template>

<script>
import { supabase } from '../supabase';

export default {
  data() {
    return {
      password: '',
      confirmPassword: '',
      message: null,
      messageClass: null,
    };
  },
  methods: {
    async updatePassword() {
      if (this.password !== this.confirmPassword) {
        this.message = 'A jelszavak nem egyeznek!';
        this.messageClass = 'error';
        return;
      }

      const code = this.$route.query?.code;

      if (code) {
        const { error: sessionError } = await supabase.auth.exchangeCodeForSession(code);

        if (sessionError) {
          this.message = 'Sikertelen frissítés, A link lejárt!';
          this.messageClass = 'error';
          return;
        }
      }

      const { error } = await supabase.auth.updateUser({
        password: this.password,
      });

      if (error) {
        this.message = 'Sikertelen frissítés, próbáld újra!';
        this.messageClass = 'error';
      } else {
        this.message = 'Jelszó sikeresen frissítve!';
        this.messageClass = 'success';
      }
    },
  },
};
</script>

<style scoped>
.reset-password {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  background-color: #ffffff;
  border-radius: 12px;
  text-align: center;
  font-family: 'Roboto', sans-serif;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

h1 {
  font-size: 1.8rem;
  margin-bottom: 20px;
  color: #333;
  font-weight: 600;
}

label {
  display: block;
  text-align: left;
  margin-bottom: 10px;
  font-weight: 500;
  color: #555;
}

input[type="password"] {
  width: 400px; 
  padding: 12px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
  transition: border-color 0.3s;
}

input[type="password"]:focus {
  border-color: #8746ef;
  outline: none;
  box-shadow: 0 0 0 3px rgba(135, 70, 239, 0.2);
}

button {
  width: 100%;
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

button:hover {
  background-color: #6f39c8;
}

button:active {
  transform: scale(0.98);
}

.message {
  margin-top: 10px;
  font-size: 0.9rem;
}

.message.error {
  color: #e63946;
}

.message.success {
  color: #43aa8b;
}
</style>

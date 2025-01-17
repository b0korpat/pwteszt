<template>
  <div class="register">
    <h1>Regisztráció</h1>
    <form @submit.prevent="register">
      <label for="email">Email:</label>
      <input type="email" id="email" v-model="email" @blur="validateEmailField" required placeholder="Kérlek add meg az emailcímed" />
      <div v-if="errorMessage.email" class="error">{{ errorMessage.email }}</div>

      <label for="password">Jelszó:</label>
      <input type="password" id="password" v-model="password" @blur="validatePasswordField" required placeholder="Kérlek adj meg egy jelszót" />
      <div v-if="errorMessage.password" class="error">{{ errorMessage.password }}</div>

      <label for="confirmPassword">Jelszó megerősítése:</label>
      <input type="password" id="confirmPassword" v-model="confirmPassword" @blur="validateConfirmPasswordField" required placeholder="Kérlek erősítsd meg a jelszót" />
      <div v-if="errorMessage.confirmPassword" class="error">{{ errorMessage.confirmPassword }}</div>

      <label for="firstName">Keresztnév:</label>
      <input type="text" id="firstName" v-model="firstName" @blur="validateFirstNameField" required placeholder="Kérlek add meg a keresztneved" />
      <div v-if="errorMessage.firstName" class="error">{{ errorMessage.firstName }}</div>

      <label for="lastName">Vezetéknév:</label>
      <input type="text" id="lastName" v-model="lastName" @blur="validateLastNameField" required placeholder="Kérlek add meg a vezetékneved" />
      <div v-if="errorMessage.lastName" class="error">{{ errorMessage.lastName }}</div>

      <label for="birthdate">Születési dátum:</label>
      <input type="date" id="birthdate" v-model="birthdate" required />

      <label for="gender">Nem:</label>
      <select id="gender" v-model="gender" required>
        <option value="" disabled selected>Válassz nemet</option>
        <option value="male">Férfi</option>
        <option value="female">Nő</option>
        <option value="other">Egyéb</option>
      </select>

      <button type="submit">Regisztráció</button>
    </form>
    <div v-if="message" :class="['message', messageClass]">{{ message }}</div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { supabase } from '@/supabase';

export default {
  setup() {
    const email = ref('');
    const password = ref('');
    const confirmPassword = ref('');
    const firstName = ref('');
    const lastName = ref('');
    const birthdate = ref('');
    const gender = ref('');
    const message = ref(null);
    const messageClass = ref(null);
    const errorMessage = ref({
      email: '',
      password: '',
      confirmPassword: '',
      firstName: '',
      lastName: ''
    });
    const router = useRouter();

    const validateEmail = (email) => {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    };

    const validateEmailField = () => {
      if (!email.value) {
        errorMessage.value.email = 'Az email mező kitöltése kötelező';
      } else if (!validateEmail(email.value)) {
        errorMessage.value.email = 'Érvénytelen email formátum';
      } else {
        errorMessage.value.email = '';
      }
    };

    const validatePasswordField = () => {
      if (!password.value) {
        errorMessage.value.password = 'A jelszó mező kitöltése kötelező';
      } else if (password.value.length < 6) {
        errorMessage.value.password = 'A jelszónak legalább 6 karakter hosszúnak kell lennie';
      } else {
        errorMessage.value.password = '';
      }
    };

    const validateConfirmPasswordField = () => {
      if (!confirmPassword.value) {
        errorMessage.value.confirmPassword = 'A jelszó megerősítése kötelező';
      } else if (password.value !== confirmPassword.value) {
        errorMessage.value.confirmPassword = 'A jelszavak nem egyeznek';
      } else {
        errorMessage.value.confirmPassword = '';
      }
    };

    const validateFirstNameField = () => {
      if (!firstName.value) {
        errorMessage.value.firstName = 'A keresztnév mező kitöltése kötelező';
      } else if (firstName.value.length < 3) {
        errorMessage.value.firstName = 'A keresztnévnek legalább 3 karakter hosszúnak kell lennie';
      } else {
        errorMessage.value.firstName = '';
      }
    };

    const validateLastNameField = () => {
      if (!lastName.value) {
        errorMessage.value.lastName = 'A vezetéknév mező kitöltése kötelező';
      } else if (lastName.value.length < 3) {
        errorMessage.value.lastName = 'A vezetéknévnek legalább 3 karakter hosszúnak kell lennie';
      } else {
        errorMessage.value.lastName = '';
      }
    };

    const register = async () => {
      validateEmailField();
      validatePasswordField();
      validateConfirmPasswordField();
      validateFirstNameField();
      validateLastNameField();

      if (Object.values(errorMessage.value).some(msg => msg)) {
        return;
      }

      const { data, error } = await supabase.auth.signUp({ email: email.value, password: password.value });
      if (error) {
        errorMessage.value.email = 'Hiba történt a regisztráció során: ' + error.message;
      } else {
        const user = data.user;
        if (user) {
          const { error: updateError } = await supabase
            .from('profiles')
            .update({ first_name: firstName.value, last_name: lastName.value, birthday: birthdate.value, gender: gender.value })
            .eq('id', user.id);
          if (updateError) {
            message.value = 'Hiba történt a felhasználói adatok frissítése során: ' + updateError.message;
            messageClass.value = 'error';
          } else {
            message.value = 'A profil létrehozva';
            messageClass.value = 'success';
          }
        }
      }
    };

    return {
      email,
      password,
      confirmPassword,
      firstName,
      lastName,
      birthdate,
      gender,
      message,
      messageClass,
      errorMessage,
      validateEmailField,
      validatePasswordField,
      validateConfirmPasswordField,
      validateFirstNameField,
      validateLastNameField,
      register
    };
  }
};
</script>

<style scoped>
.register {
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

input[type="email"],
input[type="password"],
input[type="date"],
input[type="text"],
select {
  width: 100%;
  padding: 12px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  box-sizing: border-box;
  transition: border-color 0.3s;
}

input[type="email"]:focus,
input[type="password"]:focus,
input[type="date"]:focus,
input[type="text"]:focus,
select:focus {
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

.error {
  color: #e63946;
  font-size: 0.8rem;
  margin-bottom: 10px;
}
</style>

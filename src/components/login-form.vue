<template>
  <div class="login-container">
    <h2>Login</h2>
    <form @submit.prevent="login">
      <input v-model="username" placeholder="Username" type="text" id="username" @input="handleUsernameInput" :class="{ 'error': usernameError }" />
      <input v-model="phoneNumber"  placeholder="Phone number" type="text" id="phoneNumber" @input="handlePhoneNumberInput" :class="{ 'error': phoneNumberError }" />
      <button type="submit">Register</button>
    </form>
    <div class="error">
      <p v-if="loginError" >Login error</p>
    </div>

  </div>
</template>

<script>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

export default {
  setup() {
    const router = useRouter();
    const username = ref('');
    const phoneNumber = ref('');
    const usernameError = ref(false);
    const phoneNumberError = ref(false);
    const loginError = ref(false);

    const login = async () => {
      if (!usernameError.value && !phoneNumberError.value) {
        const users = await fetchUsers();
        const user = users.find(u => u.username === username.value && u.phone.replace(/[^+\d]/g, '') === phoneNumber.value.replace(/[^+\d]/g, ''));

        if (user) {
          localStorage.setItem('user', JSON.stringify(user));
          router.push('/todo');
        } else {
          loginError.value = true;
        }
      }
    };

    const fetchUsers = async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      return await response.json();
    };

    const handleUsernameInput = () => {
      username.value = username.value.replace(/[^a-zA-Z]/g, '');
      loginError.value = false;
    };

    const handlePhoneNumberInput = () => {
      phoneNumber.value = phoneNumber.value.replace(/[^x0-9+\-()\s]/g, '');
      loginError.value = false;
    };

    return {
      username,
      phoneNumber,
      usernameError,
      phoneNumberError,
      loginError,
      login,
      handleUsernameInput,
      handlePhoneNumberInput,
    };
  },
};
</script>

<style lang="scss" scoped>
.login-container {
  width: 447px;
  margin: 50px auto;
  padding: 20px;
  padding-bottom: 0;
  border-radius: 5px;
  background-color: #f0f0f0;
  @media screen and (max-width: 968px) {
    width: 80%;
  }
}

form {
  display: flex;
  flex-direction: column;
}

label {
  margin-bottom: 5px;
}

input {
  margin-bottom: 10px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

button {
  padding: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

.error {
  color: red;
  font-size: 14px;
  margin-top: 14px;
  margin-bottom: 14px;
  height: 25px;
}
</style>

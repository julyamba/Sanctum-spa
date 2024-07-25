<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref } from 'vue';
import axios from 'axios';

axios.defaults.withCredentials = true
axios.defaults.baseURL = 'http://localhost:8000';

const form = ref({
  email: null,
  password: null
})

function getCookie(name) {
  const value = document.cookie
  const parts = value.split(name)
  if (parts.length === 2) {
    return parts.pop().split(';').shift()
  }
}

const user = ref()

async function handleSubmit() {
  try {
    await axios.get("/sanctum/csrf-cookie",{withCredentials:true})
    const csrfToken = getCookie('XSRF-TOKEN=')
    await axios.post("/login", {
      email: form.value.email,
      password: form.value.password,
    }, {
      headers: {
      'X-XSRF-TOKEN': decodeURIComponent(csrfToken)
      },
        withCredentials: true
    });
    let { data } = await axios.get("/api/user")
    user.value = data
  } catch (error) {
    alert('Failed to submit the form.');
  }
}


async function logout() {
  try {
    await axios.post('/logout', {}, {
      headers: {
        'X-XSRF-TOKEN': decodeURIComponent(getCookie('XSRF-TOKEN='))
      }, withCredentials: true
    });
    user.value = []
  } catch (error) {
    alert('Failed to logout.');
  }
}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  <main>
    {{ user }}
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="name">Email:</label>
        <input type="text" id="name" v-model="form.email" required />
      </div>
      <div>
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="form.password" required />
      </div>
      <button type="submit">Login</button>
    </form>
    <div class="logout">
      <button style="width: 100%; background-color: blueviolet; color: white;" type="button" @click="logout">Logout</button>
    </div>
    
  </main>

</template>

<style scoped>
form {
  max-width: 400px;
  margin: 0 auto;
  padding: 1em;
  background: #f9f9f9;
  border-radius: 5px;
}

div {
  margin-bottom: 1em;
}

label {
  margin-bottom: .5em;
  color: #333333;
  display: block;
}



input {
  border: 1px solid #CCCCCC;
  padding: .5em;
  font-size: 1em;
  width: 100%;
  box-sizing: border-box;
}

button {
  padding: 0.7em;
  color: #fff;
  background-color: #007BFF;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.logout {
  max-width: 400px;
  margin: 0 auto;
  margin-top: 10px;
}


@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>

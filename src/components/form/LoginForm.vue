<template>
  <div  class="login-container ">
    <div class="login-card">
    <div class="mt-3 is-flex flex-direction-column justify-content-center align-items-center">
      <img class="logo" src="/public/pwa-512x512.png" alt="">
    <div class="login-form form-wrapper">
      <h2>Login</h2>
      <form @submit.prevent="handleLogin">
        <div class="form-group">
          <label for="email">Username</label>
          <input class="input-text-modal" id="username" v-model="username" placeholder="Enter your email" required />
        </div>

        <div class="form-group">
          <label for="password">Password</label>
          <input class="input-text-modal" id="password" type="password" v-model="password"
            placeholder="Enter your password" required />
        </div>

        <div class="form-actions">
          <button type="submit" class="btn btn-primary">Login</button>
          <button type="button" class="btn btn-secondary" @click="goToSignup">Signup</button>
        </div>
      </form>
    </div>
    </div>
  </div>
  </div>

</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import { useRouter } from 'vue-router'
  import { LOGIN_STATUS } from '@/stores/types'
  import { useSeesionStore } from '@/stores/session'

  // State for form inputs
  const username = ref('')
  const password = ref('')

  // Access router instance to navigate between routes
  const router = useRouter()
  const sessionStore = useSeesionStore()

  // Handle login logic
  const handleLogin = async () => {
    if (username.value && password.value) {
      console.log('Logging in with:', { email: username.value, password: password.value })

      const respond = await sessionStore.loginUser(username.value, password.value)

      if (respond == LOGIN_STATUS.SUCCESS) {
        console.log('@__ Login Success!!!')
      } else {
        alert(respond)
      }

      // On successful login, navigate to user dashboard
    } else {
      alert('Please enter both email and password.')
    }
  }

  // Navigate to the Signup page
  const goToSignup = () => {
    router.push('/register')
  }
</script>

<style scoped>

  .login-form {
  background: linear-gradient(135deg, #51f15ec0, #4056d6c5);
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 100%;
  max-width: 400px;
  }

@keyframes shake {
  0% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  50% { transform: translateX(10px); }
  75% { transform: translateX(-10px); }
  100% { transform: translateX(0); }
}

.shake {
  animation: shake 0.5s ease-in-out;
}
@keyframes fadeIn {
  0% { opacity: 0; transform: scale(0.9); }
  100% { opacity: 1; transform: scale(1); }
}

.form-wrapper {
  animation: fadeIn 1s ease-in-out;
}


  .form-group {
    margin-bottom: 20px;
  text-align: left;
  }

  .form-actions {
    display: flex;
    justify-content: space-between;
  }

  label {
  font-size: 15px;
  color: #111010;
  display: block;
  margin-bottom: 5px;
}

.input-text-modal {
  width: 95%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 20px;
  font-size: 14px;
  outline: none;
  transition: all 0.3s;
}

.input-text-modal:focus {
  border-color: #2e7d32;
  box-shadow: 0 0 10px rgba(46, 125, 50, 0.3);
}

  .btn-primary,
.btn-secondary {
  display: inline-block;
  width: 100%;
  padding: 10px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 20px;
  cursor: pointer;
  margin-bottom: 5px;
  transition: all 0.3s;
}
.btn-primary {
  background: #2e7d32;
  color: white;
  border: none;
}

.btn-primary:hover {
  background: #1b5e20;
}

.btn-secondary {
  background: white;
  color: #2e7d32;
  border: 2px solid #2e7d32;
}

.btn-secondary:hover {
  background: #f1f8e9;
}
.form-actions {
  display: flex;
  flex-direction: column; /* Stack the buttons vertically */
  gap: 10px; /* Adds space between the two buttons */
}
</style>
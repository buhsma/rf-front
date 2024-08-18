<template>
    <main class="main">
        <form class="inputForm" @submit.prevent="login">
            <h1>Sign in</h1>
            <div class="inputField">
                <input type="email" id="email" v-model="email" required>
                <label for="email">Email</label>
            </div>
            <div class="inputField">
                <input type="password" id="password" v-model="password" required>
                <label for="password">Password</label>
                
            </div>
            <p class="forgotText">Forgot password? <router-link to="/reset-password">Reset</router-link></p>
            <button class="btn1" type="submit">Login</button>
            <p>Don't have an account? <router-link to="/register">Register</router-link></p>
            <p>{{ serverResponse }}</p>
        </form>
    </main>
</template>


<script>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import authState from '@/tools/authState'

export default {
    setup() {
        const email = ref('')
        const password = ref('')
        const serverResponse = ref('')
        const router = useRouter()

        const { setAuthState } = authState()

        const login = () => {
            axios.post('/api/token/', {
                username: email.value,
                password: password.value
            })
                .then(function (response) {
                    console.log(response)
                    serverResponse.value = 'Login successful: ' + response.status
                    localStorage.setItem('access', response.data.access)
                    localStorage.setItem('refresh', response.data.refresh)
                    setAuthState(true);
                    router.push({ name: 'dashboard' })
                })
                .catch(error => {
                    if (error.response) {
                        serverResponse.value = 'Login failed with status code: ' + error.response.status + ' ' + error.response.data.detail
                    } else if (error.request) {
                        serverResponse.value = 'Login failed: No response from server'
                    } else {
                        serverResponse.value = 'Login failed: ' + error.message
                    }
                    console.error(error)
                })
        }

        return {
            email,
            password,
            login,
            serverResponse
        }
    }
}
</script>
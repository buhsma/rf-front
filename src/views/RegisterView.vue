<template>
    <main class="main">
        <form class="inputForm" @submit.prevent="register">
            <h1>Create account</h1>
            <div class="inputField">
                <input type="email" id="email" v-model="email" required>
                <label for="email">Email</label>
            </div>
            <div class="inputField">
                <input type="password" id="password" v-model="password" required>
                <label for="password">Password</label>
            </div>
            <div class="inputField">
                <input type="password" id="confirmPassword" v-model="confirmPassword" required>
                <label for="confirmPassword">Confirm Password</label>
            </div>
            <button class="btn1" type="submit">Register</button>
            <p>Already have an account? <router-link to="/login">Login</router-link></p>
            <p>{{ serverResponse }}</p>

        </form>
    </main>
</template>

<script>
import { ref } from 'vue'
import axios from 'axios'

export default {
    setup() {
        const email = ref('')
        const password = ref('')
        const confirmPassword = ref('')
        const serverResponse = ref('')

        const register = () => {
            if (password.value !== confirmPassword.value) {
                serverResponse.value = 'Passwords do not match'
                return
            }

            axios.post('/api/register', {
                email: email.value,
                password: password.value
            })
                .then(function (response) {
                    console.log(response)
                    serverResponse.value = 'Registration successful: ' + response.status
                    localStorage.setItem('access', response.data.access)
                    localStorage.setItem('refresh', response.data.refresh)
                })
                .catch(error => {
                    if (error.response) {
                        serverResponse.value = 'Registration failed with status code: ' + error.response.status + ' ' + error.response.data.error
                    } else if (error.request) {
                        serverResponse.value = 'Registration failed: No response from server'
                    } else {
                        serverResponse.value = 'Registration failed: ' + error.message
                    }
                    console.error(error)
                })
        }

        return {
            email,
            password,
            confirmPassword,
            register,
            serverResponse
        }
    }
}
</script>

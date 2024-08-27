<template>
    <main class="main">
        <form class="inputForm" @submit.prevent="resetPassword">
            <h1>Enter your new password</h1>
            <div class="inputField">
                <input type="password" id="password" v-model="password" required>
                <label for="password">Password</label>
            </div>
            <div class="inputField">
                <input type="password" id="password2" v-model="password2" required>
                <label for="password2">Confirm Password</label>
            </div>
            <button class="btn1" type="submit">Reset Password</button>
            <p>{{ serverResponse }}</p>
        </form>
    </main>
</template>

<script>
import { ref } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'

export default {
    setup() {
        const password = ref('')
        const password2 = ref('')
        const serverResponse = ref('')
        const route = useRoute()
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;

        const resetPassword = () => {
            if (password.value !== password2.value) {
                serverResponse.value = 'Passwords do not match'
                return
            }
            else {
                axios.post(`${BACKEND_URL}/api/reset_password/${route.params.id}/${route.params.token}/`, {
                    password: password.value,
                })
                    .then(function () {
                        serverResponse.value = 'Password reset successful'
                    })
                    .catch(error => {
                        if (error.response) {
                            serverResponse.value = 'Password reset failed with status code: ' + error.response.status + ' ' + error.response.data.detail
                        } else if (error.request) {
                            serverResponse.value = 'Password reset failed: No response from server'
                        } else {
                            serverResponse.value = 'Password reset failed: ' + error.message
                        }
                        console.error(error)
                    })
            }
        }

        return {
            password,
            password2,
            serverResponse,
            resetPassword

        }
    }
}
</script>
<template>
    <main class="main">
        <form v-if="status === 'ready'" class="inputForm" @submit.prevent="resetPassword">
            <h1>Reset Password</h1>
            <div class="inputField">
                <input type="email" id="email" v-model="email" required>
                <label for="email">Email</label>
            </div>
            <button class="btn1" type="submit">Reset Password</button>
            <p>{{ serverResponse }}</p>
        </form> 
        <div v-else class="inputForm">
            <h1>Reset Password</h1>
            <p>We have sent you an email with a link to reset your password.</p>
        </div>
    </main>
</template>

<script>
import { ref } from 'vue'
import axios from 'axios'

export default {
    setup() {
        const email = ref('')
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;
        const status = ref('ready')

        const resetPassword = () => {
            axios.post(`${BACKEND_URL}/api/reset_password/`, {
                email: email.value
            })
                .then(function () {
                    status.value = 'sent'
                })
        }

        return {
            email,
            status,
            resetPassword
        }
    }
}
</script>
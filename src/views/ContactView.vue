<template>
    <main class="main">
        
        <form class="inputForm" @submit.prevent="submitContactForm">
            <h1>Contact Us</h1>
        <p>Use the form below or email us at hello@relayfox.com.</p>
            <div class="inputField">
                <input type="email" id="email" v-model="email" required>
                <label for="email">Your Email</label>
            </div>
            <div class="inputField">
                <input type="text" id="subject" v-model="subject" required>
                <label for="subject">Subject</label>
            </div>
            <div class="inputField">
                <textarea id="message" v-model="message" rows="5" required></textarea>
                <label for="message">Your Message</label>
            </div>
            <button class="btn1" type="submit">Submit</button>
            <p>{{ serverResponse }}</p>
        </form>
    </main>
</template>

<style lang="scss" scoped>
#message {
    padding: 10px;
    height: auto;
}
p {
    text-wrap: wrap;
    text-align: center;
}
</style>

<script>

import { ref } from 'vue'
import axios from 'axios'

export default {
    setup() {
        const email = ref('')
        const subject = ref('')
        const message = ref('')
        const serverResponse = ref('')
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;
        const submitContactForm = () => {
            axios.post(`${BACKEND_URL}/api/contact`, {
                email: email.value,
                subject: subject.value,
                message: message.value
            })
                .then(function (response) {
                    console.log(response)
                    serverResponse.value = 'Message sent: ' + response.status
                    alert('Message sent: ' + response.status)
                })
                .catch(error => {
                    if (error.response) {
                        serverResponse.value = 'Message failed with status code: ' + error.response.status + ' ' + error.response.data.error
                        alert('Message failed with status code: ' + error.response.status + ' ' + error.response.data.error)
                    } else if (error.request) {
                        serverResponse.value = 'Message failed: No response from server'
                        alert('Message failed: No response from server')
                    } else {
                        serverResponse.value = 'Message failed: ' + error.message
                        alert('Message failed: ' + error.message)
                    }
                    console.error(error)
                })
        }
        return { email, subject, message, submitContactForm }
    }
}

</script>
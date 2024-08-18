<template>

    <div class="inputContainer">
        <div class="field" :class="{ 'dragOnField': dragOnField, 'dragOnWindow': dragOnWindow }" @dragover.prevent
            @drop="handleDrop">
            <textarea v-model="inputText" :class="{ 'dragOnField': dragOnField, 'dragOnWindow': dragOnWindow }"
                :placeholder="dragOnWindow ? '' : placeholderText"></textarea>
            <div v-show="inputText" class="inputOverlay">
                <p>{{ maxChars - inputText.length }} characters left</p>
                <p v-if="!userIsAuthenticated">Login or create a free account to send long texts</p>
            </div>
            <div v-show="!inputText" class="inputOverlay">
                <p>{{ maxFileSize }}</p>
            </div>

            <div class="placeholder" v-show="!inputText">
                <div class="dragText" v-if="!dragOnField && dragOnWindow">
                    <span class="material-icons material-icons-outlined">backup</span>
                    <p>Drop here</p>
                </div>
                <div class="dragText" v-else-if="dragOnField">
                    <span class="material-icons material-icons-outlined">backup</span>
                    <p>Drop</p>
                </div>
                <div class="box" v-else>
                    <span class="material-icons material-icons-outlined">backup</span>
                    <button class="btn1 selectFile" @click="clickFileInput">Select a file
                    </button>


                    <p class="pl3">or drag and drop it here</p>

                </div>
                
            </div>
            <button v-show="!inputText" class="mobileBtn btn1 selectFile" @click="clickFileInput">Select a file</button>

            
                
            
        </div>


        <div v-show="inputText" class="spacer">
        </div>
        <div class="linkLife">
            <label for="linkLifetime">Destroy after:</label>
            <dropdown />
        </div>

        <button v-show="inputText.length > 0 && inputText.length < maxChars + 1" class="sendBtn btn1"
            @click="emitInput">Send</button>

        <input type="file" ref="fileInput" style="display: none" @change="handleFileSelect" />
    </div>

</template>
<style lang="scss" scoped>
.inputContainer {
    position: relative;
    grid-column: 2;
    gap: 1rem;
    margin-top: 2rem;

    & .spacer {
        margin-bottom: 3rem;

        @media (max-width: 768px) {
            margin-bottom: 0;
        }
    }

    & .sendBtn {
        position: absolute;
        bottom: -3.5rem;
        right: 0;

        @media (max-width: 768px) {
            bottom: 15px;
            z-index: 1000;
            margin-right: 1rem;
            padding-top: 10px;
            padding-bottom: 10px;
        }

        @media (max-width: 465px) {
            padding: 10px 12px;

        }
    }

    & .linkLife {
        position: absolute;
        left: 0;
        bottom: 0;
        background-color: var(--background-light);
        padding: .8rem;
        // transform: translateY(-7.5rem);
        margin-left: 5.5px;
        margin-bottom: 5.5px;
        z-index: 10;
        border-top-right-radius: var(--border-radius);
        border-bottom-left-radius: var(--border-radius);

        @media (max-width: 768px) {
            position: relative;
            margin-top: 1rem;
            text-align: left;
            border-radius: var(--border-radius);
            border: 2px solid var(--primary);
        }

        & label {
            margin-bottom: 1rem;


        }
    }

    & .field {
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 1rem;
        // padding: 1rem;
        // border: 2px dashed var(--primary);
        border-radius: var(--border-radius);
        width: 100%;
        height: 22rem;
        overflow: hidden;
        box-sizing: border-box;

        @media (max-width: 768px) {
            height: 11rem;
        }


        &::before {
            content: '';
            position: absolute;
            z-index: 0;
            left: -150%;
            top: -150%;
            width: 400%;
            height: 400%;
            background-color: var(--background);
            background-repeat: no-repeat;
            background-position: 0 0;
            background-image: conic-gradient(transparent, var(--primary-dark), transparent 30%);

            animation: rotate 4s linear infinite;
        }

        &.dragOnWindow::before {
            background-image: conic-gradient(var(--primary) 0%, transparent 25%, var(--primary) 50%, transparent 75%, var(--primary) 100%);
        }

        &.dragOnField::before {
            background-image: conic-gradient(var(--primary-light) 0%, transparent 12.5%, var(--primary-light) 25%, transparent 37.5%, var(--primary-light) 50%, transparent 62.5%, var(--primary-light) 75%, transparent 87.5%, var(--primary-light) 100%);
        }
        & .mobileBtn {
            display: none;
            position: absolute;
            z-index: 10;
            margin-top: 3rem;
            @media (max-width: 768px) {
                display: block;
            }
        }

        & .placeholder {
            color: var(--primary);
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            gap: 1rem;
            // padding: 1rem;
            // border: 1px solid var(--primary);
            border-radius: var(--border-radius);
            pointer-events: none;
            z-index: 6;

            @media (max-width: 768px) {
                display: none;
            }

            & p {
                margin: 0;
            }

            & .box {
                display: flex;
                flex-direction: column;
                margin-top: 5rem;

                border: 2px dashed grey;
                border-radius: var(--border-radius);
                padding: 1rem 2rem;

                & span {
                    font-size: 6rem;
                }

                & p {
                    margin-top: .5rem;
                    font-size: 1.2rem;
                }
            }

            & .dragText {
                font-size: 3rem;
                display: flex;
                align-items: center;
                gap: 1rem;

                .material-icons {
                    font-size: 5rem; // replace with the size you want
                }
            }
        }

        & textarea {
            position: relative;
            padding: 1.5rem;
            font-size: 1.3rem;
            border: none;
            border-radius: var(--border-radius);
            background: var(--background-dark);
            overflow: auto;
            white-space: pre-wrap;
            resize: none;
            z-index: 5;
            width: calc(100% - 12px);
            height: calc(100% - 12px);
            color: #eee;
            @media (max-width: 510px) {
                    font-size: 1rem;
                    padding: 1rem;
                }



            &::placeholder {
                text-align: left;
                color: #ccc;
                // @media (max-width: 510px) {
                //     font-size: 1rem;
                // }
            }

            &.dragOnWindow {
                background: var(--background-light);
            }

            &.dragOnField {
                background: #2a7151;
            }

        }

        & .inputOverlay {
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            position: absolute;
            bottom: 0;
            right: 0;
            color: #ccc;
            padding: 1rem;
            font-size: .8rem;
            z-index: 10;
        }
        & .inputOverlay p:nth-child(2) {
            @media (max-width: 552px) {
                display: none;
            }
        }

    }
}

.selectFile {
    font-size: .9rem;
    cursor: pointer;
    pointer-events: all;
}
</style>
<script>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';
import authState from '@/tools/authState';
import dropdown from '@/components/dropdown.vue';


export default {
    components: {
        dropdown,
    },
    emits: ['input', 'file'],
    setup(_, { emit }) {
        const inputText = ref('');
        const fileInput = ref(null);
        const linkLifetime = ref(1);
        const dragOnField = ref(false);
        const dragOnWindow = ref(false);
        const { userIsAuthenticated } = authState();
        const maxChars = ref(0);
        const maxFileSize = ref()
        const placeholderText = ref('Type your secret here');

        const updatePlaceholderText = () => {
            if (window.innerWidth <= 768) {
                placeholderText.value = 'Type your secret here or';
            } else {
                placeholderText.value = 'Type your secret here';
            }
        };

        const setMaxChars = () => {
            if (userIsAuthenticated.value) {
                maxChars.value = 10000;
                maxFileSize.value = 'MAX 128GB'
            } else {
                maxChars.value = 250;
                maxFileSize.value = 'Max 25MB'
            }
        };

        setMaxChars();
        watch(userIsAuthenticated, setMaxChars);

        const handleDrop = (event) => {
            event.preventDefault();
            const files = event.dataTransfer.files;
            for (let i = 0; i < files.length; i++) {
                emit('file', { file: files[i], linkLifetime: linkLifetime.value });
            }
        };

        const emitInput = () => {
            emit('input', { text: inputText.value, linkLifetime: linkLifetime.value });
        };

        const clickFileInput = () => {
            fileInput.value.click();
        };

        const handleFileSelect = (event) => {
            const files = event.target.files;
            for (let i = 0; i < files.length; i++) {
                console.log('linkLifetime.value', linkLifetime.value);
                emit('file', { file: files[i], linkLifetime: linkLifetime.value });
            }
        };
        let dragCounter = 0;

        const dragEnter = (event) => {
            event.stopPropagation();
            dragCounter++;
            if (dragCounter === 1) {
                console.log('dragEnter');
                dragOnField.value = true;
            }
        };

        const dragLeave = (event) => {
            event.stopPropagation();
            dragCounter--;
            if (dragCounter === 0) {
                console.log('dragLeave');
                dragOnField.value = false;
            }
        };

        const dragOver = () => {
            dragOnWindow.value = true;
        };

        const dragLeaveWindow = () => {
            dragOnWindow.value = false;
        };



        onMounted(() => {
            const field = document.querySelector('.field');
            field.addEventListener('dragenter', dragEnter);
            field.addEventListener('dragleave', dragLeave);
            window.addEventListener('dragover', dragOver);
            window.addEventListener('dragleave', dragLeaveWindow);
            setMaxChars();
            updatePlaceholderText();
        });

        onBeforeUnmount(() => {
            const field = document.querySelector('.field');
            field.removeEventListener('dragenter', dragEnter);
            field.removeEventListener('dragleave', dragLeave);
            window.removeEventListener('dragover', dragOver);
            window.removeEventListener('dragleave', dragLeaveWindow);
            window.removeEventListener('resize', updatePlaceholderText);
        });

        return {
            inputText,
            handleDrop,
            emitInput,
            clickFileInput,
            fileInput,
            linkLifetime,
            handleFileSelect,
            dragOnField,
            dragOnWindow,
            dragEnter,
            dragLeave,
            userIsAuthenticated,
            maxChars,
            maxFileSize,
            placeholderText,
        };
    }
};
</script>
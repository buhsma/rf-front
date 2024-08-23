<template>
    <nav :class="`${isExpanded && 'expanded'}`">
        <div class="logo">
            <RouterLink v-if="userIsAuthenticated" to="dashboard">
                <img alt="fox" src="@/assets/fox.png" />
            </RouterLink>
            <RouterLink v-else to="/">
                <img alt="fox" src="@/assets/fox.png" />
            </RouterLink>
        </div>
        <div class="navToggleContainer">
            <button class="navToggle" @click="toggleNav">
                <span class="material-icons">keyboard_double_arrow_right</span>
            </button>
        </div>

        <h3>MENU</h3>
        <div class="menu">
            <RouterLink v-if="userIsAuthenticated" class="button" to="dashboard">
                <span class="material-icons material-symbols-outlined">shield_person</span>
                <span class="text">Dashboard</span>
            </RouterLink>
            <RouterLink v-else class="button" to="/">
                <span class="material-icons material-symbols-outlined">shield</span>
                <span class="text">Home</span>
            </RouterLink>
            <!-- <RouterLink v-if="!userIsAuthenticated" class="button" to="register">
                <span class="material-icons">person_add</span>
                <span class="text">+ Create account</span>
            </RouterLink> -->
            <RouterLink v-if="!userIsAuthenticated" class="button" to="login">
                <span class="material-icons material-symbols-outlined">add_moderator</span>
                <span class="text">Login</span>
            </RouterLink>
            <RouterLink class="button" to="about">
                <span class="material-icons material-symbols-outlined">policy</span>
                <span class="text">About</span>
            </RouterLink>
            <RouterLink class="button" to="faq">
                <span class="material-icons material-symbols-outlined">shield_question</span>
                <span class="text">FAQ</span>
            </RouterLink>
            <!-- <RouterLink class="button" to="roadmap">
                <span class="material-icons">timeline</span>
                <span class="text">Roadmap</span>
            </RouterLink> -->
            <button v-if="userIsAuthenticated" class="button" @click="logout">
                <span class="material-icons">logout</span>
                <span class="text">Logout</span>
            </button>
        </div>
        <!-- <RouterLink v-if="userIsAuthenticated" to="dashboard">Dashboard</RouterLink>
        <RouterLink v-else to="/">Home</RouterLink>
        <RouterLink v-if="!userIsAuthenticated" to="register">+ Create account</RouterLink>
        <RouterLink v-if="!userIsAuthenticated" to="login">Login</RouterLink>
        <RouterLink to="about">About</RouterLink>
        <RouterLink to="faq">FAQ</RouterLink>
        <button v-if="userIsAuthenticated" @click="logout">Logout</button> -->
    </nav>
</template>
<style lang="scss" scoped>
nav {
    display: flex;
    flex-direction: column;
    width: calc(2rem + 32px);
    min-height: 100vh;
    // overflow: hidden;
    padding: 1rem;

    background-color: var(--secondary);
    color: var(--primary);

    transition: .2s ease-out;

    .logo {
        margin-bottom: 1rem;

        img {
            width: 2rem;
        }
    }

    .navToggleContainer {
        display: flex;
        justify-content: flex-end;
        margin-bottom: 1rem;
        position: relative;
        top: 0;
        transition: .2s ease-out;

        .navToggle {
            transition: .2s ease-out;

            .material-icons {
                font-size: 2rem;
                color: var(--primary);
                transition: .2s ease-out;
            }

            &:hover {
                .material-icons {
                    color: var(--primary);
                    transform: translateX(.5rem);
                }
            }
        }
    }
.material-symbols-outlined {
    scale: 1.2;
}
    h3,
    .button .text {
        opacity: 0;
        transition: .3s ease-out;
    }

    .menu {
        margin: 0 -1rem;

        .button {
            display: flex;
            align-items: center;
            text-decoration: none;
            padding: .5rem 1rem;
            transition: .2s ease-out;

            .material-icons {
                font-size: 2rem;
                color: var(--primary);
                transition: .2s ease-out;
                margin-bottom: .5rem;
            }

            .text {
                color: var(--primary);
                transition: .2s ease-out;
            }

            &:hover,
            &.router-link-exact-active {
                // background-color: var(--secondary);

                .material-icons,
                .text {
                    color: var(--primary-light);
                }
            }

            &.router-link-exact-active {
                border-right: 5px solid var(--primary);
            }
        }
    }

    &.expanded {
        min-width: var(--nav-width);

        .navToggleContainer {
            top: -3rem;

            .navToggle {
                transform: rotate(-180deg);
            }
        }

        h3,
        .button .text {
            opacity: 1;
            transition: .3s ease-out;
        }

        h3 {
            color: var(--color-text);
            font-size: .875rem;
            margin-bottom: .5rem;
            font-family: "Roboto", sans-serif;
            font-weight: 700;
            font-style: normal;
        }

        .button {
            font-size: 1rem;
            .material-icons {
                margin-right: 1rem;
            }
        }
    }

    @media (max-width: 768px) {
        position: fixed;
        z-index: 100;
    }
}
</style>
<script>

import { useRouter, RouterLink } from 'vue-router'
import { ref, onMounted } from 'vue'
import axios from 'axios'
import authState from '@/tools/authState'



export default {
    components: {
        RouterLink
    },
    setup() {
        const { userIsAuthenticated, setAuthState, checkAuth } = authState();
        const router = useRouter()
        const BACKEND_URL = import.meta.env.VITE_BACKEND_URL;
        const isExpanded = ref(localStorage.getItem('expanded') !== 'false');
        const toggleNav = () => {
            isExpanded.value = !isExpanded.value;
            localStorage.setItem('expanded', isExpanded.value);
        };
        const logout = () => {
            console.log('logout')
            axios.post(`${BACKEND_URL}/api/token/blacklist/`, {
                refresh: localStorage.getItem('refresh')
            })
                .then(function (response) {
                    localStorage.removeItem('access');
                    localStorage.removeItem('refresh'); 
                    setAuthState(false);
                    router.push('/');
                })
                .catch(function (error) {
                    console.error(error)
                });
        };

        onMounted(async () => {
            await checkAuth();
        });
        return {
            logout,
            userIsAuthenticated,
            toggleNav,
            isExpanded
        };

    }
};
</script>
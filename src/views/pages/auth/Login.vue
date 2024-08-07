<script setup>
import { useLayout } from '@/layout/composables/layout';
import { ref, computed, onMounted } from 'vue';
import AppConfig from '@/layout/AppConfig.vue';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
import { useRouter } from 'vue-router'; // Import useRouter

const toast = useToast();
const router = useRouter(); // Get the router instance

const baseURL = import.meta.env.VITE_BASE_URL_SERVICE;

const axiosInstance = axios.create({
    baseURL: baseURL,
    headers: {
        'Content-Type': 'application/json',
    },
});

const { layoutConfig } = useLayout();
const username = ref('');
const password = ref('');
const checked = ref(false);
const session = ref(null);

const logoUrl = computed(() => {
    return `/layout/images/${layoutConfig.darkTheme.value ? 'logo-white' : 'logo-dark'}.svg`;
});

const login = async () => {
    if (!username.value) {
        console.log("username tidak boleh kosong ! ");
        toast.add({ severity: 'error', summary: 'Error', detail: 'Username tidak boleh kosong !', life: 3000 });
        return;
    }
    if (!password.value) {
        console.log("password tidak boleh kosong ! ");
        toast.add({ severity: 'error', summary: 'Error', detail: 'Password tidak boleh kosong ! ', life: 3000 });
        return;
    }

    try {
        const response = await axiosInstance.post(`/login`, {
            username: username.value,
            password: password.value
        });
        console.log("login response: ", response.data);

        // Set session with username and unique code
        const sessionID = Math.random().toString(36).substr(2, 6); // Generate a random 6-character code
        localStorage.setItem('iduser', response.data.data.id);
        localStorage.setItem('username', username.value);
        localStorage.setItem('sessionID', sessionID);

        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
        username.value = '';
        password.value = '';

        getSession(); // Fetch session info after login

        // Redirect to dashboard
        router.push({ name: 'dashboard' });
    } catch (error) {
        console.log("Error login: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: error.response.data.message, life: 3000 });
    }
};

const getSession = () => {
    const id = localStorage.getItem('iduser');
    const username = localStorage.getItem('username');
    const sessionID = localStorage.getItem('sessionID');
    
    if (username && sessionID && id) {
        session.value = { id, username, sessionID };
    } else {
        session.value = null;
    }
};

onMounted(() => {
    getSession(); // Fetch session info on component mount
});
</script>

<template>
    <div class="surface-ground flex align-items-center justify-content-center min-h-screen min-w-screen overflow-hidden">
        <div class="flex flex-column align-items-center justify-content-center">
            <!-- <img :src="logoUrl" alt="Sakai logo" class="mb-5 w-6rem flex-shrink-0" /> -->
            <div style="border-radius: 56px; padding: 0.3rem; background: linear-gradient(180deg, var(--primary-color) 10%, rgba(33, 150, 243, 0) 30%)">
                <div class="w-full surface-card py-8 px-5 sm:px-8" style="border-radius: 53px">
                    <div class="text-center mb-5">
                        <!-- <img src="/demo/images/login/avatar.png" alt="Image" height="50" class="mb-3" /> -->
                        <div class="text-900 text-3xl font-medium mb-3">Welcome, to APlikasi EOQ </div>
                        <span class="text-600 font-medium">Sign in to continue</span>
                    </div>

                    <div>
                        <label for="username" class="block text-900 text-xl font-medium mb-2">Username</label>
                        <InputText id="username" type="text" placeholder="Username" class="w-full md:w-30rem mb-5" style="padding: 1rem" v-model="username" />

                        <label for="password" class="block text-900 font-medium text-xl mb-2">Password</label>
                        <Password id="password" v-model="password" placeholder="Password" :toggleMask="true" class="w-full mb-3" inputClass="w-full" :inputStyle="{ padding: '1rem' }"></Password>

                        <div class="flex align-items-center justify-content-between mb-5 gap-5">
                            <div class="flex align-items-center">
                                <Checkbox v-model="checked" id="rememberme1" binary class="mr-2"></Checkbox>
                                <label for="rememberme1">Remember me</label>
                            </div>
                            <!-- <a class="font-medium no-underline ml-2 text-right cursor-pointer" style="color: var(--primary-color)">Forgot password?</a> -->
                        </div>
                        <Button label="Login" class="w-full p-3 text-xl" @click="login"></Button>
                        <Toast />
                    </div>
                </div>
            </div>
        </div>
    </div>
    <AppConfig simple />
</template>

<style scoped>
.pi-eye {
    transform: scale(1.6);
    margin-right: 1rem;
}

.pi-eye-slash {
    transform: scale(1.6);
    margin-right: 1rem;
}
</style>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';
import { useLayout } from '@/layout/composables/layout';
import { useRouter } from 'vue-router';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';

const toast = useToast();
const baseURL = import.meta.env.VITE_BASE_URL_SERVICE;

const axiosInstance = axios.create({
    baseURL: baseURL,
    headers: {
        'Content-Type': 'application/json',
    },
});

const { layoutConfig, onMenuToggle } = useLayout();

const outsideClickListener = ref(null);
const topbarMenuActive = ref(false);
const router = useRouter();
const userProfileDialog = ref(false);

const id = ref(null);
const nama = ref(null);
const username = ref(null);
const password = ref(null);
const posisi = ref(null);
const hp = ref(null);
const alamat = ref(null);

onMounted(() => {
    bindOutsideClickListener();
    checkSession(); // Check session when component is mounted
});

onBeforeUnmount(() => {
    unbindOutsideClickListener();
});

const logoUrl = computed(() => {
    return `layout/images/${layoutConfig.darkTheme.value ? 'logo-white' : 'logo-dark'}.svg`;
});

const onTopBarMenuButton = () => {
    topbarMenuActive.value = !topbarMenuActive.value;
};
const onSettingsClick = () => {
    topbarMenuActive.value = false;
    router.push('/documentation');
};
const topbarMenuClasses = computed(() => {
    return {
        'layout-topbar-menu-mobile-active': topbarMenuActive.value
    };
});

const bindOutsideClickListener = () => {
    if (!outsideClickListener.value) {
        outsideClickListener.value = (event) => {
            if (isOutsideClicked(event)) {
                topbarMenuActive.value = false;
            }
        };
        document.addEventListener('click', outsideClickListener.value);
    }
};
const unbindOutsideClickListener = () => {
    if (outsideClickListener.value) {
        document.removeEventListener('click', outsideClickListener);
        outsideClickListener.value = null;
    }
};
const isOutsideClicked = (event) => {
    if (!topbarMenuActive.value) return;

    const sidebarEl = document.querySelector('.layout-topbar-menu');
    const topbarEl = document.querySelector('.layout-topbar-menu-button');

    return !(sidebarEl.isSameNode(event.target) || sidebarEl.contains(event.target) || topbarEl.isSameNode(event.target) || topbarEl.contains(event.target));
};

// Function to check session and redirect if not authenticated
const checkSession = () => {
    const username = localStorage.getItem('username');
    const sessionID = localStorage.getItem('sessionID');
    
    if (!username || !sessionID) {
        router.push('/auth/login'); // Redirect to login page if no session
    }
};

const profile = async () => {
    try {
        const response = await axiosInstance.get(`/user?id=${localStorage.getItem('iduser')}`);
        console.log("Response data:", response.data.user);
        id.value = response.data.user.id
        nama.value = response.data.user.nama
        username.value = response.data.user.username
        posisi.value = response.data.user.posisi
        hp.value = response.data.user.hp
        alamat.value = response.data.user.alamat
        userProfileDialog.value = true
    } catch (error) {
        console.error('Error fetching user :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch user', life: 3000 });
    }
}

const logout = () => {
    // Clear session data from localStorage or sessionStorage
    localStorage.removeItem('iduser');
    localStorage.removeItem('username');
    localStorage.removeItem('sessionID');
    // Or if you used sessionStorage:
    // sessionStorage.removeItem('username');
    // sessionStorage.removeItem('sessionID');

    // Redirect to login page
    router.push('/auth/login');
};
</script>

<template>
    <div class="layout-topbar">
        <router-link to="/" class="layout-topbar-logo">
            <img :src="logoUrl" alt="logo" />
            <span>Aplikasi EOQ</span>
        </router-link>

        <button class="p-link layout-menu-button layout-topbar-button" @click="onMenuToggle()">
            <i class="pi pi-bars"></i>
        </button>

        <button class="p-link layout-topbar-menu-button layout-topbar-button" @click="onTopBarMenuButton()">
            <i class="pi pi-ellipsis-v"></i>
        </button>

        <div class="layout-topbar-menu" :class="topbarMenuClasses">
            <!-- <button @click="onTopBarMenuButton()" class="p-link layout-topbar-button">
                <i class="pi pi-calendar"></i>
                <span>Calendar</span>
            </button> -->
            <!-- <button @click="onTopBarMenuButton()" class="p-link layout-topbar-button">
                <i class="pi pi-user"></i>
                <span>Profile</span>
            </button> -->
            <button @click="profile()" class="p-link layout-topbar-button">
                <i class="pi pi-user"></i>
                <span>Profile</span>
            </button>
            <!-- <button @click="onSettingsClick()" class="p-link layout-topbar-button">
                <i class="pi pi-cog"></i>
                <span>Settings</span>
            </button> -->
            <button @click="logout()" class="p-link layout-topbar-button">
                <i class="pi pi-sign-out"></i>
                <span>Logout</span>
            </button>
        </div>
    </div>

    <Dialog v-model:visible="userProfileDialog" :style="{ width: '450px' }" header="Detail Profile" :modal="true" class="p-fluid"  >

<div class="card">
    <div class="field">
        <label for="nama">Nama</label>
        <InputText id="nama" v-model="nama" required="true" autofocus :disabled="true" />
    </div>
    <div class="field">
        <label for="username">Username</label>
        <InputText id="username" v-model="username" required="true" autofocus :disabled="true" />
    </div>
    <div class="field">
        <label for="posisi">Posisi</label>
        <InputText id="posisi" v-model="posisi" required="true" autofocus :disabled="true" />
    </div>
    <div class="field">
        <label for="hp">No HP</label>
        <InputText id="hp" v-model="hp" required="true" autofocus :disabled="true" />
    </div>
    <div class="field">
        <label for="alamat">Alamat</label>
        <InputText id="alamat" v-model="alamat" required="true" autofocus :disabled="true" />
    </div>
    
</div>
</Dialog>
</template>

<style lang="scss" scoped></style>

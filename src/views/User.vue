<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
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

const listUser = ref(null);
const user = ref({});
const selectUser = ref(null);
const loadData = ref(false);
const id = ref(null);
const nama = ref(null);
const username = ref(null);
const password = ref(null);
const posisi = ref(null);
const hp = ref(null);
const alamat = ref(null);

const isAdmin = ref(false);

const createUserDialog = ref(false);
const updateUserDialog = ref(false);
const deleteUserDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    const userAdmin = localStorage.getItem('username');
    if (userAdmin === 'admin') {
        isAdmin.value = true;
    }
    getUsers();
});

/* Api get Mapping Users */
const getUsers = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/user/all`);
        listUser.value = response.data.user;
        console.log("Response data:", response.data.user);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching user :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch users', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeleteUser = (data) => {
    user.value = data;
    deleteUserDialog.value = true;
};

const deleteUser = async () => {
    console.log("id user: ", user.value.id);

    try {
        const response = await axiosInstance.delete(`/user?id=${user.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deleteUserDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + user.value.nama, life: 3000 });
        getUsers();
    } catch (error) {
        console.log("Error Delete User : ", error);
        deleteUserDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete user ' + user.value.nama, life: 3000 });
    }
};

const openNewUser = () => {
    user.value = {};
    createUserDialog.value = true;
};

const createUser = async () => {
    try {
        const response = await axiosInstance.post(`/user`, {
            nama: nama.value,
            username: username.value,
            password: password.value,
            posisi: posisi.value,
            hp: hp.value,
            alamat: alamat.value
        });
        console.log("User created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
        nama.value = '';
        username.value = '';
        password.value = '';
        posisi.value = '';
        hp.value = '';
        alamat.value = '';

        createUserDialog.value = false;
        getUsers();
    } catch (error) {
        console.log("Error creating user: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create user', life: 3000 });
        createUserDialog.value = false;
    }
};

const editUser = (editUser) => {
    user.value = [{...editUser}];
    updateUserDialog.value = true;

    // Populate form fields with existing user data
    id.value = user.value[0].id;
    nama.value = user.value[0].nama;
    username.value = user.value[0].username;
    posisi.value = user.value[0].posisi;
    hp.value = user.value[0].hp;
    alamat.value = user.value[0].alamat;
};

const saveUser = async () => {
    const isDataChanged =
        nama.value !== user.value[0].nama ||
        username.value !== user.value[0].username ||
        posisi.value !== user.value[0].posisi ||
        hp.value !== user.value[0].hp ||
        alamat.value !== user.value[0].alamat;

    if (isDataChanged) {
        try {
            const response = await axiosInstance.put(`/user`, {
                id: id.value,
                nama: nama.value,
                username: username.value,
                posisi: posisi.value,
                hp: hp.value,
                alamat: alamat.value
            });
            console.log("User updated: ", response.data);
            toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
            updateUserDialog.value = false;
            getUsers();
        } catch (error) {
            console.log("Error updating user: ", error);
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to update user', life: 3000 });
            updateUserDialog.value = false;
        }
    } else {
        updateUserDialog.value = false;
    }
};


const hideDialogEdit = () => {
    updateUserDialog.value = false;
};

const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS }
    };
};
</script>


<template>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                 <!-- Conditionally render h5 and toolbar -->
                 <template v-if="isAdmin">
                    <h5 class="font-bold">User</h5>
                    <Toolbar class="mb-4">
                        <template v-slot:start>
                            <div class="my-2">
                                <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewUser" />
                            </div>
                        </template>
                    </Toolbar>
                </template>

                <DataTable
                    ref="dt"
                    :value="listUser"
                    v-model:selection="selectUser"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Users"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage User</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <!-- <Column selectionMode="multiple" headerStyle="width: 10rem"></Column> -->
                    <Column field="nama" header="Nama" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nama }}
                        </template>
                    </Column>
                    <Column field="username" header="Username" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.username }}
                        </template>
                    </Column>
                    <Column field="posisi" header="Posisi" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.posisi }}
                        </template>
                    </Column>
                    <Column field="hp" header="No HP" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.hp }}
                        </template>
                    </Column>
                    <Column field="alamat" header="Alamat" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.alamat }}
                        </template>
                    </Column>
                    <!-- Conditionally render edit and delete columns -->
                    <template v-if="isAdmin">
                        <Column headerStyle="min-width:10rem;">
                            <template #body="slotProps">
                                <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editUser(slotProps.data)" />
                                <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeleteUser(slotProps.data)" />
                            </template>
                        </Column>
                    </template>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data users</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deleteUserDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="user"
                            >Are you sure you want to delete user <b>{{ user.nama }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deleteUserDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deleteUser" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createUserDialog" :style="{ width: '450px' }" header="Create New User" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="description">Nama</label>
                        <InputText id="nama" type="text" required="true" v-model="nama" placeholder="Nama User" />
                    </div>
                    <div class="field">
                        <label for="description">Username</label>
                        <InputText id="username" type="text" required="true" v-model="username" placeholder="Username" />
                    </div>
                    <div class="field">
                        <label for="description">Password</label>
                        <InputText id="password" type="text" required="true" v-model="password" placeholder="password" />
                    </div>
                    <div class="field">
                        <label for="description">Posisi</label>
                        <InputText id="posisi" type="text" required="true" v-model="posisi" placeholder="Posisi" />
                    </div>
                    <div class="field">
                        <label for="description">No HP</label>
                        <InputText id="hp" type="text" required="true" v-model="hp"  placeholder="Hp"/>
                    </div>
                    <div class="field">
                        <label for="description">Alamat</label>
                        <InputText id="alamat" type="text" required="true" v-model="alamat" placeholder="Alamat" />
                    </div>
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createUser" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createUser" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="updateUserDialog" :style="{ width: '450px' }" header="Update User" :modal="true" class="p-fluid"  >

                    <div class="card">
                        <div class="field">
                            <label for="nama">Nama</label>
                            <InputText id="nama" v-model="nama" required="true" autofocus  />
                        </div>
                        <div class="field">
                            <label for="username">Username</label>
                            <InputText id="username" v-model="username" required="true" autofocus  />
                        </div>
                        <div class="field">
                            <label for="posisi">Posisi</label>
                            <InputText id="posisi" v-model="posisi" required="true" autofocus  />
                        </div>
                        <div class="field">
                            <label for="hp">No HP</label>
                            <InputText id="hp" v-model="hp" required="true" autofocus  />
                        </div>
                        <div class="field">
                            <label for="alamat">Alamat</label>
                            <InputText id="alamat" v-model="alamat" required="true" autofocus  />
                        </div>
                        <div class="formgrid grid">
                            <div class="field col">
                                <Button label="Cancel" icon="pi pi-times" class="p-button-outlined p-button-danger mr-2 mb-2" @click="hideDialogEdit" />
                            </div>
                            <div class="field col">
                                <Button label="Update" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="saveUser" />
                            </div>
                        </div>
                    </div>
                </Dialog>
           </div>
        </div>
    </div>
</template>

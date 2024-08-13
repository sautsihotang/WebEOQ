<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
import moment from 'moment';

const toast = useToast();

const baseURL = import.meta.env.VITE_BASE_URL_SERVICE;

const axiosInstance = axios.create({
    baseURL: baseURL,
    headers: {
        'Content-Type': 'application/json',
    },
});

const listPenyimpanan = ref(null);
const penyimpanan = ref({});
const selectPenyimpanan = ref(null);
const loadData = ref(false);
const id = ref(null);
const jenis = ref(null);
const biaya_penyimpanan = ref(null);
const tanggal_penyimpanan = ref(null);

const createSimpanDialog = ref(false);
const updateSimpanDialog = ref(false);
const deleteSimpanDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getSimpans();
});

/* Api get Mapping Simpan */
const getSimpans = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/penyimpanan/all`);
        listPenyimpanan.value = response.data.penyimpanan;
        console.log("Response data:", response.data.penyimpanan);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching penyimpanan :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch penyimpanan', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeleteSimpan = (data) => {
    penyimpanan.value = data;
    deleteSimpanDialog.value = true;
};

const deleteSimpan = async () => {
    console.log("id simpan: ", penyimpanan.value.id);

    try {
        const response = await axiosInstance.delete(`/penyimpanan?id=${penyimpanan.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deleteSimpanDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + penyimpanan.value.nama, life: 3000 });
        getSimpans();
    } catch (error) {
        console.log("Error Delete Penyimpanan: ", error);
        deleteSimpanDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete penyimpanan ' + penyimpanan.value.nama, life: 3000 });
    }
};

const openNewSimpan = () => {
    penyimpanan.value = {};
    createSimpanDialog.value = true;
};

const createSimpan = async () => {
    try {
        const response = await axiosInstance.post(`/penyimpanan`, {
            jenis: jenis.value,
            biaya_penyimpanan: biaya_penyimpanan.value,
            tanggal_penyimpanan: tanggal_penyimpanan.value
        });
        console.log("Penyimpanan created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
        jenis.value = '';
        biaya_penyimpanan.value = '';
        tanggal_penyimpanan.value = '';

        createSimpanDialog.value = false;
        getSimpans();
    } catch (error) {
        console.log("Error creating penyimpanan: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create penyimpanan', life: 3000 });
        createSimpanDialog.value = false;
    }
};

const editSimpan = (editSimpan) => {
    penyimpanan.value = [{...editSimpan}];
    updateSimpanDialog.value = true;

    // Populate form fields with existing penyimpanan data
    id.value = penyimpanan.value[0].id;
    jenis.value = penyimpanan.value[0].jenis;
    biaya_penyimpanan.value = penyimpanan.value[0].biaya_penyimpanan;
    tanggal_penyimpanan.value = penyimpanan.value[0].tanggal_penyimpanan;
};

const saveSimpan = async () => {
    const isDataChanged =
        jenis.value !== penyimpanan.value[0].jenis ||
        biaya_penyimpanan.value !== penyimpanan.value[0].biaya_penyimpanan ||
        tanggal_penyimpanan.value !== penyimpanan.value[0].tanggal_penyimpanan;

    if (isDataChanged) {
        try {
            const response = await axiosInstance.put(`/penyimpanan`, {
                id: id.value,
                jenis: jenis.value,
                biaya_penyimpanan: biaya_penyimpanan.value,
                tanggal_penyimpanan: tanggal_penyimpanan.value
            });
            console.log("Penyimpanan updated: ", response.data);
            toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
            updateSimpanDialog.value = false;
            getSimpans();
        } catch (error) {
            console.log("Error updating penyimpanan: ", error);
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to update penyimpanan', life: 3000 });
            updateSimpanDialog.value = false;
        }
    } else {
        updateSimpanDialog.value = false;
    }
};

const formatDate = (dateString) => {
      const date = moment(dateString);
      return date.format('DD-MM-YYYY HH:mm'); // Atur format sesuai kebutuhan
    };

const hideDialogEdit = () => {
    updateSimpanDialog.value = false;
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
                <h5 class="font-bold">Biaya Penyimpanan Barang</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewSimpan" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listPenyimpanan"
                    v-model:selection="selectPenyimpanan"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} penyimpanan"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Biaya Penyimpanan</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 10rem"></Column>
                    <Column field="jenis" header="Jenis Biaya" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.jenis }}
                        </template>
                    </Column>
                    <Column field="biaya_penyimpanan" header="Biaya Penyimpanan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.biaya_penyimpanan }}
                        </template>
                    </Column>
                    <Column field="tanggal_penyimpanan" header="Tanggal Penyimpanan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ formatDate(slotProps.data.tanggal_penyimpanan) }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editSimpan(slotProps.data)" />
                            <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeleteSimpan(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data biaya penyimpanan</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deleteSimpanDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="penyimpanan"
                            >Are you sure you want to delete penyimpanan <b>{{ penyimpanan.jenis }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deleteSimpanDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deleteSimpan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createSimpanDialog" :style="{ width: '450px' }" header="Create New Penyimpanan" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="description">Jenis Biaya</label>
                        <InputText id="jenis" type="text" required="true" v-model="jenis" placeholder="Jenis Biaya" />
                    </div>
                    <div class="field">
                        <label for="description">Biaya Penyimpanan (Rp.)</label>
                        <InputText id="biaya" type="number" required="true" v-model="biaya_penyimpanan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal">Tanggal</label>
                        <Calendar inputId="tanggal" v-model="tanggal_penyimpanan"></Calendar>
                    </div>
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createSimpan" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createSimpan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="updateSimpanDialog" :style="{ width: '450px' }" header="Update Biaya Penyimpanan" :modal="true" class="p-fluid"  >

                    <div class="card">
                        <div class="field">
                        <label for="description">Jenis Biaya</label>
                        <InputText id="jenis" type="text" required="true" v-model="jenis" placeholder="Jenis Biaya" />
                    </div>
                    <div class="field">
                        <label for="description">Biaya Penyimpanan (Rp.)</label>
                        <InputText id="biaya" type="number" required="true" v-model="biaya_penyimpanan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal">Tanggal</label>
                        <Calendar inputId="tanggal" v-model="tanggal_penyimpanan"></Calendar>
                    </div>
                        <div class="formgrid grid">
                            <div class="field col">
                                <Button label="Cancel" icon="pi pi-times" class="p-button-outlined p-button-danger mr-2 mb-2" @click="hideDialogEdit" />
                            </div>
                            <div class="field col">
                                <Button label="Update" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="saveSimpan" />
                            </div>
                        </div>
                    </div>
                </Dialog>
           </div>
        </div>
    </div>
</template>

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

const listPenjualan = ref(null);
const penjualan = ref({});
const selectPenjualan = ref(null);
const loadData = ref(false);
const id = ref(null);
const id_user = ref(null);
const id_barang = ref(null);
const kuantitas = ref(0);
const harga_satuan = ref(0);
const total_harga = ref(0);
const tanggal_penjualan = ref(null);

const listBarang = ref([]);
const barang = ref(null);
const barangIdEdit = ref(null);

const autoFilteredValueBarangs = ref([]);

const createPenjualanDialog = ref(false);
const updatePenjualanDialog = ref(false);
const deletePenjualanDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getPenjualans();
});

/* Api get Mapping Penjualan */
const getPenjualans = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/penjualan/all`);
        listPenjualan.value = response.data.penjualan;
        console.log("Response data:", response.data.penjualan);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching penjualan :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch penjualan', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeletePenjualan = (data) => {
    penjualan.value = data;
    deletePenjualanDialog.value = true;
};

const deletePenjualan = async () => {
    console.log("id penjualan: ", penjualan.value.id);

    try {
        const response = await axiosInstance.delete(`/penjualan?id=${penjualan.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deletePenjualanDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + penjualan.value.nama, life: 3000 });
        getPenjualans();
    } catch (error) {
        console.log("Error Delete Penjualan: ", error);
        deletePenjualanDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete penjualan ' + penjualan.value.nama, life: 3000 });
    }
};

const openNewPenjualan = () => {
    getBarangs();
    penjualan.value = {};
    createPenjualanDialog.value = true;
};

const createPenjualan = async () => {
    try {
        const response = await axiosInstance.post(`/penjualan`, {
            id_user : id_user.value,
            id_barang : barang.value.id,
            kuantitas : kuantitas.value,
            harga_satuan : harga_satuan.value,
            tanggal_penjualan : tanggal_penjualan.value
        });
        console.log("Penjualan created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
         // Reset values
         id_user.value = '';
        id_barang.value = '';
        kuantitas.value = 0;
        harga_satuan.value = 0;
        tanggal_penjualan.value = null;

        createPenjualanDialog.value = false;
        getPenjualans();
    } catch (error) {
        console.log("Error creating penjualan: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create penjualan', life: 3000 });
        createPenjualanDialog.value = false;
    }
};

const editPenjualan = (editJual) => {
    penjualan.value = [{...editJual}];
    updatePenjualanDialog.value = true;

    getBarangs();
    
    // Populate form fields with existing penjualan data
    id.value = penjualan.value[0].id;
    id_user.value = penjualan.value[0].id_user;
    kuantitas.value = penjualan.value[0].kuantitas;
    harga_satuan.value = penjualan.value[0].harga_satuan;
    total_harga.value = penjualan.value[0].total_harga;
    tanggal_penjualan.value = penjualan.value[0].tanggal_penjualan;
    barangIdEdit.value = penjualan.value[0].id_barang;
};

const savePenjualan = async () => {
    const isDataChanged =
        id_user.value !== penjualan.value[0].id_user ||
        // id_barang.value !== penjualan.value[0].id_barang ||
        barangIdEdit.value !== barang.value.id || 
        kuantitas.value !== penjualan.value[0].kuantitas ||
        harga_satuan.value !== penjualan.value[0].harga_satuan ||
        total_harga.value !== penjualan.value[0].total_harga ||
        tanggal_penjualan.value !== penjualan.value[0].tanggal_penjualan;

    if (isDataChanged) {
        try {
            const response = await axiosInstance.put(`/penjualan`, {
                id: id.value,
                id_user: id_user.value,
                id_barang: barang.value.id,
                kuantitas: kuantitas.value,
                harga_satuan: harga_satuan.value,
                total_harga: total_harga.value,
                tanggal_penjualan: tanggal_penjualan.value
            });
            console.log("penjualan updated: ", response.data);
            toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
            updatePenjualanDialog.value = false;
            getPenjualans();
        } catch (error) {
            console.log("Error updating penjualan: ", error);
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to update penjualan', life: 3000 });
            updatePenjualanDialog.value = false;
        }
    } else {
        updatePenjualanDialog.value = false;
    }
};

/* Api get Mapping Barang */
const getBarangs = async () => {
    try {
        const response = await axiosInstance.get(`/barangs`);
        listBarang.value = response.data.barang;
        console.log("Response data bahan baku:", response.data.barang);
    } catch (error) {
        console.error('Error fetching bahan baku:', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch list bahan baku', life: 3000 });
    }
};

/* Function search or autocomplete barangs */
const searchBarangs = (event) => {
    console.log("suppliers value :", listBarang.value);
    setTimeout(() => {
        if (!event.query.trim().length) {
            autoFilteredValueBarangs.value = [...listBarang.value];
            console.log("Auto filter suppliers : ", autoFilteredValueBarangs.value);
        } else {
            autoFilteredValueBarangs.value = listBarang.value.filter((barang) => {
                return barang.nama.toLowerCase().startsWith(event.query.toLowerCase());
            });
        }
    }, 25)
};

const formatDate = (dateString) => {
      const date = moment(dateString);
      return date.format('DD-MM-YYYY HH:mm'); // Atur format sesuai kebutuhan
    };

const hideDialogEdit = () => {
    updatePenjualanDialog.value = false;
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
                <h5 class="font-bold">Data Penjualan Bahan Baku</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewPenjualan" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listPenjualan"
                    v-model:selection="selectPenjualan"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Penjualan"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Pemakaian</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 10rem"></Column>
                    <!-- <Column field="id_user" header="ID User" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.id_user }}
                        </template>
                    </Column> -->
                    <Column field="barang_nama" header="Nama Bahan Baku" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.barang_nama }}
                        </template>
                    </Column>
                    <Column field="kuantitas" header="Kuantitas" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.kuantitas }}
                        </template>
                    </Column>
                    <Column field="harga_satuan" header="Harga Satuan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.harga_satuan }}
                        </template>
                    </Column>
                    <Column field="total_harga" header="Total Harga" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.total_harga }}
                        </template>
                    </Column>
                    <Column field="tanggal_penjualan" header="Tanggal Pemakaian" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ formatDate(slotProps.data.tanggal_penjualan) }}
                        </template>
                    </Column>
                    <Column field="supplier_nama" header="Perusahaan Supplier Bahan Baku" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.supplier_nama }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;" header="Action">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editPenjualan(slotProps.data)" />
                            <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeletePenjualan(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data penjualan</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deletePenjualanDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="penjualan"
                            >Are you sure you want to delete pemakaian <b>{{ penjualan.id }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deletePenjualanDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deletePenjualan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createPenjualanDialog" :style="{ width: '900px' }" header="Create New Penjualan" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="supplier">Pilih Bahan Baku</label>
                        <AutoComplete placeholder="Search bahan baku..." id="dd" :dropdown="true" v-model="barang" :suggestions="autoFilteredValueBarangs" @complete="searchBarangs($event)" field="nama_barang" />
                    </div>
                    <div class="field">
                        <label for="kuantitas">Kuantitas (qty)</label>
                        <InputText id="kuantitas" type="number" required="true" v-model="kuantitas" placeholder="jumlah bahan baku" />
                    </div>
                    <div class="field">
                        <label for="harga_satuan">Harga Satuan  (Rp.)</label>
                        <InputText id="harga_satuan" type="number" required="true" v-model="harga_satuan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal_penjualan">Tanggal</label>
                        <Calendar inputId="tanggal_penjualan" v-model="tanggal_penjualan"></Calendar>
                    </div>
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createpenjualan" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createPenjualan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="updatePenjualanDialog" :style="{ width: '450px' }" header="Update Penjualan" :modal="true" class="p-fluid"  >
                    <div class="field">
                        <label for="supplier">Pilih Bahan Baku</label>
                        <AutoComplete placeholder="Search bahan baku..." id="dd" :dropdown="true" v-model="barang" :suggestions="autoFilteredValueBarangs" @complete="searchBarangs($event)" field="nama_barang" />
                    </div>
                    <div class="field">
                        <label for="kuantitas">Kuantitas (qty)</label>
                        <InputText id="kuantitas" type="number" required="true" v-model="kuantitas" placeholder="jumlah bahan baku" />
                    </div>
                    <div class="field">
                        <label for="harga_satuan">Harga Satuan  (Rp.)</label>
                        <InputText id="harga_satuan" type="number" required="true" v-model="harga_satuan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal_penjualan">Tanggal</label>
                        <Calendar inputId="tanggal_penjualan" v-model="tanggal_penjualan"></Calendar>
                    </div>
                    <div class="card">
                        <div class="formgrid grid">
                            <div class="field col">
                                <Button label="Cancel" icon="pi pi-times" class="p-button-outlined p-button-danger mr-2 mb-2" @click="hideDialogEdit" />
                            </div>
                            <div class="field col">
                                <Button label="Update" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="savePenjualan" />
                            </div>
                        </div>
                    </div>
                </Dialog>
           </div>
        </div>
    </div>
</template>

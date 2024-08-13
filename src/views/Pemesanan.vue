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

const listPemesanan = ref(null);
const pemesanan = ref({});
const selectPemesanan = ref(null);
const loadData = ref(false);
const id = ref(null);
const id_user = ref(null);
const id_barang = ref(null);
const kuantitas = ref(0);
const harga_satuan = ref(0);
const total_harga = ref(0);
const biaya_telepon = ref(0);
const biaya_adm = ref(0);
const biaya_transportasi = ref(0);
const total_biaya_pemesanan = ref(0);
const tanggal_pemesanan = ref(null);

const listBarang = ref([]);
const barang = ref(null);
const barangIdEdit = ref(null);

const autoFilteredValueBarangs = ref([]);

const createPemesananDialog = ref(false);
const updatePemesananDialog = ref(false);
const deletePemesananDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getPemesanans();
});

/* Api get Mapping Pemesanan */
const getPemesanans = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/pemesanan/all`);
        listPemesanan.value = response.data.pemesanan;
        console.log("Response data:", response.data.pemesanan);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching pemesanan :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch pemesanan', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeletePemesanan = (data) => {
    pemesanan.value = data;
    deletePemesananDialog.value = true;
};

const deletePemesanan = async () => {
    console.log("id pemesanan: ", pemesanan.value.id);

    try {
        const response = await axiosInstance.delete(`/pemesanan?id=${pemesanan.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deletePemesananDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + pemesanan.value.nama, life: 3000 });
        getPemesanans();
    } catch (error) {
        console.log("Error Delete Pemesanan: ", error);
        deletePemesananDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete pemesanan ' + pemesanan.value.nama, life: 3000 });
    }
};

const openNewPemesanan = () => {
    getBarangs();
    pemesanan.value = {};
    createPemesananDialog.value = true;
};

const createPemesanan = async () => {
    try {
        const response = await axiosInstance.post(`/pemesanan`, {
            id_user : id_user.value,
            id_barang : barang.value.id,
            kuantitas : kuantitas.value,
            harga_satuan : harga_satuan.value,
            biaya_telepon : biaya_telepon.value,
            biaya_adm : biaya_adm.value,
            biaya_transportasi : biaya_transportasi.value,
            tanggal_pemesanan : tanggal_pemesanan.value
        });
        console.log("Pemesanan created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
         // Reset values
         id_user.value = null;
        id_barang.value = null;
        kuantitas.value = 0;
        harga_satuan.value = 0;
        biaya_telepon.value = 0;
        biaya_adm.value = 0;
        biaya_transportasi.value = 0;
        tanggal_pemesanan.value = null;
        barang.value = null;

        createPemesananDialog.value = false;
        getPemesanans();
    } catch (error) {
        console.log("Error creating pemesanan: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create Pemesanan', life: 3000 });
        createPemesananDialog.value = false;
    }
};

const editPemesanan = (editPesan) => {
    pemesanan.value = [{...editPesan}];
    updatePemesananDialog.value = true;

    getBarangs();
    
    // Populate form fields with existing pemesanan data
    id.value = pemesanan.value[0].id;
    id_user.value = pemesanan.value[0].id_user;
    kuantitas.value = pemesanan.value[0].kuantitas;
    harga_satuan.value = pemesanan.value[0].harga_satuan;
    total_harga.value = pemesanan.value[0].total_harga;
    biaya_telepon.value = pemesanan.value[0].biaya_telepon;
    biaya_adm.value = pemesanan.value[0].biaya_adm;
    biaya_transportasi.value = pemesanan.value[0].biaya_transportasi;
    total_biaya_pemesanan.value = pemesanan.value[0].total_biaya_pemesanan;
    tanggal_pemesanan.value = pemesanan.value[0].tanggal_pemesanan;
    barangIdEdit.value = pemesanan.value[0].id_barang;
};

const savePemesanan = async () => {
    const isDataChanged =
        id_user.value !== pemesanan.value[0].id_user ||
        // id_barang.value !== pemesanan.value[0].id_barang ||
        barangIdEdit.value !== barang.value.id || 
        kuantitas.value !== pemesanan.value[0].kuantitas ||
        harga_satuan.value !== pemesanan.value[0].harga_satuan ||
        total_harga.value !== pemesanan.value[0].total_harga ||
        biaya_telepon.value !== pemesanan.value[0].biaya_telepon ||
        biaya_adm.value !== pemesanan.value[0].biaya_adm ||
        biaya_transportasi.value !== pemesanan.value[0].biaya_transportasi ||
        total_biaya_pemesanan.value !== pemesanan.value[0].total_biaya_pemesanan ||
        tanggal_pemesanan.value !== pemesanan.value[0].tanggal_pemesanan;

    if (isDataChanged) {
        try {
            const response = await axiosInstance.put(`/pemesanan`, {
                id: id.value,
                id_user: id_user.value,
                id_barang: barang.value.id,
                kuantitas: kuantitas.value,
                harga_satuan: harga_satuan.value,
                total_harga: total_harga.value,
                biaya_telepon: biaya_telepon.value,
                biaya_adm: biaya_adm.value,
                biaya_transportasi: biaya_transportasi.value,
                total_biaya_pemesanan: total_biaya_pemesanan.value,
                tanggal_pemesanan: tanggal_pemesanan.value
            });
            console.log("Pemesanan updated: ", response.data);
            toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
            updatePemesananDialog.value = false;
            getPemesanans();
        } catch (error) {
            console.log("Error updating pemesanan: ", error);
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to update pemesanan', life: 3000 });
            updatePemesananDialog.value = false;
        }
    } else {
        updatePemesananDialog.value = false;
    }
};

/* Api get Mapping Barang */
const getBarangs = async () => {
    try {
        const response = await axiosInstance.get(`/barangs`);
        listBarang.value = response.data.barang;
        console.log("Response data barang:", response.data.barang);
    } catch (error) {
        console.error('Error fetching barang:', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch list barang', life: 3000 });
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
    updatePemesananDialog.value = false;
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
                <h5 class="font-bold">Data Pemesanan Barang</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewPemesanan" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listPemesanan"
                    v-model:selection="selectPemesanan"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Pemesanan"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Pemesanan</h5>
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
                    <Column field="barang_nama" header="Nama Barang" :sortable="true" headerStyle="width:15%; min-width:10rem;">
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
                    <Column field="biaya_telepon" header="Biaya Telepon" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.biaya_telepon }}
                        </template>
                    </Column>
                    <Column field="biaya_adm" header="Biaya Adm" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.biaya_adm }}
                        </template>
                    </Column>
                    <Column field="biaya_transportasi" header="Biaya Transportasi" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.biaya_transportasi }}
                        </template>
                    </Column>
                    <Column field="total_biaya_pemesanan" header="Total Biaya Pemesanan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ "Rp."+slotProps.data.total_biaya_pemesanan }}
                        </template>
                    </Column>
                    <Column field="tanggal_pemesanan" header="Tanggal Pemesanan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ formatDate(slotProps.data.tanggal_pemesanan) }}
                        </template>
                    </Column>
                    <Column field="supplier_nama" header="Supplier Barang" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.supplier_nama }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;" header="Action">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editPemesanan(slotProps.data)" />
                            <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeletePemesanan(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data pemesanan</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deletePemesananDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="pemesanan"
                            >Are you sure you want to delete pemesanan <b>{{ pemesanan.id }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deletePemesananDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deletePemesanan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createPemesananDialog" :style="{ width: '900px' }" header="Create New Pemesanan" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="supplier">Pilih Barang</label>
                        <AutoComplete placeholder="Search barang..." id="dd" :dropdown="true" v-model="barang" :suggestions="autoFilteredValueBarangs" @complete="searchBarangs($event)" field="nama_barang" />
                    </div>
                    <div class="field">
                        <label for="kuantitas">Kuantitas (qty)</label>
                        <InputText id="kuantitas" type="number" required="true" v-model="kuantitas" placeholder="jumlah barang" />
                    </div>
                    <div class="field">
                        <label for="harga_satuan">Harga Satuan  (Rp.)</label>
                        <InputText id="harga_satuan" type="number" required="true" v-model="harga_satuan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_telepon">Biaya Telepon  (Rp.)</label>
                        <InputText id="biaya_telepon" type="number" required="true" v-model="biaya_telepon" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_adm">Biaya Adm (Rp.)</label>
                        <InputText id="biaya_adm" type="number" required="true" v-model="biaya_adm" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_transportasi">Biaya Transportasi (Rp.)</label>
                        <InputText id="biaya_transportasi" type="number" required="true" v-model="biaya_transportasi" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal_pemesanan">Tanggal</label>
                        <Calendar inputId="tanggal_pemesanan" v-model="tanggal_pemesanan"></Calendar>
                    </div>
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createPemesanan" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createPemesanan" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="updatePemesananDialog" :style="{ width: '450px' }" header="Update Pemesanan" :modal="true" class="p-fluid"  >
                    <div class="field">
                        <label for="supplier">Pilih Barang</label>
                        <AutoComplete placeholder="Search barang..." id="dd" :dropdown="true" v-model="barang" :suggestions="autoFilteredValueBarangs" @complete="searchBarangs($event)" field="nama_barang" />
                    </div>
                    <div class="field">
                        <label for="kuantitas">Kuantitas (qty)</label>
                        <InputText id="kuantitas" type="number" required="true" v-model="kuantitas" placeholder="jumlah barang" />
                    </div>
                    <div class="field">
                        <label for="harga_satuan">Harga Satuan  (Rp.)</label>
                        <InputText id="harga_satuan" type="number" required="true" v-model="harga_satuan" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_telepon">Biaya Telepon  (Rp.)</label>
                        <InputText id="biaya_telepon" type="number" required="true" v-model="biaya_telepon" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_adm">Biaya Adm (Rp.)</label>
                        <InputText id="biaya_adm" type="number" required="true" v-model="biaya_adm" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="biaya_transportasi">Biaya Transportasi (Rp.)</label>
                        <InputText id="biaya_transportasi" type="number" required="true" v-model="biaya_transportasi" placeholder="Masukkan dalam Rupiah" />
                    </div>
                    <div class="field">
                        <label for="tanggal_pemesanan">Tanggal</label>
                        <Calendar inputId="tanggal_pemesanan" v-model="tanggal_pemesanan"></Calendar>
                    </div>
                    <div class="card">
                        <div class="formgrid grid">
                            <div class="field col">
                                <Button label="Cancel" icon="pi pi-times" class="p-button-outlined p-button-danger mr-2 mb-2" @click="hideDialogEdit" />
                            </div>
                            <div class="field col">
                                <Button label="Update" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="savePemesanan" />
                            </div>
                        </div>
                    </div>
                </Dialog>
           </div>
        </div>
    </div>
</template>

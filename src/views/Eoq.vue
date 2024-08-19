<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
import moment from 'moment';
import { utils, writeFile } from 'xlsx';

const toast = useToast();

const baseURL = import.meta.env.VITE_BASE_URL_SERVICE;

const axiosInstance = axios.create({
    baseURL: baseURL,
    headers: {
        'Content-Type': 'application/json',
    },
});

const listEoq = ref(null);
const eoq = ref({});
const selectEoq = ref(null);
const loadData = ref(false);
const id = ref(null);
const id_barang = ref(null);
const nilaiEoq = ref(0);
const periode = ref(null);
const tanggal_perhitungan = ref(null);

const listBarang = ref([]);
const barang = ref(null);
const barangIdEdit = ref(null);

const autoFilteredValueBarangs = ref([]);

const createEoqDialog = ref(false);
const updateEoqDialog = ref(false);
const deleteEoqDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getEoqs();
});

/* Api get Mapping Eoq */
const getEoqs = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/eoq/all`);
        listEoq.value = response.data.eoq;
        console.log("Response data:", response.data.eoq);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching eoq :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch eoq', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeleteEoq = (data) => {
    eoq.value = data;
    deleteEoqDialog.value = true;
};

const deleteEoq = async () => {
    console.log("id eoq: ", eoq.value.id);

    try {
        const response = await axiosInstance.delete(`/eoq?id=${eoq.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deleteEoqDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + eoq.value.nama_barang, life: 3000 });
        getEoqs();
    } catch (error) {
        console.log("Error Delete Eoq: ", error);
        deleteEoqDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete eoq ' + eoq.value.nama, life: 3000 });
    }
};

const openNewEoq = () => {
    getBarangs();
    eoq.value = {};
    createEoqDialog.value = true;
};

const createEoq = async () => {
    try {
        const response = await axiosInstance.post(`/eoq`, {
            id_barang : barang.value.id,
            periode : periode.value,
        });
        console.log("Eoq created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
         // Reset values
        id_barang.value = '';
        periode.value = '';

        createEoqDialog.value = false;
        getEoqs();
    } catch (error) {
        console.log("Error creating eoq: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create eoq', life: 3000 });
        createEoqDialog.value = false;
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
    updatePenjualanDialog.value = false;
};

const exportData = () => {
    if (!listEoq.value || listEoq.value.length === 0) {
        toast.add({ severity: 'error', summary: 'Error', detail: 'No data available to export', life: 3000 });
        return;
    }

    const formattedData = listEoq.value.map(item => ({
        'Nama Barang': item.nama_barang,
        'Kebutuhan Bahan Baku (D)': item.d,
        'Biaya pemesanan sekali pesan (S)': item.s,
        'Biaya penyimpanan per unit (H)': item.h,
        'Nilai Eoq': item.nilai_eoq,
        'Periode': item.periode,
        'Tanggal Perhitungan': formatDate(item.tanggal_perhitungan),
    }));

    const ws = utils.json_to_sheet(formattedData);
    const wb = utils.book_new();
    utils.book_append_sheet(wb, ws, 'Eoq Data');

    const currentDate = moment().format('YYYY-MM-DD'); // Format current date
    writeFile(wb, `Eoq_data_${currentDate}.xlsx`);
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
                <h5 class="font-bold">Data Perhitungan Eoq</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewEoq" />
                        </div>
                    </template>
                    <template v-slot:end>
                        <div class="my-2">
                            <Button label="Export " icon="pi pi-file-export" class="mr-2" severity="warning" @click="exportData" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listEoq"
                    v-model:selection="selectEoq"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Eoq"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Perhitungan Eoq</h5>
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
                    <Column field="nama_barang" header="Nama Bahan Baku" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nama_barang }}
                        </template>
                    </Column>
                    <Column field="d" header="Kebutuhan Bahan Baku (D)" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.d }}
                        </template>
                    </Column>
                    <Column field="s" header="Biaya pemesanan sekali pesan (S)" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.s }}
                        </template>
                    </Column>
                    <Column field="h" header="Biaya penyimpanan per unit (H)" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.h }}
                        </template>
                    </Column>
                    <Column field="nilai_eoq" header="Nilai Eoq" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nilai_eoq }}
                        </template>
                    </Column>
                    <Column field="periode" header="Periode" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.periode }}
                        </template>
                    </Column>
                    <Column field="tanggal_perhitungan" header="Tanggal Perhitungan " :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ formatDate(slotProps.data.tanggal_perhitungan) }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;" header="Action">
                        <template #body="slotProps">
                            <!-- <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editPenjualan(slotProps.data)" /> -->
                            <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeleteEoq(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data Eoq</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deleteEoqDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="eoq"
                            >Are you sure you want to delete eoq <b>{{ eoq.nama_barang }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deleteEoqDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deleteEoq" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createEoqDialog" :style="{ width: '900px' }" header="Create New Penjualan" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="supplier">Pilih Bahan Baku</label>
                        <AutoComplete placeholder="Search bahan baku..." id="dd" :dropdown="true" v-model="barang" :suggestions="autoFilteredValueBarangs" @complete="searchBarangs($event)" field="nama_barang" />
                    </div>
                    <div class="field">
                        <label for="periode">Periode (tahun)</label>
                        <InputText id="periode" type="text" required="true" v-model="periode" placeholder="peride dalam tahun" />
                    </div>
                    <!-- <div class="field">
                        <label for="tanggal_perhitungan">Tanggal</label>
                        <Calendar inputId="tanggal_perhitungan" v-model="tanggal_perhitungan"></Calendar>
                    </div> -->
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createpenjualan" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createEoq" />
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

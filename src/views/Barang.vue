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

const listBarang = ref(null);
const barang = ref({});
const selectBarang = ref(null);
const loadData = ref(false);
const suppliers = ref([]);
const supplier = ref(null);
const supplierIdEdit = ref(null);

const id = ref(null);
const nama = ref(null);

const autoFilteredValueSuppliers = ref([]);

const createBarangDialog = ref(false);
const updateBarangDialog = ref(false);
const deleteBarangDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getBarangs();
});

/* Api get Mapping Barang */
const getBarangs = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/barangs`);
        listBarang.value = response.data.barang;
        console.log("Response data:", response.data.barang);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching barang:', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch list barang', life: 3000 });
        loadData.value = false;
    }
};

const confirmDeleteBarang = (data) => {
    barang.value = data;
    deleteBarangDialog.value = true;
};

const deleteBarang = async () => {
    console.log("id barang: ", barang.value.id);

    try {
        const response = await axiosInstance.delete(`/barang?id=${barang.value.id}`);
        console.log("Response Delete: ", response.data.message);

        deleteBarangDialog.value = false;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message + ' ' + barang.value.nama_barang, life: 3000 });
        getBarangs();
    } catch (error) {
        console.log("Error Delete Barang: ", error);
        deleteBarangDialog.value = false;
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete barang ' + barang.value.nama_barang, life: 3000 });
    }
};

const openNewBarang = () => {
    getSuppliers();
    barang.value = {};
    createBarangDialog.value = true;
};

const createBarang = async () => {
    try {
        const response = await axiosInstance.post(`/barang`, {
            nama_barang: nama.value,
            id_supplier: supplier.value.id
        });
        console.log("Barang created: ", response.data);
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
        nama.value = '';
        supplier.value = '';

        createBarangDialog.value = false;
        getBarangs();
    } catch (error) {
        console.log("Error creating barang: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create barang', life: 3000 });
        createBarangDialog.value = false;
    }
};

const editBarang = (editBarang) => {
    barang.value = [{...editBarang}];
    updateBarangDialog.value = true;
    console.log("update : ", barang.value);

    getSuppliers();

    // Populate form fields with existing barang data
    id.value = barang.value[0].id;
    nama.value = barang.value[0].nama_barang;
    supplierIdEdit.value = barang.value[0].id_supplier;
};

const saveBarang = async () => {
    const isDataChanged =
        nama.value !== barang.value[0].nama
        supplierIdEdit.value !== supplier.value.id;

    if (isDataChanged) {
        try {
            const response = await axiosInstance.put(`/barang`, {
                id: id.value,
                nama_barang: nama.value,
                id_supplier: supplier.value.id
            });
            console.log("barang updated: ", response.data);
            toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
            updateBarangDialog.value = false;
            supplier.value = '';
            getBarangs();
        } catch (error) {
            console.log("Error updating barang: ", error);
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to update barang', life: 3000 });
            updateBarangDialog.value = false;
        }
    } else {
        updateBarangDialog.value = false;
    }
};

const getSuppliers = async () => {
    try {
        const response = await axiosInstance.get(`/suppliers`);
        suppliers.value = response.data.supplier;
        console.log("Response data:", response.data.supplier);
    } catch (error) {
        console.error('Error fetching supplier :', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch suppliers', life: 3000 });
    }
};

/* Function search or autocomplete suppliers */
const searchSuppliers = (event) => {
    console.log("suppliers value :", suppliers.value);
    setTimeout(() => {
        if (!event.query.trim().length) {
            autoFilteredValueSuppliers.value = [...suppliers.value];
            console.log("Auto filter suppliers : ", autoFilteredValueSuppliers.value);
        } else {
            autoFilteredValueSuppliers.value = suppliers.value.filter((supplier) => {
                return supplier.nama.toLowerCase().startsWith(event.query.toLowerCase());
            });
        }
    }, 25)
};

const hideDialogEdit = () => {
    updateBarangDialog.value = false;
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
                <h5 class="font-bold">Barang</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="openNewBarang" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listBarang"
                    v-model:selection="selectBarang"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Barang"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Barang</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 10rem"></Column>
                    <Column field="nama_barang" header="Nama Barang" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nama_barang }}
                        </template>
                    </Column>
                    <Column field="supplier_nama" header="Supplier" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.supplier_nama }}
                        </template>
                    </Column>
                    <Column field="supplier_perusahaan" header="Perusahaan Supplier" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.supplier_perusahaan }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editBarang(slotProps.data)" />
                            <Button icon="pi pi-trash" class="mt-2" severity="danger" rounded @click="confirmDeleteBarang(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data suppliers</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deleteBarangDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="barang"
                            >Are you sure you want to delete barang <b>{{ barang.nama_barang }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deleteBarangDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deleteBarang" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createBarangDialog" :style="{ width: '450px' }" header="Create New Barang" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="nama">Nama Barang</label>
                        <InputText id="nama" type="text" required="true" v-model="nama" placeholder="Nama barang" />
                    </div>
                    <div class="field">
                        <label for="supplier">Supplier</label>
                        <h5>Pilih Supplier</h5>
                            <AutoComplete placeholder="Search" id="dd" :dropdown="true" v-model="supplier" :suggestions="autoFilteredValueSuppliers" @complete="searchSuppliers($event)" field="nama" />
                    </div>
                    <template #footer>
                        <!-- <Button label="Create" icon="pi pi-check" text="" @click="createSupplier" /> -->
                        <Button label="Create" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="createBarang" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="updateBarangDialog" :style="{ width: '450px' }" header="Update Supplier" :modal="true" class="p-fluid"  >

                    <div class="card">
                        <div class="field">
                            <label for="nama">Nama</label>
                            <InputText id="nama" v-model="nama" required="true" autofocus  />
                        </div>
                        <div class="field">
                            <label for="supplier">Supplier</label>
                            <AutoComplete placeholder="Search" id="dd" :dropdown="true" v-model="supplier" :suggestions="autoFilteredValueSuppliers" @complete="searchSuppliers($event)" field="nama" />
                        </div>
                        
                        <div class="formgrid grid">
                            <div class="field col">
                                <Button label="Cancel" icon="pi pi-times" class="p-button-outlined p-button-danger mr-2 mb-2" @click="hideDialogEdit" />
                            </div>
                            <div class="field col">
                                <Button label="Update" icon="pi pi-check" class="p-button-outlined p-button-success mr-2 mb-2" @click="saveBarang" />
                            </div>
                        </div>
                    </div>
                </Dialog>
           </div>
        </div>
    </div>
</template>

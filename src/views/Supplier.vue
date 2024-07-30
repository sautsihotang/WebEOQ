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

const listSupplier = ref(null);
const supplier = ref({});
const selectSupplier = ref(null);
const loadData = ref(false);
const newSupplier = ref(null);

const createSupplierDialog = ref(false);
const deleteSupplierDialog = ref(false);
const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    getsuppliers();
});

/* Api get Mapping Supplier */
const getsuppliers = async () => {
    try {
        loadData.value = true;
        const response = await axiosInstance.get(`/suppliers`);
        listSupplier.value = response.data.supplier;
         console.log("Response data:", response.data.supplier);
        loadData.value = false;
    } catch (error) {
        console.error('Error fetching supplier Combinations:', error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to fetch suppliers', life: 2000 });
        loadData.value = false
    }
};



const opennewSupplier = () => {
    supplier.value = {};
    createSupplierDialog.value = true;
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
                <h5 class="font-bold">Supplier</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Create New " icon="pi pi-plus" class="mr-2" severity="success" @click="opennewSupplier" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listSupplier"
                    v-model:selection="selectSupplier"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} suppliers"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Suppliers</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 10rem"></Column>
                    <Column field="nama" header="Nama" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nama }}
                        </template>
                    </Column>
                    <Column field="perusahaan" header="Perusahaan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.perusahaan }}
                        </template>
                    </Column>
                    <Column field="kontak" header="Kontak" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.kontak }}
                        </template>
                    </Column>
                    <Column field="alamat" header="Alamat" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.alamat }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="mr-2 p-button-warning mr-2" severity="success" rounded @click="editblock(slotProps.data)" />
                            <Button icon="pi pi-trash" class="mt-2" severity="warning" rounded @click="confirmDeleteBlock(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="loadData" :style="{ width: '900px' }" header="Loading..." :modal="true" class="p-fluid">
                    <div class="spinner-container">
                        <div class="spinner"></div>
                        <p>Sedang load data Block Combinations</p>
                    </div>
                </Dialog>

                <Dialog v-model:visible="deleteSupplierDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="block"
                            >Are you sure you want to delete <b>{{ block.combination }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" text @click="deleteSupplierDialog= false" />
                        <Button label="Yes" icon="pi pi-check" text @click="deleteBlock" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="createSupplierDialog" :style="{ width: '450px' }" header="Create Block Combinnation" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="description">New Block Combination</label>
                        <InputText id="block" type="text" required="true" v-model="newSupplier" maxlength="3" />
                    </div>
                    <template #footer>
                        <Button label="Create" icon="pi pi-check" text="" @click="createBlockCombination" />
                    </template>
                </Dialog>
            </div>
        </div>
    </div>
</template>

<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';
import { read, writeFile, utils } from 'xlsx';

const toast = useToast();

const baseURL = import.meta.env.VITE_BASE_URL_SERVICE;

const axiosInstance = axios.create({
    baseURL: baseURL,
    headers: {
        'Content-Type': 'application/json',
    },
});

const listStock = ref(null);    
const selectStock = ref(null);
const startDate = ref(null);
const endDate = ref(null);

const dt = ref(null);
const filters = ref({});

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    // getStocks();
});

const getStocks = async () => {
    if (startDate.value == null || endDate.value == null) {
        toast.add({ severity: 'error', summary: 'Error', detail: "Start Date dan End Date tidak boleh kosong !", life: 3000 });
        return;
    }
    try {
        const response = await axiosInstance.post(`/stock`, {
            start_date : startDate.value,
            end_date : endDate.value,
        });
        console.log("stock : ", response.data);
        listStock.value = response.data.stock;
        toast.add({ severity: 'success', summary: 'Success', detail: response.data.message, life: 3000 });
    } catch (error) {
        console.log("Error creating eoq: ", error);
        toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to create eoq', life: 3000 });
    }
};

const exportData = () => {
    if (!listStock.value || listStock.value.length === 0) {
        toast.add({ severity: 'error', summary: 'Error', detail: 'No data available to export', life: 3000 });
        return;
    }

    // Prepare data for Excel
    const formattedData = listStock.value.map(item => [
        item.id_barang,
        item.nama_barang,
        item.total_kuantitas_pemesanan,
        item.total_kuantitas_penjualan,
        item.stok_barang,
    ]);

    // Create an array with start and end dates as headers
    const headerData = [
        ['Report Date Range', '', '', '', ''],
        [`Start Date: `, `${startDate.value.toISOString().split('T')[0]}`, '', '', ''],
        [`End Date: `, `${endDate.value.toISOString().split('T')[0]}`, '', '', ''],
        [], // Empty row for spacing
        ['ID Bahan Baku', 'Nama Bahan Baku', 'Total Pemesanan', 'Total Penjualan', 'Stock Bahan Baku'], // Column headers
        ...formattedData // Add stock data
    ];

    // Create a worksheet and a workbook
    const ws = utils.aoa_to_sheet(headerData);
    const wb = utils.book_new();
    utils.book_append_sheet(wb, ws, 'Stock Bahan Baku');

    // Create an Excel file and trigger download
    writeFile(wb, `Stock_Bahan_Baku_${startDate.value.toISOString().split('T')[0]}_${endDate.value.toISOString().split('T')[0]}.xlsx`);
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
                <h5 class="font-bold">Stock Bahan Baku</h5>
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2 formgroup-inline">
                            <div class="field">
                                <Button label="Export " icon="pi pi-file-export" class="mr-2" severity="warning" @click="exportData" />
                            </div>
                        </div>
                    </template>
                    <template v-slot:end>
                        <div class="my-2 formgroup-inline">
                            <div class="field">
                                <label for="start_date">Start Date</label>
                                <Calendar inputId="start_date" v-model="startDate"></Calendar>
                            </div>
                            <div class="field">
                                <label for="end_date">End Date</label>
                                <Calendar inputId="end_date" v-model="endDate"></Calendar>
                            </div>
                            <Button label="Cari Stock Bahan Baku " icon="pi pi-search" class="mr-2" severity="success" @click="getStocks" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="listStock"
                    v-model:selection="selectStock"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} Stocks"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Stock Bahan Baku</h5>
                            <IconField iconPosition="left" class="block mt-2 md:mt-0">
                                <InputIcon class="pi pi-search" />
                                <InputText class="w-full sm:w-auto" v-model="filters['global'].value" placeholder="Search..." />
                            </IconField>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 10rem"></Column>
                    <!-- <Column field="id_barang" header="ID Barang" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.id_barang }}
                        </template>
                    </Column> -->
                    <Column field="nama_barang" header="Nama Bahan Baku" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.nama_barang }}
                        </template>
                    </Column>
                    <Column field="total_kuantitas_pemesanan" header="Total Pemesanan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.total_kuantitas_pemesanan }}
                        </template>
                    </Column>
                    <Column field="total_kuantitas_penjualan" header="Total Penjualan" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.total_kuantitas_penjualan }}
                        </template>
                    </Column>
                    <Column field="stok_barang" header="Stock Bahan Baku" :sortable="true" headerStyle="width:15%; min-width:10rem;">
                        <template #body="slotProps">
                            {{ slotProps.data.stok_barang }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <template v-if="slotProps.data.stok_barang > 1">
                                <Button label="Stock Tersedia" icon="pi pi-check" iconPos="right" class="mr-2 mb-2"></Button>
                            </template>
                            <template v-else>
                                <Button label="Stock Habis" icon="pi pi-times" iconPos="right" severity="danger" class="mr-2 mb-2"></Button>
                            </template>
                        </template>
                    </Column>
                </DataTable>
           </div>
        </div>
    </div>
</template>

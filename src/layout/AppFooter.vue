<script setup>
import { useLayout } from '@/layout/composables/layout';
import { computed } from 'vue';

const { layoutConfig } = useLayout();

const logoUrl = computed(() => {
    return `layout/images/${layoutConfig.darkTheme.value ? 'logo-white' : 'logo-dark'}.svg`;
});
</script>

<template>
    <div class="layout-footer">
        <!-- <img :src="logoUrl" alt="Logo" height="20" class="mr-2" /> -->
        <!-- by -->
        <span class="font-medium ml-2">Aplikasi EOQ </span>
        <span class="font-medium ml-2 mr-2">by </span>
        <img src="/demo/images/login/eoq.png" alt="Logo" height="20" class="mr-2" />
        CV. LAM CEMERLANG
        
        
    </div>
</template>
<style lang="scss" scoped></style>

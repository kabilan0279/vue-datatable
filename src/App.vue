<template>
  <div class="p-2">
    <h1 class="text-2xl font-bold mb-4 "> Client Information</h1>
    <DataTable />
     <router-view />
  </div>
</template>

<script setup>
import DataTable from './components/HelloWorld.vue';

</script>

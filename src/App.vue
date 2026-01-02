<template>
  <div class="p-2 bg-gray-50">
    <h1 class="text-2xl text-center font-bold mb-2 ">CLIENT INFORMATION</h1>
    <DataTable />
     <router-view />
  </div>
</template>

<script setup>
import DataTable from './components/HelloWorld.vue';

</script>

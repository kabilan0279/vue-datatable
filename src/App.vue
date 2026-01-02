<template>
  <div class="p-2 bg-gray-50">
    <div class="flex items-center justify-between mb-4">
      <!-- Left: Back Button -->
      <button
        @click="moveToVueBasics"
        class="flex items-center gap-2 px-4 py-2 text-gray-700 hover:text-blue-600 rounded-lg transition"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="w-6 h-6"
          viewBox="0 0 24 24"
          fill="currentColor"
        >
          <path
            d="M12 3l9 7v11a1 1 0 01-1 1h-5v-6H9v6H4a1 1 0 01-1-1V10l9-7z"
          />
        </svg>
        <span class="text-sm font-medium">HOME</span>
      </button>

      <!-- Center: Heading -->
      <h1 class="text-2xl font-bold text-center flex-1">CLIENT INFO</h1>

      <!-- Right: Spacer (keeps heading centered) -->
      <div class="w-[88px]"></div>
    </div>

    <DataTable />
    <router-view />
  </div>
</template>

<script setup>
import DataTable from "./components/HelloWorld.vue";

const moveToVueBasics = () => {
  alert("Navigating to vueBasics");
  // router.push({ name: 'vueBasics' });
};
</script>

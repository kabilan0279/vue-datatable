<template>
  <div class="p-4">
    <div>
      <button
        @click="moveToVueBasics"
        class="bg-blue-600 px-4 py-2 text-white rounded-lg"
      >
        back
      </button>
    </div>

    <!-- Global Search -->
    <div
      class="mb-4 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-2 sm:gap-0"
    >
      <div class="flex items-center gap-2">
        Show
        <select
          v-model.number="perPage"
          class="border border-gray-300 rounded px-2 py-1 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option v-for="n in [10, 50, 100, 300]" :key="n" :value="n">
            {{ n }}
          </option>
        </select>
        entries
      </div>

      <div class="exportrows">
        <button 
          @click="exportToExcel"
          class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors duration-200 flex items-center gap-2"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
            <polyline points="7 10 12 15 17 10" />
            <line x1="12" y1="15" x2="12" y2="3" />
          </svg>
          EXCEL EXPORT
        </button>
      </div>

      <div class="relative flex gap-2" ref="dropdownContainerRef">
        <!-- Dropdown button -->
        <button
          @click="toggleColumnDropdown"
          type="button"
          class="w-40 border rounded-md bg-white px-3 py-2 text-left flex justify-between items-center focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          Hide Column
          <svg
            class="w-4 h-4 ml-2 transition-transform duration-200"
            :class="{ 'rotate-180': showColumnDropdown }"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M19 9l-7 7-7-7"
            />
          </svg>
        </button>

        <!-- Dropdown content -->
        <div
          v-show="showColumnDropdown"
          class="absolute left-0 top-full mt-1 w-40 bg-white border rounded-md shadow-md z-10"
        >
          <div class="flex flex-col gap-1 p-2 max-h-60 overflow-y-auto">
            <label
              v-for="(column, index) in columnVisibilityOptions"
              :key="column.key"
              class="inline-flex items-center cursor-pointer hover:bg-gray-50 p-1 rounded"
            >
              <input
                type="checkbox"
                :checked="column.visible"
                @change="toggleColumnVisibility(index)"
                class="form-checkbox h-4 w-4 text-blue-600 rounded focus:ring-blue-500"
              />
              <span class="ml-2 text-sm text-gray-700">{{ column.label }}</span>
            </label>
          </div>
        </div>

        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search..."
          class="border border-gray-300 rounded px-3 py-2 w-64 lg:w-80 sm:w-32 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>
    </div>

    <!-- Table Container -->
    <div class="overflow-x-auto">
      <table class="min-w-[700px] divide-y divide-gray-200">
        <thead class="bg-gray-50">
          <tr>
            <th
              v-for="column in visibleColumns"
              :key="column.key"
              @click="column.key !== 'View' ? sortBy(column.key) : null"
              :class="[
                'px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider',
                column.key !== 'View' ? 'cursor-pointer hover:text-blue-700' : ''
              ]"
            >
              {{ column.label }}
              <span v-if="sortKey === column.key && column.key !== 'View'">
                {{ sortOrder === 1 ? "▲" : "▼" }}
              </span>
            </th>
          </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
          <tr
            v-for="row in paginatedData"
            :key="row.id"
            class="hover:bg-gray-100"
          >
            <td
              v-for="column in visibleColumns"
              :key="column.key"
              class="px-6 py-4 whitespace-nowrap truncate max-w-[150px]"
            >
              <template v-if="column.key === 'View'">
                <button
                  @click="openDrawer(row)"
                  class="rounded-md bg-gray-950/5 px-3 py-1 text-sm font-semibold text-gray-900 hover:bg-gray-950/10"
                >
                  VIEW
                </button>
              </template>
              <template v-else>
                {{ row[column.key] }}
              </template>
            </td>
          </tr>
          <tr v-if="paginatedData.length === 0">
            <td
              :colspan="visibleColumns.length"
              class="px-6 py-4 text-center text-gray-500"
            >
              No results found.
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div
      class="mt-4 flex flex-col sm:flex-row items-center justify-between gap-4"
    >
      <!-- Centered page numbers -->
      <div class="flex flex-wrap items-center justify-center gap-2 flex-1">
        <!-- First page -->
        <button
          :class="[
            'w-8 h-8 rounded-full flex items-center justify-center',
            currentPage === 1 ? 'bg-blue-600 text-white' : 'bg-gray-200',
          ]"
          @click="goToPage(1)"
        >
          1
        </button>

        <!-- Left ellipsis -->
        <span v-if="currentPage > 4" class="px-2">...</span>

        <!-- Middle pages -->
        <template v-for="page in visiblePages" :key="page">
          <button
            v-if="page > 1 && page < totalPages"
            :class="[
              'w-8 h-8 rounded-full flex items-center justify-center',
              currentPage === page ? 'bg-blue-600 text-white' : 'bg-gray-200',
            ]"
            @click="goToPage(page)"
          >
            {{ page }}
          </button>
        </template>

        <!-- Right ellipsis -->
        <span v-if="currentPage < totalPages - 3" class="px-2">...</span>

        <!-- Last page -->
        <button
          v-if="totalPages > 1"
          :class="[
            'w-8 h-8 rounded-full flex items-center justify-center',
            currentPage === totalPages
              ? 'bg-blue-600 text-white'
              : 'bg-gray-200',
          ]"
          @click="goToPage(totalPages)"
        >
          {{ totalPages }}
        </button>
      </div>

      <!-- Right-aligned Previous & Next -->
      <div class="flex items-center gap-2">
        <button
          class="px-3 py-1 bg-gray-200 rounded disabled:opacity-50 hover:bg-gray-300"
          :disabled="currentPage === 1"
          @click="prevPage"
        >
          Previous
        </button>

        <button
          class="px-3 py-1 bg-gray-200 rounded disabled:opacity-50 hover:bg-gray-300"
          :disabled="currentPage === totalPages"
          @click="nextPage"
        >
          Next
        </button>
      </div>
    </div>

    <!-- Drawer -->
    <dialog
      id="drawer"
      aria-labelledby="drawer-title"
      class="fixed inset-0 size-auto max-h-none max-w-none overflow-hidden bg-transparent not-open:hidden backdrop:bg-transparent"
      ref="drawerRef"
      @click="handleDrawerClick"
    >
      <div
        class="absolute inset-0 bg-gray-500/75 transition-opacity duration-500 ease-in-out"
      ></div>

      <div
        tabindex="0"
        class="absolute inset-0 pl-10 focus:outline-none sm:pl-16"
      >
        <div
          class="group/dialog-panel relative ml-auto block size-full max-w-md transform transition duration-500 ease-in-out sm:duration-700"
          :class="drawerOpen ? 'translate-x-0' : 'translate-x-full'"
          ref="drawerPanelRef"
        >
          <!-- Close button, show/hide based on slide-over state. -->
          <div
            class="absolute top-0 left-0 -ml-8 flex pt-4 pr-2 duration-500 ease-in-out sm:-ml-10 sm:pr-4"
          >
            <button
              type="button"
              @click="closeDrawer"
              class="relative rounded-md text-gray-300 hover:text-white focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600"
            >
              <span class="absolute -inset-2.5"></span>
              <span class="sr-only">Close panel</span>
              <svg
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.5"
                data-slot="icon"
                aria-hidden="true"
                class="size-6"
              >
                <path
                  d="M6 18 18 6M6 6l12 12"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </button>
          </div>

          <div
            class="relative flex h-full flex-col overflow-y-auto bg-white py-6 shadow-xl"
          >
            <div class="px-4 sm:px-6">
              <h2
                id="drawer-title"
                class="text-base font-semibold text-gray-900"
              >
                Client Details
              </h2>
            </div>
            <div class="relative mt-6 flex-1 px-4 sm:px-6">
              <!-- Content -->
              <div v-if="selectedRow" class="space-y-4">
                <div class="grid grid-cols-2 gap-4">
                  <div>
                    <label class="text-sm font-medium text-gray-500" for="id"
                      >ID</label
                    >
                    <input
                      type="text"
                      id="id"
                      :value="selectedRow.id"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div>
                    <label
                      class="text-sm font-medium text-gray-500"
                      for="client_code"
                      >Client Code</label
                    >
                    <input
                      type="text"
                      id="client_code"
                      :value="selectedRow.client_code"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div class="col-span-2">
                    <label class="text-sm font-medium text-gray-500" for="name"
                      >Name</label
                    >
                    <input
                      type="text"
                      id="name"
                      :value="selectedRow.name"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div class="col-span-2">
                    <label class="text-sm font-medium text-gray-500" for="email"
                      >Email</label
                    >
                    <input
                      type="email"
                      id="email"
                      :value="selectedRow.email"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div>
                    <label
                      class="text-sm font-medium text-gray-500"
                      for="mobile"
                      >Mobile</label
                    >
                    <input
                      type="text"
                      id="mobile"
                      :value="selectedRow.mobile"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div>
                    <label class="text-sm font-medium text-gray-500" for="pan"
                      >PAN</label
                    >
                    <input
                      type="text"
                      id="pan"
                      :value="selectedRow.pan"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>

                  <div class="col-span-2">
                    <label
                      class="text-sm font-medium text-gray-500"
                      for="entry_date"
                      >Entry Date</label
                    >
                    <input
                      type="text"
                      id="entry_date"
                      :value="selectedRow.entry_date"
                      readonly
                      class="mt-1 w-full border rounded-md bg-gray-100 px-3 py-2 text-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, onUnmounted } from "vue";
import * as XLSX from 'xlsx';

// Table data
const tableData = ref([]);

// All columns with visibility state
const columnVisibilityOptions = ref([
  { key: "id", label: "ID", visible: true },
  { key: "client_code", label: "Client Code", visible: true },
  { key: "name", label: "Name", visible: true },
  { key: "email", label: "Email", visible: true },
  { key: "mobile", label: "Mobile", visible: true },
  { key: "pan", label: "PAN", visible: true },
  { key: "entry_date", label: "ENTRY DATE", visible: true },
  { key: "View", label: "VIEW", visible: true },
]);

// Computed property for visible columns only
const visibleColumns = computed(() => {
  return columnVisibilityOptions.value.filter((column) => column.visible);
});

// Refs
const dropdownContainerRef = ref(null);
const drawerPanelRef = ref(null);

// Dropdown visibility
const showColumnDropdown = ref(false);

// Drawer state
const drawerOpen = ref(false);
const selectedRow = ref(null);
const drawerRef = ref(null);

// Sorting
const sortKey = ref("");
const sortOrder = ref(1);

const sortBy = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = -sortOrder.value;
  } else {
    sortKey.value = key;
    sortOrder.value = 1;
  }
};

// Global search
const searchQuery = ref("");

// Filtered data
const filteredData = computed(() => {
  if (!searchQuery.value) return tableData.value;
  const query = searchQuery.value.toLowerCase();
  return tableData.value.filter((row) =>
    visibleColumns.value.some((col) =>
      String(row[col.key]).toLowerCase().includes(query)
    )
  );
});

// Sorted data
const sortedData = computed(() => {
  if (!sortKey.value) return filteredData.value;
  return [...filteredData.value].sort((a, b) => {
    const aValue = a[sortKey.value];
    const bValue = b[sortKey.value];

    if (!isNaN(Date.parse(aValue)) && !isNaN(Date.parse(bValue))) {
      return (new Date(aValue) - new Date(bValue)) * sortOrder.value;
    } else if (!isNaN(aValue) && !isNaN(bValue)) {
      return (aValue - bValue) * sortOrder.value;
    } else {
      return (
        aValue.toString().localeCompare(bValue.toString()) * sortOrder.value
      );
    }
  });
});

// Pagination
const currentPage = ref(1);
const perPage = ref(10);

const totalPages = computed(() =>
  Math.ceil(sortedData.value.length / perPage.value)
);

// Compute visible page numbers (with ellipsis logic)
const visiblePages = computed(() => {
  const pages = [];
  const total = totalPages.value;

  if (total <= 7) {
    // Show all pages if total is 7 or less
    for (let i = 2; i < total; i++) {
      pages.push(i);
    }
  } else {
    // Show pages around current page
    const start = Math.max(2, currentPage.value - 1);
    const end = Math.min(total - 1, currentPage.value + 1);

    for (let i = start; i <= end; i++) {
      pages.push(i);
    }
  }

  return pages;
});

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * perPage.value;
  const end = start + perPage.value;
  return sortedData.value.slice(start, end);
});

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++;
};

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};

const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
  }
};

// Column dropdown functions
const toggleColumnDropdown = () => {
  showColumnDropdown.value = !showColumnDropdown.value;
};

const toggleColumnVisibility = (index) => {
  columnVisibilityOptions.value[index].visible =
    !columnVisibilityOptions.value[index].visible;
};

// Close dropdown when clicking outside
const handleClickOutside = (event) => {
  // Close column dropdown if clicking outside
  if (
    dropdownContainerRef.value &&
    !dropdownContainerRef.value.contains(event.target)
  ) {
    showColumnDropdown.value = false;
  }
};

// Drawer functions
const openDrawer = (row) => {
  selectedRow.value = row;
  drawerOpen.value = true;
  // Show the dialog element
  if (drawerRef.value) {
    drawerRef.value.showModal();
  }
};

const closeDrawer = () => {
  drawerOpen.value = false;
  selectedRow.value = null;
  if (drawerRef.value) {
    drawerRef.value.close();
  }
};

// Handle drawer click to close when clicking on backdrop
const handleDrawerClick = (event) => {
  // If the click is on the backdrop (not on the drawer panel), close the drawer
  if (drawerPanelRef.value && !drawerPanelRef.value.contains(event.target)) {
    closeDrawer();
  }
};

// Excel Export Function
const exportToExcel = () => {
  try {
    // Prepare data for export (excluding the "View" column)
    const exportData = sortedData.value.map(row => {
      const exportRow = {};
      visibleColumns.value.forEach(col => {
        // Skip the "View" column from export
        if (col.key !== 'View') {
          exportRow[col.label] = row[col.key];
        }
      });
      return exportRow;
    });

    if (exportData.length === 0) {
      alert("No data to export!");
      return;
    }

    // Create worksheet
    const ws = XLSX.utils.json_to_sheet(exportData);
    
    // Create workbook
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Client Data");

    // Generate Excel file
    const fileName = `client_data_export_${new Date().toISOString().split('T')[0]}.xlsx`;
    XLSX.writeFile(wb, fileName);

    console.log(`Excel file "${fileName}" exported successfully with ${exportData.length} rows.`);
    
  } catch (error) {
    console.error("Error exporting to Excel:", error);
    alert("An error occurred while exporting to Excel. Please try again.");
  }
};

// Reset to first page when search query changes or perPage changes
watch([searchQuery, perPage], () => {
  currentPage.value = 1;
});

// Fetch JSON from public folder
onMounted(async () => {
  try {
    const res = await fetch(
      "https://phpstack-994140-5900184.cloudwaysapps.com/f7/users.json"
    );
    tableData.value = await res.json();
  } catch (error) {
    console.error("Error loading JSON:", error);
  }

  // Add click event listener to close dropdown when clicking outside
  document.addEventListener("click", handleClickOutside);
});

onUnmounted(() => {
  // Cleanup event listener
  document.removeEventListener("click", handleClickOutside);
});

// back button click event
const moveToVueBasics = () => {
  alert("Navigating to vueBasics");
  // router.push({ name: 'vueBasics' });
};
</script>

<style scoped>
th:hover {
  color: #1d4ed8; /* Tailwind blue-700 */
}

/* Hover effect for buttons */
button.bg-gray-200:hover:not(:disabled) {
  background-color: #e5e7eb; /* Tailwind gray-300 */
}
button.bg-blue-600:hover:not(:disabled) {
  background-color: #1d4ed8; /* Tailwind blue-700 */
}

/* Table styling */
table {
  border-collapse: collapse;
  width: 100%;
  table-layout: fixed;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
th,
td {
  padding: 12px 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
th {
  background-color: #f1f1f1;
  color: black;
  text-transform: uppercase;
}
tr:hover {
  background-color: #f1f1f1;
}
caption {
  caption-side: top;
  font-size: 1.5em;
  margin-bottom: 10px;
}

/* Dropdown arrow rotation */
.rotate-180 {
  transform: rotate(180deg);
}

/* Disable cursor for View header */
th:last-child {
  cursor: default !important;
}
</style>
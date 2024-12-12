<template>
  <v-app theme="dark">
    <v-main class="pa-6">
      <v-tabs v-model="activeTab" class="mb-4">
        <v-tab prepend-icon="mdi-pin" value="pinned">PINNED (4)</v-tab>
        <v-tab value="transit">TRANSIT NOTES</v-tab>
        <v-tab value="drafts">DRAFTS</v-tab>
      </v-tabs>

      <v-row class="mb-4">
        <v-col cols="12">
          <v-btn prepend-icon="mdi-magnify" variant="text" color="grey">
            Advanced Search
          </v-btn>
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field
            v-model="searchReference"
            label="Shipping Note Reference"
            placeholder="Enter value"
            variant="outlined"
            density="compact"
            @input="filterItems"
          ></v-text-field>
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            label="Departure Office"
            :items="['57 items available']"
            variant="outlined"
            density="compact"
          ></v-select>
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            label="Destination Office"
            :items="['57 items available']"
            variant="outlined"
            density="compact"
          ></v-select>
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            label="Status"
            :items="['9 items available']"
            variant="outlined"
            density="compact"
          ></v-select>
        </v-col>
      </v-row>

      <v-btn color="primary" prepend-icon="mdi-plus" class="mb-4" size="large">
        CREATE NEW SHIPPING NOTE
      </v-btn>

      <v-data-table
        :headers="headers"
        :items="filteredItems"
        :loading="loading"
        hover
        class="elevation-1"
      >
        <template v-slot:item.status="{ item }">
          <v-chip
            :color="getStatusColor(item.status)"
            size="small"
            class="text-uppercase"
          >
            {{ item.status }}
          </v-chip>
        </template>
      </v-data-table>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";

const activeTab = ref("transit");
const loading = ref(true);
const items = ref([]);
const searchReference = ref('')

const headers = [
  { title: "Shipping Note Reference", key: "reference" },
  { title: "Status", key: "status" },
  { title: "Operation", key: "operation" },
  { title: "Principle Name", key: "principleName" },
  { title: "Departure Office", key: "departureOffice" },
  { title: "Destination Office", key: "destinationOffice" },
  { title: "Submitted Date", key: "submittedDate" },
  { title: "Departure Date", key: "departureDate" },
  { title: "Scheduled Date Time", key: "scheduledDateTime" },
];

const getStatusColor = (status) => {
  const colors = {
    PAID: "success",
    DEPARTED: "warning",
    ARRIVED: "success",
    EXITED: "info",
    LATE: "error",
  };
  return colors[status] || "grey";
};

const fetchData = async () => {
  try {
    loading.value = true;
    const response = await axios.get("/api/L0Yu2N");
    const responseData = response.data.content;
    console.log(responseData);

    if (Array.isArray(responseData)) {
      items.value = responseData.map((item) => ({
        reference: item.reference,
        status: item.status,
        operation: item.operation.name,
        principleName: item.principalName,
        departureOffice: item.officeDeparture.code,
        destinationOffice: item.officeDestination.code,
        submittedDate: new Date(item.submittedDate).toLocaleString(),
        departureDate: new Date(item.departureDateTime).toLocaleString(),
        scheduledDateTime: new Date(item.estimatedArrivalDate).toLocaleString(),
      }));
    } else if (typeof responseData === "object" && responseData !== null) {
      // If the response is a single object, wrap it in an array
      items.value = [
        {
          reference: responseData.reference,
          status: responseData.status,
          operation: responseData.operation.name,
          principleName: responseData.principalName,
          departureOffice: responseData.officeDeparture.code,
          destinationOffice: responseData.officeDestination.code,
          submittedDate: new Date(responseData.submittedDate).toLocaleString(),
          departureDate: new Date(
            responseData.departureDateTime
          ).toLocaleString(),
          scheduledDateTime: new Date(
            responseData.estimatedArrivalDate
          ).toLocaleString(),
        },
      ];
    } else {
      console.error("Unexpected response format:", responseData);
      items.value = [];
    }
  } catch (error) {
    console.error("Error fetching data:", error);
    items.value = [];
  } finally {
    loading.value = false;
  }
};

const filteredItems = computed(() => {
  if (!searchReference.value) return items.value
  const searchTerm = searchReference.value.toLowerCase()
  return items.value.filter(item => 
    item.reference.toLowerCase().includes(searchTerm)
  )
})

onMounted(() => {
  fetchData();
});
</script>

<style scoped>
.v-data-table {
  background-color: transparent !important;
}

.v-data-table :deep(th) {
  color: #9e9e9e !important;
  font-weight: 500 !important;
}

.v-data-table :deep(td) {
  color: #ffffff !important;
}
</style>

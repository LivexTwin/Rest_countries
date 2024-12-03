<template>
  <div class="container">
    <!-- Search box with a search button -->
    <div class="search_container">
      <div class="input_wrapper">
        <input
          type="text"
          v-model="searchQuery"
          @keyup.enter="searchCountries"
          placeholder="Search for a country..."
          class="search_input"
        />
      </div>

      <!-- Search Button -->
    </div>

    <!-- Region Filter Dropdown -->
    <div class="filter-container">
      <!-- Region Filter Dropdown -->
      <SelectRoot
        class="SelectRoot"
        :model-value="selectedRegion"
        @update:modelValue="onRegionChange"
      >
        <SelectTrigger class="SelectTrigger">
          <SelectValue placeholder="Filter by Region" />
          <!-- Optionally add an icon for the dropdown -->
          <Icon icon="radix-icons:chevron-down" />
        </SelectTrigger>

        <!-- Dropdown Content (Select Portal) -->
        <SelectPortal>
          <SelectContent class="SelectContent" :side-offset="5">
            <!-- Scrollable dropdown -->
            <SelectScrollUpButton class="SelectScrollButton">
              <Icon icon="radix-icons:chevron-up" />
            </SelectScrollUpButton>

            <SelectViewport class="SelectViewport">
              <!-- Options in the dropdown -->
              <SelectItem value="Africa" class="SelectItem">
                <SelectItemIndicator class="SelectItemIndicator">
                  <Icon icon="radix-icons:check" />
                </SelectItemIndicator>
                <SelectItemText>Africa</SelectItemText>
              </SelectItem>
              <SelectItem value="Americas" class="SelectItem">
                <SelectItemIndicator class="SelectItemIndicator">
                  <Icon icon="radix-icons:check" />
                </SelectItemIndicator>
                <SelectItemText>Americas</SelectItemText>
              </SelectItem>
              <SelectItem value="Asia" class="SelectItem">
                <SelectItemIndicator class="SelectItemIndicator">
                  <Icon icon="radix-icons:check" />
                </SelectItemIndicator>
                <SelectItemText>Asia</SelectItemText>
              </SelectItem>
              <SelectItem value="Europe" class="SelectItem">
                <SelectItemIndicator class="SelectItemIndicator">
                  <Icon icon="radix-icons:check" />
                </SelectItemIndicator>
                <SelectItemText>Europe</SelectItemText>
              </SelectItem>
              <SelectItem value="Oceania" class="SelectItem">
                <SelectItemIndicator class="SelectItemIndicator">
                  <Icon icon="radix-icons:check" />
                </SelectItemIndicator>
                <SelectItemText>Oceania</SelectItemText>
              </SelectItem>
            </SelectViewport>

            <SelectScrollDownButton class="SelectScrollButton">
              <Icon icon="radix-icons:chevron-down" />
            </SelectScrollDownButton>
          </SelectContent>
        </SelectPortal>
      </SelectRoot>
    </div>
  </div>
  <!-- Show an error message if the fetch fails -->
  <div v-if="error" class="error-message">
    <p>{{ error }}</p>
  </div>

  <!-- Show loading message only when data is being fetched -->
  <div
    v-else-if="displayedCountries.length === 0 && !searchQuery"
    class="loading-message"
  >
    <p>Loading countries...</p>
  </div>

  <!-- Display countries or a message if no countries match -->
  <!-- <div v-if="filteredCountries.length > 0" class="country_cards">
    <CountryCard
      v-for="country in filteredCountries"
      :key="country.cca3"
      :country="country"
    />
  </div> -->

  <div v-if="paginatedCountries.length > 0" class="country_cards">
    <CountryCard
      v-for="country in paginatedCountries"
      :key="country.cca3"
      :country="country"
    />
  </div>

  <!-- Show More button -->
  <div
    v-if="paginatedCountries.length < filteredCountries.length"
    class="pagination"
  >
    <button @click="loadMore" class="show-more-btn">Show More</button>
  </div>

  <!-- Show 'No countries match your search' when filteredCountries is empty -->
  <div
    v-else-if="filteredCountries.length === 0 && searchQuery"
    class="no_results-message"
  >
    <p>No countries match your search.</p>
  </div>
</template>

<script setup>
import {
  SelectContent,
  SelectItem,
  SelectItemText,
  SelectRoot,
  SelectTrigger,
  SelectValue,
  SelectPortal,
  SelectItemIndicator,
  SelectScrollUpButton,
  SelectScrollDownButton,
  SelectViewport,
} from "radix-vue";
import { Icon } from "@iconify/vue";

// Define reactive variables
const selectedRegion = ref(""); // For selected region
const displayedCountries = ref([]); // For countries displayed based on selected region
const error = ref(null); // Error state for failed API requests
const searchQuery = ref(""); // For search query input
const paginatedCountries = ref([]); // Initialize as an empty array

// Function to fetch countries based on the selected region
async function fetchCountriesByRegion(region) {
  try {
    const apiUrl = `https://restcountries.com/v3.1/region/${region}`;
    const data = await $fetch(apiUrl);
    console.log("Fetched countries by region:", data); // Log the fetched data
    displayedCountries.value = data || [];
  } catch (err) {
    console.error("Error fetching countries by region:", err);
    error.value = "Error fetching countries";
  }
}

async function fetchAllCountries() {
  try {
    const apiUrl = "https://restcountries.com/v3.1/all";
    const data = await $fetch(apiUrl);
    console.log("Fetched all countries:", data); // Log the fetched data
    displayedCountries.value = data || [];
  } catch (err) {
    console.error("Error fetching all countries:", err);
    error.value = "Error fetching countries";
  }
}

// Define onRegionChange function to handle region selection changes
async function onRegionChange(region) {
  selectedRegion.value = region;
  try {
    const apiUrl = region
      ? `https://restcountries.com/v3.1/region/${region}`
      : "https://restcountries.com/v3.1/all";
    const data = await $fetch(apiUrl);
    displayedCountries.value = data || [];
  } catch (err) {
    console.error("Error fetching countries:", err);
    error.value = "Error fetching countries.";
  }
}

// Watch for changes in the search query or selected region
watch([searchQuery, selectedRegion], () => {
  currentPage.value = 1; // Reset to the first page when filters change
});

// Initially fetch all countries when the component is mounted
onMounted(() => {
  fetchAllCountries();
  paginatedCountries.value = filteredCountries.value.slice(0, itemsPerPage);
});

// Function to handle search execution (either by Enter key or search button)
watch([searchQuery, selectedRegion], () => {
  currentPage.value = 1; // Reset to the first page when filters change
  paginatedCountries.value = filteredCountries.value.slice(0, itemsPerPage); // Load the initial items again
});

// Computed property to filter countries based on the search query
const filteredCountries = computed(() => {
  const query = searchQuery.value.toLowerCase();
  return displayedCountries.value.filter((country) => {
    const matchesSearch = country.name.common.toLowerCase().includes(query);
    const matchesRegion =
      !selectedRegion.value || country.region === selectedRegion.value;
    return matchesSearch && matchesRegion;
  });
});

watch(filteredCountries, () => {
  currentPage.value = 1; // Reset pagination
  paginatedCountries.value = filteredCountries.value.slice(0, itemsPerPage); // Reset displayed items
});

// pagination
const currentPage = ref(1); // Tracks the current page
const itemsPerPage = 8; // Number of countries per page

// Function to load the next page
function loadMore() {
  const start = paginatedCountries.value.length; // Start after the currently loaded items
  const end = start + itemsPerPage; // Load the next set of items
  paginatedCountries.value = [
    ...paginatedCountries.value, // Keep the existing items
    ...filteredCountries.value.slice(start, end), // Add the new items
  ];
}
</script>

<style scoped>
.country_cards {
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
  padding-bottom: 2rem;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 1.5rem;
  justify-content: center;
}

.search_container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.input_wrapper {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
}

.search_input {
  width: 100%;
  font-size: var(--size-base);
  border: none;
  background-color: var(--card-bg);
  box-shadow: var(--bs);
  border-radius: var(--br);
  padding: 0.75rem 1.5rem;
}

.no_results-message {
  text-align: center;
  font-size: var(--size-sm);
  color: var(--clr-secondary-1);
}

/* ms lq */
@media (min-width: 1440px) {
  .container {
    flex-direction: row;
    justify-content: space-between;
    margin-bottom: 1.75rem;
  }

  .country_cards {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }

  .search_input {
    width: 400px;
  }
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 2rem;
}

.show-more-btn {
  padding: 0.75rem 1.5rem;
  font-size: var(--size-sm);
  background-color: var(--card-bg);
  color: var(--text-light);
  margin-top: 1rem;
  border-radius: var(--br);
  margin-bottom: 1.75rem;
  box-shadow: var(--bs);
}

.show-more-btn:hover {
  background-color: var(--primary-dark);
}
/*  */
</style>

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
        v-model="selectedRegion"
        @change="onRegionChange"
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
  <div v-if="filteredCountries.length > 0" class="country_cards">
    <CountryCard
      v-for="country in filteredCountries"
      :key="country.cca3"
      :country="country"
    />
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
function onRegionChange() {
  if (selectedRegion.value) {
    fetchCountriesByRegion(selectedRegion.value); // Fetch countries for selected region
  } else {
    fetchAllCountries(); // Fetch all countries if no region is selected
  }
}

// Watch for changes in the selected region and trigger an API call
watch(selectedRegion, (newRegion) => {
  if (newRegion) {
    fetchCountriesByRegion(newRegion); // Fetch countries for selected region
  } else {
    fetchAllCountries(); // Fetch all countries if no region is selected
  }
});

// Initially fetch all countries when the component is mounted
onMounted(() => {
  fetchAllCountries();
});

// Function to handle search execution (either by Enter key or search button)
function searchCountries() {
  // This function is here to handle the search when the user presses enter or clicks search
}

// Computed property to filter countries based on search query
const filteredCountries = computed(() => {
  if (!searchQuery.value) {
    return displayedCountries.value; // No search query, return all countries
  }
  const query = searchQuery.value.toLowerCase();
  return displayedCountries.value.filter((country) =>
    country.name.common.toLowerCase().includes(query)
  );
});
</script>

<style scoped>
.country_cards {
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
}

/* search styles */
/* .search_container {
  padding: 0 1rem;
  display: flex;
}

.search_input {
  width: 100%;
  padding: 0.5rem 1.5rem;
  font-size: var(--size-xxs);
  color: var(--secondary-1);
  border: none;
  box-shadow: var(--bs);
} */

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
  font-size: var(--size-xs);
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
  }

  .country_cards {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }

  .search_input {
    width: 400px;
  }
}
/*  */
</style>

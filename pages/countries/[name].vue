<template>
  <div class="country_details">
    <!-- Back Button -->
    <button class="back btn" @click="goBack">
      <Icon name="material-symbols:arrow-left-alt" class="back_btn-icon" /> Back
    </button>

    <!-- Country Details -->
    <div v-if="country" class="country_info">
      <!-- Flag -->
      <img
        :src="country.flags.svg"
        :alt="`${country.name.common} flag`"
        class="flag"
      />

      <!-- General Info -->
      <div class="info">
        <h1>{{ country.name.common }}</h1>

        <div class="info_main">
          <p>
            <strong>Population:</strong>
            {{ country.population.toLocaleString() }}
          </p>
          <p><strong>Region:</strong> {{ country.region }}</p>
          <p><strong>Subregion:</strong> {{ country.subregion }}</p>
          <p><strong>Capital:</strong> {{ country.capital?.join(", ") }}</p>
        </div>

        <!-- Additional Info -->
        <div class="info_add">
          <p>
            <strong>Top Level Domain:</strong> {{ country.tld?.join(", ") }}
          </p>
          <p><strong>Currencies:</strong> {{ currencies }}</p>
          <p><strong>Languages:</strong> {{ languages }}</p>
        </div>

        <!-- Border Countries -->
        <div v-if="country.bordersData" class="borders">
          Border Countries:
          <ul>
            <li
              class="btn btn-2"
              v-for="(name, code) in country.bordersData"
              :key="code"
            >
              {{ name }}
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Error Message -->
    <div v-else-if="error" class="error">
      <p>{{ error }}</p>
    </div>
  </div>
</template>

<script setup>
const route = useRoute();
const router = useRouter();
const country = ref(null);
const error = ref(null);

async function fetchBorderData(codes) {
  const apiUrl = `https://restcountries.com/v3.1/alpha?codes=${codes.join(
    ","
  )}`;
  const data = await $fetch(apiUrl);
  const bordersData = {};
  data.forEach((country) => {
    bordersData[country.cca3] = country.name.common;
  });
  return bordersData;
}

console.log(country.value?.borders); // Check if borders array exists

// async function fetchCountryByHandle(handle) {
//   try {
//     const apiUrl = `https://restcountries.com/v3.1/all`;
//     const countries = await $fetch(apiUrl);
//     const decodedHandle = decodeURIComponent(handle).replace(/-/g, " "); // Reverse the hyphenation
//     const matchedCountry = countries.find(
//       (c) => c.name.common.toLowerCase() === decodedHandle.toLowerCase()
//     );
//     if (matchedCountry) {
//       country.value = matchedCountry;
//     } else {
//       throw new Error("Country not found");
//     }
//   } catch (error) {
//     console.error("Error fetching country by handle:", error);
//   }
// }

async function fetchCountryByHandle(handle) {
  try {
    const apiUrl = `https://restcountries.com/v3.1/all`;
    const countries = await $fetch(apiUrl);
    const decodedHandle = decodeURIComponent(handle).replace(/-/g, " ");
    const matchedCountry = countries.find(
      (c) => c.name.common.toLowerCase() === decodedHandle.toLowerCase()
    );

    if (matchedCountry) {
      country.value = matchedCountry;

      // Fetch border country names if borders exist
      if (matchedCountry.borders?.length) {
        const bordersData = await fetchBorderData(matchedCountry.borders);
        country.value.bordersData = bordersData;
      }
    } else {
      throw new Error("Country not found");
    }
  } catch (error) {
    console.error("Error fetching country by handle:", error);
  }
}

onMounted(() => {
  fetchCountryByHandle(route.params.name);
});

// Computed properties for currencies and languages
const currencies = computed(() =>
  country.value?.currencies
    ? Object.values(country.value.currencies)
        .map((currency) => currency.name)
        .join(", ")
    : "N/A"
);

const languages = computed(() =>
  country.value?.languages
    ? Object.values(country.value.languages).join(", ")
    : "N/A"
);

// Navigation back to previous page
function goBack() {
  router.back();
}
</script>

<style scoped>
.country_details {
  padding: 2rem;
}

.flag {
  width: 100%;

  max-height: 300px;
  object-fit: contain;
  margin-bottom: 2rem;
}

.country_info {
  display: flex;
  flex-direction: column;
}

.info {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.info p {
  font-size: var(--size-sm);
  color: var(--clr-secondary-1);
  line-height: 2;
}

.borders ul {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  list-style: none;
  padding-top: 1rem;
}

.back {
  margin-bottom: 2rem;
}

.btn-2 {
  padding: 0.4rem 0.75rem;
}

/* mq lg */
@media (min-width: 1440px) {
  .country_info {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }

  .country_details {
    padding: 0;
  }

  .back {
    margin-bottom: 4rem;
  }

  .flag {
    max-height: 400px;

    object-fit: contain;
  }

  /* .info {
    flex-direction: row;
  } */
  .info {
    display: grid;
    grid-template-rows: auto 1fr auto; /* Three rows: h1, two divs, and the last div */
    grid-template-columns: 1fr 1fr; /* Two equal columns for the middle row */
    gap: 1.5rem; /* Space between items */
    width: 100%; /* Full width for layout consistency */
    padding: 2rem 5rem;
  }

  .info p {
    font-size: var(--size-base);
    line-height: 2.2;
  }

  /* Ensure the h1 spans the entire first row */
  .info h1 {
    grid-column: 1 / -1; /* Spans both columns */
    grid-row: 1; /* First row */
    font-size: var(--size-3xl);
  }

  /* The first div (info_main) occupies the first column of the second row */
  .info_main {
    grid-column: 1; /* First column */
    grid-row: 2; /* Second row */
  }

  /* The second div (info_add) occupies the second column of the second row */
  .info_add {
    grid-column: 2; /* Second column */
    grid-row: 2; /* Second row */
  }

  /* The third div (borders) spans the entire last row */
  .borders {
    grid-column: 1 / -1; /* Spans both columns */
    grid-row: 3; /* Third row */
    display: flex;
    gap: 1rem;
    align-items: center;
  }

  .borders ul {
    padding-top: unset;
  }

  .back_btn-icon {
    font-size: var(--size-2xl);
  }
}
</style>

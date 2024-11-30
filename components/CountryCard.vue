<template>
  <div class="country_card">
    <NuxtLink
      :to="`/countries/${formatHandle(country.name.common)}`"
      class="country_link"
    >
      <img
        :src="country.flags.png"
        alt="Flag of {{ country.name.common }}"
        class="country_flag"
        width="280px"
        height="150px"
      />
      <div class="country_card-info">
        <h3>{{ country.name.common }}</h3>
        <p>
          <strong>Population:</strong> {{ country.population.toLocaleString() }}
        </p>
        <p><strong>Region:</strong> {{ country.region }}</p>
        <p>
          <strong>Capital:</strong>
          {{ country.capital ? country.capital[0] : "N/A" }}
        </p>
      </div>
    </NuxtLink>
  </div>
</template>

<script setup>
defineProps({
  country: {
    type: Object,
    required: true,
  },
});

function formatHandle(name) {
  return name.toLowerCase().replace(/\s+/g, "-"); // Replace spaces with hyphens and convert to lowercase
}
</script>

<style scoped>
.country_card {
  background: var(--card-bg);
  display: flex;
  flex-direction: column;
  box-shadow: var(--bs);
  max-width: 280px;
  margin: 0 auto;
  overflow: hidden; /* This will clip child elements (like the flag) to fit inside the rounded corners */
  border-radius: var(--br);
}

.country_flag {
  width: 100%;
  /* object-fit: cover; */
  width: 100%;
  width: 280px;
  object-fit: cover;
  height: 150px;
}

.country_card h3 {
  padding-bottom: 1rem;
}
.country_card-info {
  padding: 1.5rem;
  padding-bottom: 2.5rem;
}

.country_card-info p {
  color: var(--clr-secondary-1);
  font-size: var(--size-sm);
}
</style>

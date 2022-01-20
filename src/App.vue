<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
const countries = ref([]);
const filteredCountries = ref([]);
const capitalSearchText = ref("");
const searchText = ref("");
const loading = ref(true);
const onKeyupAll = () => {
  let data;

  if (!searchText.value.length) {
    data = countries.value;
  } else {
    data = countries.value.filter((item) => {
      let arr = JSON.parse(JSON.stringify(item));
      if (!item.capital) item.capital = "";
      return Object.keys(arr).some((key) => {
        if (typeof item[key] === "string") {
          return item[key]
            .toString()
            .toLocaleLowerCase("tr-TR")
            .includes(searchText.value.toLocaleLowerCase("tr-TR"));
        }
        if (typeof item[key] === "number") {
          return item[key].toString().includes(searchText.value);
        }
        if (typeof item[key] == "object") {
          return Object.values(
            JSON.parse(JSON.stringify(Object.values(item[key])))[0]
          ).includes(searchText.value);
        }
        if (Array.isArray(item[key])) {
          return JSON.parse(JSON.stringify(item[key])).includes(
            searchText.value
          );
        }
      });
    });
  }
  filteredCountries.value = data;
  console.log(
    "filtered data : ",
    JSON.parse(JSON.stringify(filteredCountries.value))
  );
};
const onKeyupCapital = () => {
  let data;
  if (!capitalSearchText.value.length) {
    data = countries.value;
  } else {
    data = countries.value.filter((item) => {
      if (!item.capital) item.capital = "";
      return item.capital
        .toLocaleLowerCase("tr-TR")
        .includes(capitalSearchText.value.toLocaleLowerCase("tr-TR"));
    });
  }
  filteredCountries.value = data;
};
onMounted(async () => {
  await axios
    .get("https://restcountries.com/v2/all")
    .then((res) => {
      countries.value = res.data;
      loading.value = false;
    })
    .catch((e) => console.error(e))
    .finally(() => {
      loading.value = false;
    });
  filteredCountries.value = countries.value;
});
</script>

<template>
  <div
    class="p-5 bg-dark h-full d-flex flex-column align-items-center justify-content-center"
  >
    <h3 class="text-white display-4 mb-5">
      COUNTRIES ({{ filteredCountries.length }})
    </h3>
    <div class="d-flex justify-content-start w-50 mb-4">
      <input
        type="text"
        placeholder="Başkent ara..."
        v-model="capitalSearchText"
        @keyup="onKeyupCapital"
        class="form-control d-inline"
      />
      <input
        type="text"
        placeholder="Tüm Alanda Ara..."
        v-model="searchText"
        @keyup="onKeyupAll"
        class="form-control d-inline ms-4"
      />
    </div>
    <table
      v-if="!loading"
      class="table table-hover table-light table-striped w-50 rounded"
    >
      <thead class="bg-white">
        <th>Name</th>
        <th>Capital</th>
        <th>Region</th>
        <th>Flag</th>
      </thead>
      <tbody>
        <tr v-for="country in filteredCountries" :key="country.name">
          <td>{{ country.name }}</td>
          <td>{{ country.capital }}</td>
          <td>{{ country.region }}</td>
          <td>
            <img
              class="img-fluid rounded-circle"
              style="width: 30px; height: 30px"
              :src="country.flag"
              :alt="country.name"
            />
          </td>
        </tr>
      </tbody>
    </table>
    <div class="w-full d-flex justify-content-center py-5" v-else>
      <div class="spinner-grow text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>
  </div>
</template>
<style></style>

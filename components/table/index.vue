<template>
  <div>
    <div class="filters">
      <input
        v-for="(key, index) in filterKeys"
        :key="index"
        v-model="filters[key]"
        @input="updateFilters"
        :placeholder="capitalizeFirstLetter(key) + ' Filter'"
      />
    </div>
    <table class="table">
      <thead>
      <tr>
        <th
          v-for="(header, index) in tableHeaders"
          :key="index"
          @click="header.sortable ? sortData(header.name) : null"
          :class="{ 'sortable': header.sortable && header.name === sortBy }"
        >
          <nuxt-link
            v-if="header.sortable"
            class="link"
            :to="generateSortLink(header.name)"
          >
            {{ header.name }}
          </nuxt-link>
          <span v-else>{{ header.name }}</span>
        </th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(item, index) in paginatedData" :key="index">
        <td v-for="(value, key) in item" :key="key">{{ value }}</td>
      </tr>
      </tbody>
    </table>

    <div class="pagination">
      <button @click="nextPage" :disabled="currentPage === totalPages" class="pagination-button">بعدی</button>
      <span class="pagination-info">صفحه {{ currentPage }} از {{ totalPages }}</span>
      <button @click="previousPage" :disabled="currentPage === 1" class="pagination-button">قبلی</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: Array,
    itemsPerPage: Number,
    tableHeaders: Array, // Modify this to include sortable property
    filterKeys: Array,
  },
  data() {
    const filterDefaults = {};
    this.filterKeys.forEach(key => {
      filterDefaults[key] = this.$route.query[key] || '';
    });

    return {
      filters: filterDefaults,
      currentPage: 1,
      sortBy: this.$route.query.sortBy || null,
      sortDesc: this.$route.query.sortDesc === 'true' || false,
    };
  },
  watch: {
    $route(to, from) {
      const query = to.query;
      const newFilters = {};
      this.filterKeys.forEach(key => {
        newFilters[key] = query[key] || '';
      });

      this.filters = newFilters;
      this.sortBy = query.sortBy || null;
      this.sortDesc = query.sortDesc === 'true' || false;
    },
  },
  computed: {
    paginatedData() {
      const { currentPage, itemsPerPage, sortBy, sortDesc } = this;
      const startIndex = (currentPage - 1) * itemsPerPage;
      const endIndex = startIndex + itemsPerPage;

      let filteredData = this.data
        .filter(item => (
          item.name.toLowerCase().includes(this.filters.name.toLowerCase()) &&
          item.date.toLowerCase().includes(this.filters.date.toLowerCase()) &&
          item.title.toLowerCase().includes(this.filters.title.toLowerCase())
        ));

      let sortedData = [...filteredData];

      if (sortBy) {
        sortedData.sort((a, b) => {
          const valA = a[sortBy];
          const valB = b[sortBy];
          return sortDesc ? valB.localeCompare(valA) : valA.localeCompare(valB);
        });
      }

      return sortedData.slice(startIndex, endIndex);
    },
    totalPages() {
      return Math.ceil(this.data.length / this.itemsPerPage);
    },
  },
  methods: {
    updateFilters() {
      const query = { ...this.filters, sortBy: this.sortBy, sortDesc: this.sortDesc };
      this.$router.push({ query });
    },
    capitalizeFirstLetter(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    sortData(header) {
      const headerInfo = this.tableHeaders.find(h => h.name === header);
      if (headerInfo && headerInfo.sortable) {
        if (this.sortBy === header) {
          this.sortDesc = !this.sortDesc;
        } else {
          this.sortBy = header;
          this.sortDesc = false;
        }
      }
    },
    generateSortLink(header) {
      return {
        query: {
          sortBy: header,
          sortDesc: this.sortBy === header ? !this.sortDesc : false,
        },
      };
    },
  },
};
</script>

<style scoped>
.link {
  text-decoration: none;
}
table {
  margin: auto;
  border-collapse: collapse;
  font-family: Tahoma, Geneva, sans-serif;
}
table td {
  padding: 15px;
}
table thead th {
  background-color: #54585d;
  color: #ffffff;
  padding: 20px;
  font-weight: bold;
  font-size: 13px;
  border: 1px solid #54585d;
}
table tbody td {
  color: #636363;
  border: 1px solid #dddfe1;
}
table tbody tr {
  background-color: #f9fafb;
}
table tbody tr:nth-child(odd) {
  background-color: #ffffff;
}
/* Add CSS styles for the input fields */
input {
  border: 1px solid #54585d;
  border-radius: 10px;
  margin-right: 20px;
  padding: 15px;
}

/* Add styles for the pagination buttons */
.pagination {
  margin: 50px auto;
  text-align: center;
}
.pagination-button {
  background-color: #54585d;
  color: white;
  padding: 5px 10px;
  border: none;
  cursor: pointer;
}

.pagination-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
.filters {
  margin: 50px 0;
  text-align: center;
}
</style>

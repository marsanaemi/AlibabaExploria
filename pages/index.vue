<template>
  <div>
    <my-table
      :data="jsonData"
      :itemsPerPage="10"
      :tableHeaders="tableHeaders"
      :filter-keys="tableFilters"
    />
  </div>
</template>
<script>
import MyTable from "~/components/table/index.vue";
import axios from "axios";

export default {
  name: 'Index',
  async asyncData() {
    const response = await axios.get('/records.json');
    const jsonData = response.data;
    return {
      jsonData,
    };
  },
  components: {
    MyTable
  },
  data() {
    return {
      tableHeaders: [
        {
          name: 'id',
          sortable: false,
        },
        {
          name: 'name',
          sortable: false,
        },
        {
          name:'date' ,
          sortable: true,
        },
        {
          name:  'title',
          sortable: false,
        },
        {
          name:'field' ,
          sortable: false,
        },
        {
          name:'old_value' ,
          sortable: false,
        },
        {
          name: 'new_value' ,
          sortable: false,
        },
      ],
      tableFilters: ['name', 'date', 'title']
    }
  }
}
</script>

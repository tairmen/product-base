<template>
  <div class="main">
    <h3>Products Table</h3>
    <b-table
      class="products-table"
      striped
      hover
      :items="items"
      :fields="fields"
    ></b-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fields: [
        {
          key: "name",
          sortable: true,
        },
        {
          key: "cost",
          sortable: true,
        },
        {
          key: "amount",
          sortable: true,
        },
        {
          key: "code",
          sortable: false,
        },
      ],
      items: JSON.parse(localStorage.products),
    };
  },
  components: {},
};
</script>

<style scoped>
.main {
  margin: 10px;
}
.products-table {
  margin: 10px;
}
</style>

<template>
  <div id="app">
    <b-container >
      <b-row>
        <b-col sm="6">
          <b-row class="user-panel"><p style="margin: auto">User: <b> Admin </b></p></b-row>
          <b-row class="pr-panel"><ProductPanel :products="products" @products-change="pChange" /></b-row>
        </b-col>
        <b-col class="pr-table" sm="6"><ProductList :items="products" /></b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import ProductPanel from "./components/product-panel.vue";
import ProductList from "./components/product-list.vue";

export default {
  name: "App",
  data() {
    return {
      products: [],
    }
  },
  created() {
    let me = this;
    if (!localStorage.products) {
      localStorage.products = JSON.stringify([]);
    }
    me.products = JSON.parse(localStorage.products);
  },
  components: {
    ProductPanel,
    ProductList,
  },
  methods: {
    pChange(products) {
      let me = this;
      me.products = products;
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 30px;
}
.user-panel {
  min-height: 40px;
  font-size: 20px;
}
</style>

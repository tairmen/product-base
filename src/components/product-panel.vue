<template>
  <div>
    <b-container class="buttons-panel">
      <b-row class="my-1">
        <b-col sm="4">
          <b-button class="bar-item" @click="addProduct" variant="outline-secondary">Add</b-button>
        </b-col>
        <b-col sm="8">
          <b-form-file
            @change="fileChanged"
            class="bar-item"
            accept="image/*"
          ></b-form-file>
        </b-col>
      </b-row>
      <b-row class="my-1">
        <b-col sm="4">
          <b-button
            class="bar-item"
            variant="outline-secondary"
            @click="runClicked"
            :disabled="!Boolean(file)"
            >Scan</b-button
          >
        </b-col>
        <b-col sm="8">
          <b-dropdown class="bar-item" text="Options">
            <b-form-select
              v-model="resolution"
              :options="resolutionOptions"
            ></b-form-select>
            <b-form-select
              v-model="patchSize"
              :options="patchSizeOptions"
            ></b-form-select>
          </b-dropdown>
        </b-col>
      </b-row>
    </b-container>
    <div>
    <b-container class="forms-panel" fluid="sm">
      <div v-if="!loading && loaded">
        <b-row>
          <b-col sm="12">
            <label><b>{{ formTitle }}</b></label>
          </b-col>
        </b-row>
        <b-row>
          <b-col sm="3">
            <label>Code:</label>
          </b-col>
          <b-col sm="9">
            <b-form-input
              v-model="code"
              :disabled="formTitle != 'Add'"
              type="number"
              min="1000000000000" 
              max="9999999999999"
              placeholder="Code"
            ></b-form-input>
          </b-col>
        </b-row>
        <b-row>
          <b-col sm="3">
            <label>Name</label>
          </b-col>
          <b-col sm="9">
            <b-form-input v-model="pname" placeholder="Name"></b-form-input>
          </b-col>
        </b-row>
        <b-row>
          <b-col sm="3">
            <label>Cost:</label>
          </b-col>
          <b-col sm="9">
            <b-form-input v-model="cost" placeholder="Cost"></b-form-input>
          </b-col>
        </b-row>
        <b-row>
          <b-col sm="3">
            <label>Amount:</label>
          </b-col>
          <b-col sm="9">
            <b-form-input v-model="amount" placeholder="Amount"></b-form-input>
          </b-col>
        </b-row>
        <b-row>
          <b-button
            class="save-button"
            variant="outline-primary"
            :disabled="!cost || !amount || !pname || !code"
            @click="saveProduct"
            >Save</b-button
          >
        </b-row>
      </div>
      <div v-if="loading && !loaded" class="text-center">
        <b-spinner label="Spinning"></b-spinner>
      </div>
      <div v-if="!loading && !loaded" class="error-box">{{ errorText }}</div>
    </b-container>
    </div>
  </div>
</template>

<script>
import Quagga from "quagga";
import $ from "jquery";

export default {
  props: {
    products: null,
  },
  data() {
    return {
      state: {
        inputStream: {
          size: 640,
          singleChannel: false,
        },
        locator: {
          patchSize: "small",
          halfSample: true,
        },
        decoder: {
          readers: [
            {
              format: "ean_reader",
              config: {},
            },
          ],
        },
        locate: true,
        src: null,
      },
      file: null,
      resolution: "640",
      errorText: "",
      resolutionOptions: [
        { value: "320", text: "320px" },
        { value: "640", text: "640px" },
        { value: "800", text: "800px" },
        { value: "1280", text: "1280px" },
        { value: "1600", text: "1600px" },
        { value: "1920", text: "1920px" },
      ],
      patchSize: "small",
      patchSizeOptions: [
        { value: "x-small", text: "x-small" },
        { value: "small", text: "small" },
        { value: "medium", text: "medium" },
        { value: "large", text: "large" },
        { value: "x-large", text: "x-large" },
      ],
      loading: false,
      loaded: false,
      formTitle: "",
      editedIndex: -1,
      code: null,
      pname: null,
      cost: null,
      amount: null,
    };
  },
  mounted() {
    let me = this;
    Quagga.onDetected(function(result) {
      me.loaded = true;
      me.loading = false;
      me.code = result.codeResult.code;
      me.editedIndex = me.products.findIndex(el => {
        return el.code == me.code;
      });
      if (me.editedIndex > -1) {
        me.formTitle = "Finded";
        me.pname = me.products[me.editedIndex].name;
        me.amount = me.products[me.editedIndex].amount;
        me.cost = me.products[me.editedIndex].cost;
      } else {
        me.formTitle = "New";
      }
    });
  },
  methods: {
    fileChanged(e) {
      let me = this;
      if (e.target.files && e.target.files.length) {
        me.file = e.target.files[0];
      }
    },
    runClicked() {
      let me = this;
      me.code = 0;
      me.pname = "";
      me.cost = "";
      me.amount = "";
      me.loaded = false;
      if (me.file) {
        me.loading = true;
        setTimeout(() => {
          if (!me.loaded) {
            me.loading = false;
            me.errorText = "Can't read barcode";
          }
        }, 3000);
        me.decode(URL.createObjectURL(me.file));
      }
    },
    addProduct() {
      let self = this;
      self.editedIndex = -1;
      self.formTitle = "Add";
      self.loaded = true;
      self.loading = false;
      self.code = 0;
      self.pname = "";
      self.amount = "";
      self.cost = "";
    },
    decode(src) {
      let self = this;
      self.setState();
      let config = $.extend({}, self.state, { src: src });
      Quagga.decodeSingle(config, function(result) {});
    },
    setState() {
      let self = this;
      self.state.inputStream.size = parseInt(self.resolution);
      self.state.locator.patchSize = self.patchSize;
    },
    saveProduct() {
      let self = this;
      let res = {};
      res.code = self.code;
      res.name = self.pname;
      res.amount = self.amount;
      res.cost = self.cost;
      let allItems = JSON.parse(localStorage.products);
      if (self.editedIndex == -1) {
        allItems.push(res);
      } else {
        allItems[self.editedIndex] = res;
      }
      localStorage.products = JSON.stringify(allItems);
      self.$emit("products-change", allItems);
      self.errorText = "";
      self.loaded = false;
      self.loading = false;
    },
  },
};
</script>

<style scoped>
.buttons-panel {
  margin: 20px auto;
  max-width: 600px;
}
.bar-item {
  margin: 5px 10px;
}
.bar-item .custom-select {
  margin: 4px 0px;
}
label {
  margin-top: 8px;
}
.forms-panel {
  border: 1px solid gray;
  margin: 10px;
  border-radius: 10px;
  padding: 20px;
}
.error-box {
  height: 200px;
  color: darkred;
  font-size: 18px;
}
.save-button {
  margin-top: 8px;
  margin-left: auto;
  margin-right: 20px;
}
</style>

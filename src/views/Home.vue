<template>
  <v-container>
    <v-row v-if="loading && !error">
      <v-col
        v-for="i in [1, 2, 3, 4, 5, 6, 7, 8]"
        :key="i"
        cols="6"
        md="4"
        lg="3"
      >
        <v-skeleton-loader type="article, actions"></v-skeleton-loader>
      </v-col>
    </v-row>
    <v-alert
      v-if="!loading && error"
      border="top"
      colored-border
      type="error"
      elevation="2"
    >
      Can not get products!
    </v-alert>

    <FilterCategory @change="onCategoryChange" />
    <div v-if="!loading && !error && products.length > 0">
      <v-row align="stretch">
        <v-col
          v-for="product in products"
          :key="product.ID"
          :id="product.ID"
          cols="6"
          md="4"
          lg="3"
        >
          <v-card>
            <v-card-title class="product-title">
              {{ product.Name }}
            </v-card-title>
            <v-card-subtitle class="product-description">
              {{ product.Description }}&nbsp;
            </v-card-subtitle>

            <v-card-actions class="d-block">
              <v-row>
                <v-col v-if="product.Ratings > 0">
                  <v-icon>fas fa-star</v-icon>
                  {{ product.Ratings }}
                </v-col>
                <v-col class="product-price text-right">
                  Rp <b>{{ currencyFormat(product.Price) }}</b>
                </v-col>
              </v-row>

              <div class="mt-3">
                <v-btn
                  class="button-beli"
                  small
                  outlined
                  color="primary"
                  @click="gotoProductDetail(product)"
                  >Detail</v-btn
                >

                <v-btn
                  class="button-beli ml-3"
                  small
                  outlined
                  color="primary"
                  @click="setOrders(product)"
                  >Beli</v-btn
                >
              </div>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </div>
    <v-alert
      v-if="!loading && !error && products.length <= 0"
      border="top"
      colored-border
      type="info"
      elevation="2"
    >
      Products is empty!
    </v-alert>
  </v-container>
</template>

<script>
import { get, call, dispatch } from "vuex-pathify";
import FilterCategory from "@/components/FilterCategory.vue";

export default {
  components: {
    FilterCategory,
  },
  computed: {
    products: get("product/products"),
    loading: get("product/loading"),
    error: get("product/error"),
    orders: get("cart/orders"),
  },

  mounted() {
    this.getAllProducts();
  },

  methods: {
    getAllProducts: call("product/getAllProducts"),
    async onCategoryChange(id) {
      await this.getAllProducts(id);
    },
    setOrders(product) {
      let orders = [];
      if (this.orders.map((order) => order.ID).includes(product.ID)) {
        orders = this.orders.map((order) => {
          if (order.ID == product.ID) {
            order.qty++;
          }
          return order;
        });
      } else {
        orders = [
          ...this.orders,
          {
            ...product,
            qty: 1,
          },
        ];
      }

      dispatch("cart/setOrders", orders);
    },
    gotoProductDetail(product) {
      this.$router.push({ name: "Products", params: { id: product.ID } });
    },
    currencyFormat(val) {
      return new Intl.NumberFormat("id-ID", {
        maximumFractionDigits: 0,
      }).format(val);
    },
  },
};
</script>

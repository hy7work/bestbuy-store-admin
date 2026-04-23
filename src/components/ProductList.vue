<template>
  <div>
    <div class="header">
      <h2>Manage Products project demo for CI/CD on store admin</h2>
      <router-link to="/product/add">
        <button class="add-btn">+ Add Product</button>
      </router-link>
    </div>

    <div class="product-list">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Product</th>
            <th>Description</th>
            <th>Price ($)</th>
          </tr>
        </thead>

        <tr v-for="product in products" :key="product.id">
          <td>
            <router-link :to="`/product/${product.id}`">
              {{ product.id }}
            </router-link>
          </td>
          <td>{{ product.name }}</td>
          <td>{{ product.description }}</td>
          <td class="price">{{ formatPrice(product.price) }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProductList',
  props: ['products'],

  mounted() {
    this.$emit('getProducts')
  },

  methods: {
    formatPrice(price) {
      return Number(price).toFixed(2)
    }
  }
}
</script>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.add-btn {
  background-color: #0046be;
  color: white;
  border: none;
  padding: 8px 16px;
  cursor: pointer;
  border-radius: 4px;
}

.add-btn:hover {
  background-color: #003399;
}

.product-list {
  text-align: left;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

th {
  background-color: #0046be;
  color: white;
  padding: 10px;
}

td {
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

tr:hover {
  background-color: #f5f5f5;
}

.price {
  font-weight: bold;
  color: #222;
}

a {
  color: #0046be;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
</style>
<template>
  <div>
    <div class="action-bar">
      <router-link :to="`/product/${$route.params.id}/edit`">
        <button class="edit-btn">Edit Product</button>
      </router-link>
    </div>

    <div class="product-detail" v-if="productExists">
      <div class="product-image">
        <img :src="product.image" alt="Product Image">
      </div>

      <div class="product-info">
        <h2>{{ product.name }}</h2>
        <p class="price">${{ formatPrice(product.price) }}</p>
        <small class="product-id">Product ID: {{ product.id }}</small>
        <p class="description">{{ product.description }}</p>
      </div>
    </div>

    <div class="product-detail" v-else>
      <h3>Product not found</h3>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProductDetail',
  props: ['products'],
  computed: {
    product() {
      return this.products.find(product => product.id == this.$route.params.id)
    },
    productExists() {
      return !!this.product
    }
  },
  methods: {
    formatPrice(price) {
      return Number(price).toFixed(2)
    }
  }
}
</script>

<style scoped>
.action-bar {
  margin-bottom: 20px;
}

.edit-btn {
  background-color: #0046be;
  color: white;
  border: none;
  padding: 8px 16px;
  cursor: pointer;
  border-radius: 4px;
}

.edit-btn:hover {
  background-color: #003399;
}

.product-detail {
  display: flex;
  gap: 24px;
  align-items: flex-start;
}

.product-image {
  flex: 1;
  max-width: 320px;
}

.product-image img {
  width: 100%;
  height: auto;
  border: 1px solid #ddd;
  border-radius: 6px;
  background: #fff;
}

.product-info {
  flex: 2;
  text-align: left;
}

.product-info h2 {
  font-size: 28px;
  margin-bottom: 10px;
}

.price {
  font-size: 22px;
  font-weight: bold;
  color: #0046be;
  margin-bottom: 10px;
}

.product-id {
  display: block;
  margin-bottom: 16px;
  color: #666;
}

.description {
  font-size: 16px;
  line-height: 1.5;
}

a {
  color: #0046be;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .product-detail {
    flex-direction: column;
  }
}
</style>
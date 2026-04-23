<template>
  <TopNav />
  <router-view
    :orders="orders"
    :products="products"
    @fetchOrders="fetchOrders"
    @completeOrder="completeOrder"
    @addProductsToList="addProductsToList"
    @updateProductInList="updateProductInList"
    @getProduct="getProduct"
    @getProducts="getProducts"
  ></router-view>
</template>

<script>
import TopNav from './components/TopNav.vue'

const productServiceUrl = '/products/'
const singleProductServiceUrl = '/product/'
const makelineServiceUrl = '/makeline/'

export default {
  name: 'App',
  components: {
    TopNav
  },
  data() {
    return {
      orders: [],
      products: [],
      product: {}
    }
  },
  mounted() {
    this.getProducts()
  },
  methods: {
    addProductsToList(newProduct) {
      this.products.push(newProduct)
    },

    updateProductInList(updatedProduct) {
      const index = this.products.findIndex(product => product.id === updatedProduct.id)
      if (index !== -1) {
        this.products[index] = updatedProduct
      }
    },

    async getProduct(id) {
      try {
        const response = await fetch(`${singleProductServiceUrl}${id}`)
        const product = await response.json()

        this.product.id = product.id
        this.product.name = product.name
        this.product.image = product.image
        this.product.description = product.description
        this.product.price = product.price
      } catch (error) {
        console.log(error)
        alert('Error occurred while fetching product')
      }
    },

    async getProducts() {
      try {
        const response = await fetch(productServiceUrl)
        const products = await response.json()
        this.products = products || []
      } catch (error) {
        console.log(error)
        alert('Error occurred while fetching products')
      }
    },

    async fetchOrders() {
      try {
        const response = await fetch('/makeline/order/fetch')
        const data = await response.json()

        console.log('Orders from API:', data)
        this.orders = data || []
      } catch (error) {
        console.error('Failed to fetch orders:', error)
        this.orders = []
      }
    },

    async completeOrder(orderId) {
      const order = this.orders.find(order => order.orderId === orderId)
      if (!order) return

      order.status = 1

      try {
        const response = await fetch(`${makelineServiceUrl}order`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(order)
        })

        if (!response.ok) {
          alert('Error occurred while processing order')
          return
        }

        alert('Order successfully processed')
        this.orders = this.orders.filter(order => order.orderId !== orderId)
        this.$router.go(-1)
      } catch (error) {
        console.log(error)
        alert('Error occurred while processing order')
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #1f1f1f;
  margin-top: 110px;
  padding: 1rem 1.5rem 2rem;
  background-color: #f5f6f8;
  min-height: 100vh;
  box-sizing: border-box;
}

table {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  background: #fff;
  border-radius: 8px;
  overflow: hidden;
}

th,
td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #0046be;
  color: white;
}

button {
  padding: 10px 14px;
  background-color: #0046be;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  min-height: 42px;
  font-weight: bold;
}

button:hover {
  background-color: #003399;
}

.action-button {
  float: right;
}

.order-detail {
  text-align: left;
}

.product-detail {
  text-align: left;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  gap: 1rem;
  margin: 2rem auto;
}

.product-form {
  display: flex;
  flex-direction: column;
  align-items: left;
  justify-content: center;
  margin: 2rem auto;
  width: 50%;
}

.form-row {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.ai-button {
  margin-left: 10px;
  padding: 10px;
  border-radius: 5px;
  border: none;
  background-color: #ffde00;
  color: #111;
  cursor: pointer;
  font-weight: bold;
}

.ai-button:hover {
  background-color: #f5d300;
}

textarea,
input {
  width: 100%;
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ccc;
  box-sizing: border-box;
}

label {
  text-align: right;
  margin-right: 10px;
  width: 100px;
  font-weight: bold;
}
</style>

<template>
  <div class="product-form-page">
    <div class="action-bar">
      <button @click="saveProduct" class="save-btn">Save Product</button>
    </div>

    <div v-if="showValidationErrors" class="error-box">
      <ul>
        <li v-for="error in validationErrors" :key="error">{{ error }}</li>
      </ul>
    </div>

    <div class="product-form">
      <table>
        <tr>
          <td><label for="product-name">Name</label></td>
          <td>
            <input
              id="product-name"
              placeholder="Product Name"
              v-model="product.name"
            />
          </td>
          <td></td>
        </tr>

        <tr>
          <td><label for="product-price">Price</label></td>
          <td>
            <input
              id="product-price"
              placeholder="Product Price"
              v-model="product.price"
              type="number"
              step="0.01"
            />
          </td>
          <td></td>
        </tr>

        <tr>
          <td><label for="product-tags">Keywords</label></td>
          <td>
            <input
              id="product-tags"
              placeholder="Product Keywords"
              v-model="product.tags"
            />
          </td>
          <td></td>
        </tr>

        <tr>
          <td><label for="product-description">Description</label></td>
          <td>
            <textarea
              rows="8"
              id="product-description"
              placeholder="Product Description"
              v-model="product.description"
            ></textarea>
            <input
              type="hidden"
              id="product-id"
              placeholder="Product ID"
              v-model="product.id"
            />
          </td>
          <td>
            <button
              @click="generateDescription"
              class="ai-button"
              v-show="aiCapabilities.includes('description')"
            >
              Ask AI Assistant
            </button>
          </td>
        </tr>

        <tr>
          <td><label for="product-image">Image</label></td>
          <td>
            <input
              id="product-image-text"
              placeholder="Product Image"
              v-model="product.image"
              v-show="!aiCapabilities.includes('image')"
            />

            <div
              id="product-image-container"
              class="image-container"
              :class="{ loading: isLoadingImage }"
              v-show="aiCapabilities.includes('image')"
            >
              <img v-if="product.image" :src="product.image" alt="Product Image" />
              <div class="overlay">{{ overlayText }}</div>
            </div>
          </td>
          <td>
            <button
              id="product-image-btn"
              @click="generateImage"
              class="ai-button"
              v-show="aiCapabilities.includes('image')"
            >
              Generate Image
            </button>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
const aiServiceUrl = '/ai/'
const productServiceUrl = '/product/'

export default {
  name: 'ProductForm',
  props: ['products'],
  emits: ['addProductsToList', 'updateProductInList'],
  data() {
    return {
      product: {
        id: 0,
        name: '',
        image: '/placeholder.png',
        description: '',
        price: 0.0,
        tags: []
      },
      aiCapabilities: [],
      showValidationErrors: false,
      isLoadingImage: false,
      overlayText: ''
    }
  },
  mounted() {
    if (this.$route.params.id) {
      const product = this.products.find(
        product => product.id == this.$route.params.id
      )
      this.product = Object.assign({}, product)

      if (!this.product.tags) {
        this.product.tags = []
      }
    }

    fetch(`${aiServiceUrl}health`)
      .then(response => response.json())
      .then(data => {
        if (data.status === 'ok') {
          this.aiCapabilities = data.capabilities
        }
      })
      .catch(error => {
        console.log('error occurred when evaluating ai service health')
        console.log(error)
      })
  },
  computed: {
    validationErrors() {
      const errors = []

      if (this.product.name.length === 0) {
        errors.push('Please enter a value for the name field')
      }

      if (this.product.description.length === 0) {
        errors.push('Please enter a value for the description field')
      }

      if (this.product.price <= 0) {
        errors.push('Please enter a value greater than 0 for the price field')
      }

      return errors
    }
  },
  methods: {
    generateDescription() {
      if (this.product.tags.length === 0) {
        alert('Please enter a value for the keywords field')
        return
      }

      const intervalId = this.waitForAI()

      const requestBody = {
        name: this.product.name,
        tags: this.product.tags.split(',').map(tag => tag.trim())
      }

      this.product.description = ''

      fetch(`${aiServiceUrl}generate/description`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(requestBody)
      })
        .then(response => response.json())
        .then(product => {
          this.product.description = product.description
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while generating product description')
        })
        .finally(() => {
          clearInterval(intervalId)
        })
    },

    generateImage() {
      if (this.product.description === '') {
        alert('Please enter a product description')
        return
      }

      this.isLoadingImage = true
      this.overlayText = 'Generating image...'

      const requestBody = {
        name: this.product.name,
        description: this.product.description
      }

      fetch(`${aiServiceUrl}generate/image`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(requestBody)
      })
        .then(response => response.json())
        .then(product => {
          this.overlayText = 'Downloading...'
          this.product.image = ''
          this.product.image = product.image
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while generating product image')
        })
        .finally(() => {
          this.isLoadingImage = false
        })
    },

    waitForAI() {
      let dots = ''
      const intervalId = setInterval(() => {
        dots += '.'
        this.product.description = `Thinking${dots}`
      }, 500)
      return intervalId
    },

    saveProduct() {
      if (this.validationErrors.length > 0) {
        this.showValidationErrors = true
        return
      }

      let method = 'PUT'
      const path = this.$route.path

      if (path.includes('add')) {
        method = 'POST'
      }

      this.product.price = parseFloat(this.product.price)

      fetch(`${productServiceUrl}`, {
        method: method,
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(this.product)
      })
        .then(response => response.json())
        .then(product => {
          alert('Product saved successfully')

          if (method === 'PUT') {
            this.$emit('updateProductInList', this.product)
          } else {
            this.$emit('addProductsToList', product)
          }

          this.$router.push(`/product/${product.id}`)
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while saving product')
        })
    }
  }
}
</script>

<style scoped>
.product-form-page {
  max-width: 1000px;
  margin: 0 auto;
}

.action-bar {
  margin-bottom: 20px;
}

.save-btn {
  background-color: #0046be;
  color: white;
  border: none;
  padding: 10px 18px;
  cursor: pointer;
  border-radius: 4px;
  font-weight: bold;
}

.save-btn:hover {
  background-color: #003399;
}

.error-box {
  background-color: #fff3f3;
  border: 1px solid #ffb3b3;
  color: #b30000;
  padding: 12px 16px;
  margin-bottom: 20px;
  border-radius: 6px;
}

.error-box ul {
  list-style: disc;
  margin: 0;
  padding-left: 20px;
}

img {
  width: 100%;
  border-radius: 6px;
  border: 1px solid #ddd;
}

table {
  border-collapse: collapse;
  width: 100%;
}

td {
  vertical-align: top;
  border: none;
  padding: 10px;
}

label {
  font-weight: bold;
}

.product-form {
  display: flex;
  justify-content: center;
  background: #fff;
  padding: 20px;
  border-radius: 8px;
}

.product-form input,
.product-form textarea {
  width: 100%;
  padding: 8px 10px;
  margin: 4px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: inherit;
}

.ai-button {
  background-color: #ffde00;
  color: #111;
  border: none;
  min-width: 140px;
  height: 44px;
  padding: 0 12px;
  cursor: pointer;
  border-radius: 4px;
  font-weight: bold;
}

.ai-button:hover {
  background-color: #f5d300;
}

.image-container {
  position: relative;
  width: 100%;
  min-height: 180px;
  display: flex;
  align-items: center;
}

.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  font-size: 1.2rem;
  font-weight: bold;
  border-radius: 6px;
}

.image-container.loading .overlay {
  opacity: 1;
}
</style>
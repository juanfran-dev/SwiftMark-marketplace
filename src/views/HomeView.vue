<template>
  <div class="flex flex-col">
    <NavbarComponent />
    <div class="flex flex-col min-h-screen w-full items-center justify-start">
      <!-- filtres -->
      <div class="flex w-full justify-start px-4 py-3">
        <div class="flex w-full max-w-6xl flex-wrap items-center justify-center gap-x-8 gap-y-3">
          <span>Filters:</span>
          <div class="flex items-center gap-2">
            <label>Category:</label>
            <select v-model="selectedCategory" class="cursor-pointer">
              <option value="all">All</option>
              <option value="women's clothing">Women's Clothing</option>
              <option value="men's clothing">Men's Clothing</option>
              <option value="jewelery">Jewelery</option>
              <option value="electronics">Electronics</option>
            </select>
          </div>
          <div class="flex items-center gap-2">
            <label for="rating-filter">Rating:</label>
            <select id="rating-filter" v-model.number="minimumRating" class="cursor-pointer">
              <option :value="0">All</option>
              <option v-for="rating in 5" :key="rating" :value="rating">
                <span v-if="rating != 5">{{ rating }}+ stars</span>
                <div v-else>
                  <span>{{ rating }}</span>
                  <span> stars</span>
                </div>
              </option>
            </select>
          </div>
          <div
            class="flex w-full max-w-xs flex-col gap-2 rounded-lg border border-gray-200 bg-white px-4 py-2 shadow-sm"
          >
            <label class="font-semibold text-gray-800">Price</label>
            <div class="flex flex-col gap-2">
              <div class="relative flex h-4 w-full items-center px-2">
                <div class="absolute h-1 w-full rounded bg-gray-200"></div>
                <div class="absolute h-1 rounded bg-cyan-600" :style="priceRangeStyle"></div>
                <input
                  :value="minPrice ?? priceBounds.min"
                  type="range"
                  :min="priceBounds.min"
                  :max="maxPrice ?? priceBounds.max"
                  step="0.01"
                  aria-label="Minimum price slider"
                  class="price-range z-10"
                  @input="updateMinPrice"
                />
                <input
                  :value="maxPrice ?? priceBounds.max"
                  type="range"
                  :min="minPrice ?? priceBounds.min"
                  :max="priceBounds.max"
                  step="0.01"
                  aria-label="Maximum price slider"
                  class="price-range"
                  @input="updateMaxPrice"
                />
              </div>
              <div class="grid grid-cols-[1fr_auto_1fr] items-center gap-2">
                <label for="min-price" class="sr-only">Minimum price</label>
                <div class="relative">
                  <span class="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500">$</span>
                  <input
                    id="min-price"
                    :value="minPrice ?? ''"
                    type="number"
                    :min="priceBounds.min"
                    :max="maxPrice ?? priceBounds.max"
                    step="0.01"
                    placeholder="Min"
                    class="w-full rounded-md border border-gray-300 py-1 pl-7 pr-2 text-sm focus:border-cyan-600 focus:outline-none focus:ring-1 focus:ring-cyan-600"
                    @input="updateMinPrice"
                  />
                </div>
                <span class="text-gray-500">-</span>
                <label for="max-price" class="sr-only">Maximum price</label>
                <div class="relative">
                  <span class="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500">$</span>
                  <input
                    id="max-price"
                    :value="maxPrice ?? ''"
                    type="number"
                    :min="minPrice ?? priceBounds.min"
                    :max="priceBounds.max"
                    step="0.01"
                    placeholder="Max"
                    class="w-full rounded-md border border-gray-300 py-1 pl-7 pr-2 text-sm focus:border-cyan-600 focus:outline-none focus:ring-1 focus:ring-cyan-600"
                    @input="updateMaxPrice"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- products -->
      <ul
        class="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-3 2xl:grid-cols-4"
      >
        <li v-for="product in filteredProducts" :key="product.id">
          <ProductComponent :productData="product" />
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import NavbarComponent from '../components/comun/NavbarComponent.vue'
import ProductComponent from '../components/product/ProductCompent.vue'
import { computed, onMounted, ref } from 'vue'
import type { Product } from '../interfaces/product'
import productData from '../components/product/ProductData.json'

const products = ref<Product[]>([])
const selectedCategory = ref('all')
const minimumRating = ref(0)
const minPrice = ref<number | null>(null)
const maxPrice = ref<number | null>(null)

const priceBounds = computed(() => {
  if (products.value.length === 0) {
    return { min: 0, max: 100 }
  }

  const prices = products.value.map((product) => product.price)
  return { min: Math.min(...prices), max: Math.max(...prices) }
})

const priceRangeStyle = computed(() => {
  const range = priceBounds.value.max - priceBounds.value.min || 1
  const minPercent =
    (((minPrice.value ?? priceBounds.value.min) - priceBounds.value.min) / range) * 100
  const maxPercent =
    (((maxPrice.value ?? priceBounds.value.max) - priceBounds.value.min) / range) * 100

  return {
    left: `${minPercent}%`,
    width: `${maxPercent - minPercent}%`,
  }
})

const updateMinPrice = (event: Event) => {
  const value = (event.target as HTMLInputElement).value
  minPrice.value = value === '' ? null : Number(value)
}

const updateMaxPrice = (event: Event) => {
  const value = (event.target as HTMLInputElement).value
  maxPrice.value = value === '' ? null : Number(value)
}

const filteredProducts = computed(() =>
  products.value.filter(
    (product) =>
      (selectedCategory.value === 'all' || product.category === selectedCategory.value) &&
      product.rating.rate >= minimumRating.value &&
      (minPrice.value === null || product.price >= minPrice.value) &&
      (maxPrice.value === null || product.price <= maxPrice.value),
  ),
)

const getData = async () => {
  try {
    const response = await fetch('https://fakestoreapi.com/products')
    if (!response.ok) {
      throw new Error(`Product API returned HTTP ${response.status}`)
    }

    products.value = await response.json()
  } catch (error) {
    console.error('Could not load products from the API; using local sample data.', error)
    products.value = productData
  }
}

onMounted(() => {
  getData()
})
</script>

<style scoped>
.price-range {
  position: absolute;
  width: 100%;
  height: 1.25rem;
  margin: 0;
  appearance: none;
  background: transparent;
  pointer-events: none;
}

.price-range::-webkit-slider-runnable-track {
  height: 0.25rem;
  background: transparent;
}

.price-range::-moz-range-track {
  height: 0.25rem;
  background: transparent;
}

.price-range::-webkit-slider-thumb {
  width: 1.25rem;
  height: 1.25rem;
  margin-top: -0.5rem;
  appearance: none;
  border: 2px solid #0891b2;
  border-radius: 9999px;
  background: white;
  pointer-events: auto;
  cursor: pointer;
}

.price-range::-moz-range-thumb {
  width: 1rem;
  height: 1rem;
  border: 2px solid #0891b2;
  border-radius: 9999px;
  background: white;
  pointer-events: auto;
  cursor: pointer;
}
</style>

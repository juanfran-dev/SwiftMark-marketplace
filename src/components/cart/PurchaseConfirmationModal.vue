<template>
  <div
    @click.self="closeModal"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4"
  >
    <div
        v-if="!isAccepted"
        class="flex max-h-[90vh] w-full max-w-2xl flex-col items-center gap-4 overflow-y-auto rounded-md bg-white p-4 sm:p-6"
    >
        <h2 class="text-xl font-bold text-cyan-700">Purchase Confirmation</h2>

        <div class="w-full overflow-x-auto rounded-md border border-cyan-200">
          <table class="w-full min-w-96 border-collapse text-left">
            <thead class="bg-cyan-50 text-sm tracking-wide text-cyan-800">
              <tr>
                <th scope="col" class="px-4 py-3 font-semibold">PRODUCT</th>
                <th scope="col" class="px-4 py-3 text-center font-semibold">QUANTITY</th>
                <th scope="col" class="px-4 py-3 text-right font-semibold">PRICE</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-cyan-100">
              <tr
                v-for="item in cart.items"
                :key="item.product.id"
                class="transition-colors hover:bg-cyan-50/60 text-sm"
              >
                <td class="px-4 py-3 font-medium text-cyan-900">
                  {{ item.product.title }}
                </td>
                <td class="px-4 py-3 text-center text-cyan-700">
                  {{ item.quantity }}
                </td>
                <td class="whitespace-nowrap px-4 py-3 text-right font-medium text-cyan-900">
                  ${{ (item.product.price * item.quantity).toFixed(2) }}
                </td>
              </tr>
            </tbody>
            <tfoot class="border-t-2 border-cyan-200 bg-amber-50">
              <tr>
                <th scope="row" colspan="2" class="px-4 py-3 text-right font-bold text-cyan-900">
                  Total:
                </th>
                <td class="whitespace-nowrap px-4 py-3 text-right font-bold text-cyan-900">
                  ${{ cart.totalPrice.toFixed(2) }}
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
        <div class="flex items-center justify-center w-full gap-4 mt-auto">
          <button
            class="flex items-center py-2 px-4 rounded-md text-md font-medium border border-cyan-500 hover:bg-cyan-50 text-cyan-700 cursor-pointer"
            @click="closeModal"
          >
            Continue shopping
          </button>
          <button
            class="flex items-center py-2 px-4 rounded-md text-md font-medium bg-cyan-500 hover:bg-cyan-600 active:bg-cyan-700 text-white cursor-pointer"
            @click="acceptPurchase"
          >
            <span>Confirm Test Purchase</span>
          </button>
        </div>
    </div>

    <div
        v-else
        class="flex max-h-[90vh] w-full max-w-2xl flex-col items-center gap-4 overflow-y-auto rounded-md bg-white p-4 sm:p-6"
    >
        <h2 class="text-xl font-bold text-cyan-700">Demo Purchase Successful</h2>
        <p class="text-cyan-800 text-center">
          Thank you for your purchase! No payment was processed.
        </p>
        <div class="flex items-center justify-center w-full gap-4 mt-auto">
          <button
            class="flex items-center py-2 px-4 rounded-md text-md font-medium border border-cyan-500 hover:bg-cyan-50 text-cyan-700 cursor-pointer"
            @click="closeModal"
          >
            Close
          </button>
        </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useCartStore } from '../../stores/cartStore'
import { ref } from 'vue'

const cart = useCartStore()
const isAccepted = ref(false)

const emit = defineEmits<{ close: [] }>()

const acceptPurchase = () => {
  isAccepted.value = true
  cart.clearCart()
}

const closeModal = () => {
  emit('close')
}
</script>

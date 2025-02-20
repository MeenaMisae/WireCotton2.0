<template>
  <div class="relative">
    <input
      id="quantity"
      v-model="localQuantity"
      type="number"
      inputmode="numeric"
      class="input-number border rounded h-12 px-5 text-[#959595] w-full shadow-sm placeholder:text-[#959595]"
      :class="{ 'text-inherit': localQuantity > 0 }"
      min="1"
      oninput="this.value = Math.round(this.value)"
      placeholder="0"
    />
    <div class="input-buttons absolute top-1 right-0 mr-0.5 space-y-1">
      <div class="flex justify-end">
        <button class="flex items-center justify-center w-10 quantity-button" @click="increment">
          <ChevronLeft class="rotate-90" :stroke="'#94A3B8'" :width="16" />
        </button>
      </div>
      <div class="flex justify-end">
        <button class="flex items-center justify-center w-10 quantity-button" @click="decrement">
          <ChevronLeft class="-rotate-90" :stroke="'#94A3B8'" :width="16" />
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import ChevronLeft from './icons/ChevronLeft.vue'

const props = defineProps({
  modelValue: {
    type: Number,
    default: 0
  }
})
const emit = defineEmits(['update:modelValue'])
const localQuantity = ref(props.modelValue)

watch(localQuantity, (newValue) => {
  emit('update:modelValue', newValue)
})

const increment = () => {
  localQuantity.value++
}

const decrement = () => {
  if (localQuantity.value > 0) {
    localQuantity.value--
  }
}
</script>

<style scoped>
input[type='number']::-webkit-inner-spin-button,
input[type='number']::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.input-number {
  transition:
    background-color 0.2s,
    color 0.2s,
    border-color 0.2s,
    box-shadow 0.2s,
    outline-color 0.2s;
  outline-color: transparent;
  border: 1px solid #cbd5e1;
}

.input-number:focus {
  outline: 1px solid #000000;
  outline-offset: -1px;
  border-color: #94a3b8;
}

.input-number:hover {
  border-color: #94a3b8;
}

.quantity-button {
  transition:
    background-color 0.2s,
    color 0.2s,
    border-color 0.2s,
    box-shadow 0.2s,
    outline-color 0.2s;
  outline-color: transparent;
  opacity: 0.6;
}

.quantity-button:active {
  opacity: 1;
}

.quantity-button:hover {
  background-color: #f1f5f9;
  border-radius: 4px;
}
</style>

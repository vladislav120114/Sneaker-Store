<script setup>
  import DrawerHead from './DrawerHead.vue';
  import CartList from './CartList.vue';
import InfoBlock from './InfoBlock.vue';

  const emit = defineEmits(['createOrder'])

  defineProps({
    totalPrice: Number,
    taxPrice: Number,
    disabledButton: Boolean
  })

</script>


<template>
  <div class="fixed top-0 left-0 w-full h-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    
    <DrawerHead />
    <div v-if="!totalPrice" class="flex h-full items-center">
      <InfoBlock 
       
        title="Корзина пустая" 
        description="Добавьте хотя бы одну пару кроссовок чтобы сделать заказ."
        image-url="/package-icon.png"
      />
    </div>
    <div v-else>
      <CartList />
      <div v-if="totalPrice" class="flex flex-col gap-4 mt-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1"></div>
          <b>{{ totalPrice }} руб.</b>
        </div>
        <div class="flex gap-2">
          <span>Налог 5%:</span>
          <div class="flex-1"></div>
          <b>{{ taxPrice }} руб.</b>
        </div>
        <button 
          :disabled="disabledButton"
          @click="() => emit('createOrder')"
          class="mt-4 bg-lime-500 w-full rounded-xl py-3 text-white cursor-pointer transition hover:bg-lime-600 active:bg-lime-700 disabled:bg-slate-400"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>
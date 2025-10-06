<script setup>
  import axios from 'axios';
  import { ref, inject, computed } from 'vue';

  import DrawerHead from './DrawerHead.vue';
  import CartList from './CartList.vue';
  import InfoBlock from './InfoBlock.vue';

  const props = defineProps({
    totalPrice: Number,
    taxPrice: Number,
  })

  const isCreatingOrder = ref(false)
  const orderId = ref(null)

  const { cart } = inject('cart')

  const createOrder = async () => {
		try {
			isCreatingOrder.value = true
			const {data} = await axios.post(`https://595d54c558fafcda.mokky.dev/orders`, {
				items: cart.value,
				totalPrice: props.totalPrice.value
			})
			cart.value = []
      orderId.value = data.id
		} catch (e) {
			console.log(e)
		} finally {
			isCreatingOrder.value = false
		}
	}

  const cartButtonDisabled = computed(() => isCreatingOrder.value || props.totalPrice.value === 0)

</script>


<template>
  <div class="fixed top-0 left-0 w-full h-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">

    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <InfoBlock
        v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок чтобы сделать заказ."
        image-url="/package-icon.png"
      />
      <InfoBlock
        v-if="orderId"
        title="Заказ оформлен!"
        :description="`Ваш заказ #${orderId} скоро будет передан курьерской доставке`"
        image-url="/order-success-icon.png"
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
          :disabled="cartButtonDisabled"
          @click="createOrder"
          class="mt-4 bg-lime-500 w-full rounded-xl py-3 text-white cursor-pointer transition hover:bg-lime-600 active:bg-lime-700 disabled:bg-slate-400"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>

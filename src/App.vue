<script setup>
	import VueHeader from './components/VueHeader.vue';
	import VueDrawer from './components/VueDrawer.vue';
	import { ref, watch, provide, computed} from 'vue';
	import axios from 'axios';


	const cart = ref([])
	const isCreatingOrder = ref(false)

	const drawerOpen = ref(false)

	const totalPrice = computed(() => cart.value.reduce((total, item) => total + item.price, 0))
	const taxPrice = computed(() => totalPrice.value / 100 * 5)

	const cartButtonDisabled = computed(() => isCreatingOrder.value || totalPrice.value === 0)
	
	const closeDrawer = () => {
		drawerOpen.value = false
	}

	const openDrawer = () => {
		drawerOpen.value = true
	}

	const addToCart = (item) => {
		cart.value.push(item)
		item.isAdded = true
	}

	const removeFromCart = (item) => {
		cart.value.splice(cart.value.indexOf(item), 1)
		item.isAdded = false
	}

	const createOrder = async () => {
		try {
			isCreatingOrder.value = true
			await axios.post(`https://595d54c558fafcda.mokky.dev/orders`, {
				items: cart.value,
				totalPrice: totalPrice.value
			})
			cart.value = []
		} catch (e) {
			console.log(e)
		} finally {
			isCreatingOrder.value = false
		}
	}
	
	watch(cart, () => { 
		localStorage.setItem('cart', JSON.stringify(cart.value))
	}, 
	{ deep: true })
	

	provide('cart', {
		cart,
		closeDrawer,
		openDrawer,
		addToCart,
		removeFromCart
	})
</script>

<template>
	<VueDrawer 
		v-if="drawerOpen" 
		:total-price="totalPrice" 
		:tax-price="taxPrice" 
		@createOrder="createOrder"
		:disabled-button="cartButtonDisabled"
	/>
	<div class="bg-white w-4/5 mt-14 m-auto rounded-xl shadow-xl">	
		<VueHeader :totalPrice="totalPrice" @openDrawer="openDrawer"/>

		<div class="p-10">
			<router-view></router-view>
		</div>
	</div>
</template>

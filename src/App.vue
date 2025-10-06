<script setup>
	import VueHeader from './components/VueHeader.vue';
	import CardList from './components/CardList.vue';
	import VueDrawer from './components/VueDrawer.vue';
	import { onMounted, reactive, ref, watch, provide, computed} from 'vue';
	import axios from 'axios';
	

	const items = ref([])
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

	const filters = reactive({
		sortBy: 'title',
		searchQuery: ''
	})

	const onChangeSelect = (event) => {
		filters.sortBy = event.target.value
	}

	const onChangeInput = (event) => {
		filters.searchQuery = event.target.value
	}

	const fetchFavorites = async () => {
		try {
			const {data} = await axios.get('https://595d54c558fafcda.mokky.dev/favorites');
			items.value = items.value.map(item => {
				const favorite = data.find(favorite => favorite.parentId === item.id)
				if (!favorite){
					return item;
				}
				return {
					...item,
					isFavorite: true,
					favoriteId: favorite.id
				}
			})
		} catch (e) {
			console.log(e)
		}
	}

	const addFavorite = async (item) => {
		try {
			if (!item.isFavorite) {

				item.isFavorite = true
				const obj = {
					parentId: item.id
				}
				const {data} = await axios.post('https://595d54c558fafcda.mokky.dev/favorites', obj)
				
				item.favoriteId = data.id
			} else {
				item.isFavorite = false
				await axios.delete(`https://595d54c558fafcda.mokky.dev/favorites/${item.favoriteId}`)
				item.favoriteId = null
			}
			
		} catch (e) {
			console.log(e)
		}
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
	
	const onClickAtPlus = (item) => {
		if (!item.isAdded) {
			addToCart(item)
		} else {
			removeFromCart(item)
		}
	}
	
	const fetchItems = async () => {
		try {
			const params = {
				sortBy: filters.sortBy,
			}

			if (filters.searchQuery) {
				params.title = `*${filters.searchQuery}`;
			}

			const {data} = await axios.get('https://595d54c558fafcda.mokky.dev/items', {params});
			items.value = data.map(obj => ({
				...obj,
				isFavorite: false,
				favoriteId: null,
				isAdded: false,
				addId: null
			}))
		} catch (e) {
			console.log(e)
		}
	}

	onMounted(async () => {
		
		const localCart = localStorage.getItem('cart')
		cart.value = localCart ? JSON.parse(localCart) : []

		await fetchItems();
		await fetchFavorites();

		items.value = items.value.map((item) => ({
			...item,
			isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
		}))
	})
	
	watch(filters, fetchItems)
	watch(cart, () => {
		items.value = items.value.map((item) => (
		{
			...item,
			isAdded: false
		}))
	})
	watch(cart, () => { localStorage.setItem('cart', JSON.stringify(cart.value))}, { deep: true })

	provide('cart', {
		cart,
		closeDrawer,
		openDrawer,
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
			<div class="flex justify-between items-center">
				<h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
				<div class="flex gap-4">
					<select @change="onChangeSelect" class="py-2 px-1 border border-gray-200 rounded-md outline-none">
						<option value="name">По названию</option>
						<option value="price">Цена по возрастанию</option>
						<option value="-price">Цена по убыванию</option>
					</select>
					<div class="relative">
						<img src="/search.svg" alt="search" class="absolute left-4 top-3"/>
						<input
							@change="onChangeInput"
							class="border border-gray-200 rounded-md py-2 pl-10 pr-4 outline-none transition focus:border-gray-400" 
							type="text"
							placeholder="Поиск..."
					/>
					</div>
				</div>
			</div>
				
			<CardList :items="items" @addFavorite="addFavorite" @addCart="onClickAtPlus"/>
		</div>
	</div>
</template>

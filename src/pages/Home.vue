<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
	import { inject, reactive, watch, ref, onMounted } from 'vue';
	import CardList from '../components/CardList.vue';
	import axios from 'axios';

	const items = ref([])

	const { cart, addToCart, removeFromCart } = inject('cart')

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

	const onClickAtPlus = (item) => {
		if (!item.isAdded) {
			addToCart(item)
		} else {
			removeFromCart(item)
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

</script>

<template>
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
	<div class="mt-10">
			<CardList :items="items" @addFavorite="addFavorite" @addCart="onClickAtPlus"/>
	</div>
</template>
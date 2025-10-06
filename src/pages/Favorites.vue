<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
	import CardList from '@/components/CardList.vue';
	import axios from 'axios';
	import { onMounted, ref } from 'vue';

	const favorites = ref([])

	onMounted(async () => {
		try {
			const {data} = await axios.get('YOUR-API-LINK')

			favorites.value = data.map(obj => obj.item)
		} catch (e) {
			console.log(e)
		}
	})
</script>

<template>
	<h2 class="text-3xl font-bold mb-8">Мои закладки</h2>
	<CardList :items="favorites" is-favorites/>
</template>

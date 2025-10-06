<script setup>
	import VueCard from './VueCard.vue';
	defineProps({
		items: Array,
	});

	const emit = defineEmits(['addFavorite', 'addCart']);

</script>


<template>
	<div class="grid grid-cols-4 gap-5" v-auto-animate>
		<VueCard 
			v-for="item in items"
			:key="item.id"
			:id="item.id"
			:title="item.title" 
			:imageUrl="item.imageUrl" 
			:price="item.price"
			:onClickFavorite="() => emit('addFavorite', item)"
			:onClickAdd="() => emit('addCart', item)"
			:isFavorite="item.isFavorite"
			:isAdded="item.isAdded"
		/>
	</div>
</template>
<script setup>

	const emit = defineEmits(['onClickRemove'])

	defineProps({
		imageUrl: String,
		title: String,
		price: Number,
	})
</script>

<template>
	<div class="flex items-center border border-slate-100 p-4 rounded-xl gap-4">
		<img :src="imageUrl" alt="sneaker" class="w-16 h-16">
		<div class="flex flex-col flex-1">
			<p>{{ title }}</p>
			<div class="flex justify-between mt-2">
				<b class="flex-1">{{ price }} руб.</b>
				<img @click="emit('onClickRemove')" src="/close.svg" alt="svg" class="opacity-40 hover:opacity-100 cursor-pointer transition"/>
			</div>
		</div>
	</div>
</template>
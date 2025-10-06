<script setup>
  defineProps({
		title: String,
		description: String,
		imageUrl: String,
	})
</script>

<template>
  <img class="width=" :src="imageUrl" alt="info">
  <h2>{{ title }}</h2>
  <p>{{ description }}</p>
</template>
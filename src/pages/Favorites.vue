<script setup>
import { onMounted, ref } from 'vue';
import axios from 'axios';
import CardList from '../components/CardList.vue';

const serviceUrl = 'https://110fd33f1ed87123.mokky.dev';

const favorites = ref([]);

onMounted(async () => {
  try {
    const { data } = await axios.get(`${serviceUrl}/favorites?_relations=items`);
    favorites.value = data.map((o) => o.item);
  } catch (e) {
    console.log('Data favorite load error', e);
  }
});
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>
  </div>
  <div class="mt-10">
    <CardList :items="favorites" hide-icons="true" />
  </div>
</template>

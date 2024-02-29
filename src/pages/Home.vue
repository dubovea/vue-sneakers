<script setup>
import { onMounted, inject, ref, watch } from 'vue';
import axios from 'axios';

import CardList from '../components/CardList.vue';

const serviceUrl = 'https://110fd33f1ed87123.mokky.dev';

const { cartItems, addToCart, addToFavorite, removeFromCart } = inject('drawerProvider');

const items = ref([]),
  filters = ref({
    sortParam: '',
    searchParam: ''
  });
const handleChangeSort = (event) => {
  filters.value.sortParam = event.target.value;
};
const handleChangeSearch = (event) => {
  filters.value.searchParam = event.target.value;
};

const fetchItems = async () => {
  try {
    let params = {};
    if (filters.value.searchParam) {
      params.title = `*${filters.value.searchParam}*`;
    }
    if (filters.value.sortParam) {
      params.sortBy = filters.value.sortParam;
    }
    const { data } = await axios.get(`${serviceUrl}/items`, { params });
    items.value = data.filter((item) => item.title && item.price && item.imageUrl);
  } catch (e) {
    console.log('Data load error', e);
  }
};
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`${serviceUrl}/favorites`),
      mapFavorites = new Map();
    favorites.forEach((favorite) => {
      if (!mapFavorites.has(favorite.parentId)) {
        mapFavorites.set(favorite.parentId, favorite);
      }
    });

    items.value = items.value.map((item) => ({
      ...item,
      isFavorite: mapFavorites.has(item.id)
    }));
  } catch (e) {
    console.log('Data favorite load error', e);
  }
};
onMounted(async () => {
  await fetchItems();
  await fetchFavorites();
});

watch(
  cartItems,
  () => {
    const mapCartData = new Map();
    cartItems.value.forEach((item) => {
      if (!mapCartData.has(item.parentId)) {
        mapCartData.set(item.parentId, item);
      }
    });
    items.value = items.value.map((item) => ({
      ...item,
      cartId: mapCartData.get(item.id)?.parentId ?? null,
      isAdded: mapCartData.has(item.id)
    }));
  },
  {
    deep: true
  }
);

watch(filters.value, fetchItems);
</script>
<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>
    <div class="flex gap-4">
      <select
        class="border rounded-md py-2 px-3 outline-none focus:border-gray-400"
        @change="handleChangeSort"
      >
        <option value="title">По названию</option>
        <option value="-price">По цене (убыванию)</option>
        <option value="price">По цене (возрастанию)</option>
      </select>
      <div class="relative">
        <img class="absolute left-4 top-3" src="/search.svg" alt="Search" />
        <input
          class="border rounded-md py-2 pl-12 pr-4 outline-none focus:border-gray-400"
          type="text"
          placeholder="Поиск"
          @change="handleChangeSearch"
        />
      </div>
    </div>
  </div>
  <div class="mt-10">
    <CardList
      :items="items"
      @add-to-favorite="addToFavorite"
      @add-to-cart="addToCart"
      @remove-from-cart="removeFromCart"
    />
  </div>
</template>

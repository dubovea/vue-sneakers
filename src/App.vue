<script setup>
import { onMounted, ref, watch, provide } from 'vue'
import axios from 'axios'

import Header from './components/Header.vue'
import CardList from './components/CardList.vue'
import Drawer from './components/Drawer.vue'

const items = ref([]),
  filters = ref({
    sortParam: '',
    searchParam: ''
  })

const fetchItems = async () => {
  try {
    let params = {}
    if (filters.value.searchParam) {
      params.title = `*${filters.value.searchParam}*`
    }
    if (filters.value.sortParam) {
      params.sortBy = filters.value.sortParam
    }
    const { data } = await axios.get('https://604781a0efa572c1.mokky.dev/items', { params })
    items.value = data.filter((item) => item.title && item.price && item.imageUrl)
  } catch (e) {
    console.log('Data load error', e)
  }
}
const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://604781a0efa572c1.mokky.dev/favorites'),
      mapFavorites = new Map()
    favorites.forEach((favorite) => {
      if (!mapFavorites.has(favorite.parentId)) {
        mapFavorites.set(favorite.parentId, favorite)
      }
    })

    items.value = items.value.map((item) => ({
      ...item,
      isFavorite: mapFavorites.has(item.id)
    }))
  } catch (e) {
    console.log('Data load error', e)
  }
}
const handleChangeSort = (event) => {
  filters.value.sortParam = event.target.value
}
const handleChangeSearch = (event) => {
  filters.value.searchParam = event.target.value
}

onMounted(async () => {
  await fetchItems()
  await fetchFavorites()
})
watch(filters.value, fetchItems)

provide('addToFavorite', (id) => {
  debugger
})
</script>

<template>
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <!-- <Drawer /> -->
    <Header />
    <div class="p-10">
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
        <CardList :items="items" />
      </div>
    </div>
  </div>
</template>

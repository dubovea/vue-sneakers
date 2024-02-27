<script setup>
import { computed, onMounted, provide, ref, watch } from 'vue';
import axios from 'axios';

import Header from './components/Header.vue';
import CardList from './components/CardList.vue';
import Drawer from './components/Drawer.vue';
const serviceUrl = 'https://110fd33f1ed87123.mokky.dev';

const items = ref([]),
  cartItems = ref([]),
  drawerOpen = ref(false),
  filters = ref({
    sortParam: '',
    searchParam: ''
  }),
  totalPrice = computed(() => cartItems.value.reduce((acc, item) => acc + item.price, 0));

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
const fetchCartItems = async () => {
  try {
    const { data: cartData } = await axios.get(`${serviceUrl}/cart`),
      mapCartData = new Map();
    cartData.forEach((item) => {
      if (!mapCartData.has(item.parentId)) {
        mapCartData.set(item.parentId, item);
      }
    });
    items.value = items.value.map((item) => ({
      ...item,
      cartId: mapCartData.get(item.id)?.parentId ?? null,
      isAdded: mapCartData.has(item.id)
    }));
    cartItems.value = cartData;
  } catch (e) {
    console.log('Data cart load error', e);
  }
};
const handleChangeSort = (event) => {
  filters.value.sortParam = event.target.value;
};
const handleChangeSearch = (event) => {
  filters.value.searchParam = event.target.value;
};

const addToFavorite = (item) => {
  item.isFavorite = true;
  const itemId = item.id;
  axios.get(`${serviceUrl}/favorites?parentId=${itemId}`).then(({ data }) => {
    if (!data.length) {
      axios
        .post(`${serviceUrl}/favorites`, {
          parentId: item.id
        })
        .then(({ data }) => {
          item.favoriteId = data.id;
          item.isFavorite = true;
        })
        .catch((e) => {
          console.log('Favorite create error', e);
        });
      return;
    }
    let promises = data.map((favorite) => {
      axios
        .delete(`${serviceUrl}/favorites/${favorite.id}`)
        .then(() => {
          item.isFavorite = false;
        })
        .catch((e) => {
          console.log('Favorite delete error', e);
        });
    });
    Promise.allSettled(promises).then((promise) => {});
  });
};
const addToCart = (item) => {
  cartItems.value.push({ ...item, parentId: item.id });
  item.isAdded = true;
  const itemId = item.id;
  axios.get(`${serviceUrl}/cart?parentId=${itemId}`).then(({ data }) => {
    if (!data.length) {
      axios
        .post(`${serviceUrl}/cart`, {
          ...item,
          parentId: item.id
        })
        .then(({ data }) => {})
        .catch((e) => {
          console.log('Favorite create error', e);
        });
      return;
    }
  });
};
const removeFromCart = (item) => {
  item.isAdded = false;

  const parentId = item.parentId ?? item.cartId;
  items.value.find((i) => i.id === parentId).isAdded = false;
  cartItems.value.splice(cartItems.value.indexOf(item), 1);

  axios.get(`${serviceUrl}/cart?parentId=${parentId}`).then(({ data }) => {
    if (!data.length) {
      return;
    }
    let promises = data.map((cartItem) => {
      axios
        .delete(`${serviceUrl}/cart/${cartItem.id}`)
        .then(() => {
          item.isAdded = false;
        })
        .catch((e) => {
          console.log('Cart item delete error', e);
        });
    });
    Promise.allSettled(promises).then((promise) => {});
  });
};

const createOrder = () => {
  axios
    .post(`${serviceUrl}/orders`, { items: cartItems.value, totalPrice: totalPrice.value })
    .then(({ data }) => {
      cartItems.value = [];
    })
    .catch((e) => {
      console.log('Order create error', e);
    });
};

onMounted(async () => {
  await fetchItems();
  await fetchFavorites();
  await fetchCartItems();
});
watch(filters.value, fetchItems);

const changeDrawerVisible = () => {
  drawerOpen.value = !drawerOpen.value;
};
provide('drawerProvider', { cartItems, removeFromCart, changeDrawerVisible });
</script>

<template>
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Drawer v-if="drawerOpen" :total-price="totalPrice" @create-order="createOrder" />
    <Header :total-price="totalPrice" />
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
        <CardList
          :items="items"
          @add-to-favorite="addToFavorite"
          @add-to-cart="addToCart"
          @remove-from-cart="removeFromCart"
        />
      </div>
    </div>
  </div>
</template>

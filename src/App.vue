<script setup>
import { computed, onMounted, provide, ref } from 'vue';
import axios from 'axios';

import Header from './components/Header.vue';
import Drawer from './components/Drawer.vue';

import Home from './pages/Home.vue';

const serviceUrl = 'https://110fd33f1ed87123.mokky.dev';

const cartItems = ref([]),
  drawerOpen = ref(false),
  totalPrice = computed(() => cartItems.value.reduce((acc, item) => acc + item.price, 0));

const changeDrawerVisible = () => {
  drawerOpen.value = !drawerOpen.value;
};

const fetchCartItems = async () => {
  try {
    const { data: cartData } = await axios.get(`${serviceUrl}/cart`);
    cartItems.value = cartData;
  } catch (e) {
    console.log('Data cart load error', e);
  }
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
  const itemId = item.id;
  cartItems.value.push({ ...item, parentId: itemId });
  axios.get(`${serviceUrl}/cart?parentId=${itemId}`).then(({ data }) => {
    if (!data.length) {
      axios
        .post(`${serviceUrl}/cart`, {
          ...item,
          parentId: itemId
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

const clearCart = () => {
  axios.get(`${serviceUrl}/cart`).then(({ data }) => {
    if (!data.length) {
      return;
    }
    let promises = data.map((cartItem) => {
      axios.delete(`${serviceUrl}/cart/${cartItem.id}`).catch((e) => {
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
      clearCart();
      cartItems.value = [];
    })
    .catch((e) => {
      console.log('Order create error', e);
    });
};

onMounted(async () => {
  await fetchCartItems();
});

provide('drawerProvider', {
  cartItems,
  addToCart,
  addToFavorite,
  removeFromCart,
  changeDrawerVisible
});
</script>

<template>
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Drawer v-if="drawerOpen" :total-price="totalPrice" @create-order="createOrder" />
    <Header :total-price="totalPrice" />
    <div class="p-10">
      <Home />
    </div>
  </div>
</template>

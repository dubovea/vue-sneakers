<script setup>
import { inject, ref } from 'vue';
import axios from 'axios';
import DrawerHeader from './DrawerHeader.vue';
import BasketList from './BasketList.vue';
import InfoBlock from './InfoBlock.vue';

const { cartItems, closeDrawer } = inject('drawerProvider');
const props = defineProps({
  totalPrice: Number
});
const numberOrder = ref(null),
  isCreated = ref(false);

const serviceUrl = 'https://110fd33f1ed87123.mokky.dev';
const createOrder = () => {
  axios
    .post(`${serviceUrl}/orders`, { items: cartItems.value, totalPrice: props.totalPrice })
    .then(({ data }) => {
      numberOrder.value = data.id;
      isCreated.value = true;
      clearCart();
      cartItems.value = [];
    })
    .catch((e) => {
      console.log('Order create error', e);
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
</script>
<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70" @click="closeDrawer"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHeader />

    <div v-if="!!isCreated" class="h-4/5 flex items-center">
      <InfoBlock
        title="Заказ оформлен"
        :description="`Ваш заказ №${numberOrder} скоро будет передан курьерской доставке.`"
        image-url="/order-success-icon.png"
      />
    </div>
    <div v-else-if="!totalPrice" class="h-4/5 flex items-center">
      <InfoBlock
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ."
        image-url="/package-icon.png"
      />
    </div>
    <div v-else>
      <BasketList />

      <div class="flex flex-col gap-4 mt-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b> {{ totalPrice }} Р</b>
        </div>
        <div class="flex gap-2">
          <span>Налог 5%:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b> {{ Math.round(totalPrice * 0.05) }} Р</b>
        </div>

        <button
          :disabled="totalPrice === 0"
          @click="createOrder"
          class="w-full mt-4 rounded-xl py-3 bg-lime-500 hover:bg-lime-600 active:bg-lime-700 disabled:bg-slate-300 cursor-pointer transition"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>

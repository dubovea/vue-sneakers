<script setup>
import { inject } from 'vue';
import BasketItem from './BasketItem.vue';
const { cartItems, removeFromCart } = inject('drawerProvider');
</script>

<template>
  <div class="flex flex-col gap-4" v-auto-animate>
    <BasketItem
      v-for="item in cartItems"
      :key="item.id"
      :parentId="item.parentId"
      :title="item.title"
      :imageUrl="item.imageUrl"
      :price="item.price"
      @on-click-remove="() => removeFromCart(item)"
    />
  </div>
</template>

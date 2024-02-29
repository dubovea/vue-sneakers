<script setup>
import { inject } from 'vue';
defineProps({
  totalPrice: Number
});
const { changeDrawerVisible } = inject('drawerProvider');
</script>
<template>
  <header class="flex justify-between border-b border-slate-200 px-8 py-8">
    <router-link to="/">
      <div class="flex items-center gap-4">
        <img src="/logo.png" alt="Logo" class="w-10" />
        <div>
          <h2 class="text-xl font-bold uppercase">Vue Sneakers</h2>
          <p class="text-slate-400">Магазин лучших кроссовок</p>
        </div>
      </div></router-link
    >

    <ul class="flex items-center gap-8">
      <li class="header-item" @click="changeDrawerVisible">
        <img src="/cart.svg" alt="Cart" />
        <span>{{ totalPrice }} руб.</span>
      </li>

      <router-link to="/favorites">
        <li class="header-item">
          <img src="/heart.svg" alt="Cart" />
          <span>Закладки</span>
        </li></router-link
      >

      <li class="header-item">
        <img src="/profile.svg" alt="Profile" />
        <span>Профиль</span>
      </li>
    </ul>
  </header>
</template>

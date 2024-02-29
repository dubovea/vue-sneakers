<script setup>
import DrawerHeader from './DrawerHeader.vue';
import BasketList from './BasketList.vue';
import InfoBlock from './InfoBlock.vue';
const emit = defineEmits(['createOrder']);
defineProps({
  totalPrice: Number
});
</script>
<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHeader />

    <div v-if="!totalPrice" class="h-4/5 flex items-center">
      <InfoBlock
        image-url="/package-icon.png"
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ."
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
          @click="emit('createOrder')"
          class="w-full mt-4 rounded-xl py-3 bg-lime-500 hover:bg-lime-600 active:bg-lime-700 disabled:bg-slate-300 cursor-pointer transition"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import Card from './Card.vue';

defineProps({
  items: Array,
  hideIcons: Boolean
});

const emit = defineEmits(['addToFavorite', 'addToCart', 'removeFromCart']);

const actionAddOrRemove = (item) => {
  if (item.isAdded) {
    emit('removeFromCart', item);
    return;
  }
  emit('addToCart', item);
};
</script>
<template>
  <div class="grid grid-cols-4 gap-5" v-auto-animate>
    <Card
      v-for="item in items"
      :key="item.id"
      :id="item.id"
      :title="item.title"
      :imageUrl="item.imageUrl"
      :price="item.price"
      :isFavorite="item.isFavorite"
      :isAdded="item.isAdded"
      :hide-icons="hideIcons"
      :onClickFavorite="() => emit('addToFavorite', item)"
      :onClickAdd="() => actionAddOrRemove(item)"
    />
  </div>
</template>

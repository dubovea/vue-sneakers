<script setup>
import Card from './Card.vue'

const onClickAdd = () => {
  alert('123')
}
const onClickFavorite = () => {
  alert('123')
}
</script>
<template>
  <div class="grid grid-cols-4 gap-5">
    <Card
      title="Мужские кроссовки Nike Blazer Mid Suede"
      imageUrl="/sneakers/sneakers-1.jpg"
      :price="5000"
      isFavorite
      isAdded
      :onClickAdd="onClickAdd"
      :onClickFavorite="onClickFavorite"
    />
  </div>
</template>

<template>
  <div id="app">
    <ShoppingList />
  </div>
</template>

<script>
import ShoppingList from './components/ShoppingList.vue';

export default {
  components: {
    ShoppingList
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
</style>

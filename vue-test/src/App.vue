<template>
  <div id="app">
  <h1>Тестовое задание</h1>
    <hr>
  </div>
    <AllPhoto
      v-bind:all="all"
    />
</template>

<script>
import  AllPhoto from '@/components/AllPhotos' //инициализируем компонент
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return{
      all: null
    }
  },
  mounted() {
    axios
        .get('https://boiling-refuge-66454.herokuapp.com/images') //отправляем запрос
        .then(response => (this.all = response.data)); //записываем в all все полученные фото
  },
  components: {
    AllPhoto
  }
}
</script>

<style>
body{
  background-color: beige;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;

}
</style>

<template>
  <div class="allPhoto">
    <PhotoItem
      v-for="photo in all"
      v-bind:photo="photo"
      v-bind:key="photo.id"
    />
  </div>
</template>

<script>
import PhotoItem from '@/components/OnePhoto'

export default {
  props: ['all'], //получаем все фото из App
  components:
      {
        PhotoItem
      },
}
</script>
<style>
  .allPhoto {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
  }

</style>
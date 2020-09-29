<template>
  <div class="container">
    <PrefChart poplation-type="total" :is-dynamic-load="true" />
    <!-- <PrefChart poplation-type="working" :is-dynamic-load="false" :prefectures="['東京都','大阪府','福岡県']" /> -->
  </div>
</template>

<script>
export default {}
</script>

<style>
.container {
  margin: 0 auto;
  justify-content: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial,
    sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

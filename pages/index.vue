<template>
  <div class="container">
    <PrefChart
      chart-title="東京都と大阪府と福岡県の 総人口 の推移"
      poplation-type="total"
      :prefectures="['東京都', '大阪府', '福岡県']"
    />
    <PrefChart
      poplation-type="yang"
      :prefectures="['東京都', '大阪府', '京都府', '福岡県', '沖縄県']"
    />
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
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
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

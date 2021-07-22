<template>
  <div class="header">
    <div class="header-content">
      ワンナイト人狼
      {{activePage.name ? ' - ' + activePage.name : ''}}
    </div>
  </div>
</template>

<script>
export default {
  name: 'Header',
  props: {
    activePage: Object
  }
}
</script>

<style scoped>
.header {
  position: fixed;
  max-width: 600px;
  width: 100%;
  height: 100px;
  top: 0;
  background: #455A64;
}

.header-content {
  color: #fff;
  font-size: 18px;
  font-weight: bold;
  text-align: left;
  line-height: 100px;
  padding: 0px 20px;
}
</style>
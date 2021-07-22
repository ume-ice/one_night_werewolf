<template>
  <div class="footer">
    <div class="next-button" @click="moveTopPage">トップ</div>
    <div v-show="activePage.page !== 'result'" class="next-button" @click="nextPage">{{buttonText}}</div>
  </div>
</template>

<script>
export default {
  name: 'Footer',
  props: {
    activePage: Object,
    buttonText: String,
    moveTopPage: Function,
    nextPage: Function,
  }
}
</script>

<style scoped>
.footer {
  position: fixed;
  max-width: 600px;
  width: 100%;
  height: 80px;
  background: #CFD8DC;
  bottom: 0px;
  color: #fff;
  text-align: center;
}

.next-button {
  display: inline-block;
  min-width: 100px;
  height: 40px;
  margin: 20px;
  border-radius: 50px;
  line-height: 40px;
  font-weight: bold;
  background: #455A64;
}
</style>
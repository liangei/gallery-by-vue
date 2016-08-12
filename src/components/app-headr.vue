<template>
  <header class="header">
    <a href="" class="logo"><img  src="../assets/logo.png"></a>
    <h1 class="title" v-text="msg"></h1>
  </header>
</template>

<script>
export default {
  data () {
    return {
      msg: '图片画廊'
    }
  }
}
</script>

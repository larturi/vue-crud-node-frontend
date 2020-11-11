<template>
  <div class="about">
    <h1>This is an about page</h1>
    <router-link :to="{query: {seccion: 1}}" class="btn btn-block btn-primary">1</router-link>
    <router-link :to="{query: {seccion: 2}}" class="btn btn-block btn-primary">2</router-link>
  </div>
</template>

<script>
export default {
  data() {
    return {
      
    }
  },
  watch: {
    "$route.query.seccion": {
      inmediate: true,
      handler(seccion) {
        console.log('Seccion ha cambiado: ' + seccion);
      }
    }
  },
}
</script>
<template>
  <v-app dark>
    <v-app-bar
      app
      color="primary"
      dark
    ></v-app-bar>

    <v-main dark>
      <newton-raphson/>
    </v-main>
  </v-app>
</template>

<script>
import newtonRaphson from './components/newton_raphson.vue';
export default {
  name: 'App',

  components: {
    newtonRaphson,
  },

  data(){
    return{
      expresion:""
    }
  },
  mounted(){
  }
};
</script>

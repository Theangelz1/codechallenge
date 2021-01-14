<template>
  <v-app>
    <v-parallax jumbotron dark height="700" :src="imagen.register">
      <v-container>
        <nuxt />
      </v-container>
    </v-parallax>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      imagen: {
        register: require("~/assets/img/register.png"),
      },
    };
  },
};
</script>

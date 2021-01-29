<template>
  <v-app class="main">
    <v-container fill-height class="align-start">
      <v-row class="justify-center">
        <v-col class="col-12 col-md-6">
          <FileLoader/>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
  import FileLoader from './components/FileLoader';

  export default {
    name: 'App',

    components: {
      FileLoader,
    },

    data: () => ({
      
    }),
  };
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css?family=Rubik:300,400,500&display=swap');

  html {
    font-family: 'Rubik', sans-serif;;
    font-color: #202124;
    font-size: 20px;
  }
</style>
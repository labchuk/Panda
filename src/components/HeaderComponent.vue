<script setup>
  import {RouterLink} from "vue-router";

</script>

<template>
  <header>
      <div class="container">
          <div class="header-inner">
              <div class="header-inner__logo">
                  <img src="@/assets/sun.png" alt="logo">
                  <p>Weather app</p>
              </div>
              <nav class="header-inner__navBar">
                  <RouterLink :to="{name: 'home'}">Home</RouterLink>
                  <RouterLink :to="{name: 'favorites'}">Favorites</RouterLink>
              </nav>
          </div>
          </div>
  </header>
</template>

<style scoped lang="scss">
  .container{
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 15px;
  }
  header{
      background-color: #48484a;
      padding: 20px 0;
  }
  .header-inner{
      display: flex;
      justify-content: space-between;
      &__logo{
          display: flex;
          cursor: pointer;
          img{
              width: 30px;
          }
          p{
              margin: auto 5px;
              color: #fff;
          }
      }
      a{
          margin-left: 20px;
          color: #fff;
          text-decoration: none;

      }
  }

</style>
<script setup>
import { RouterView } from 'vue-router'
import HeaderComponent from "@/components/HeaderComponent.vue";
</script>
<template>
    <HeaderComponent/>
    <main class="page">
        <section class="main-block">
            <div class="main-block__container">
                <RouterView v-slot="{ Component }">
                    <KeepAlive exclude="FavoriteView">
                        <component :is="Component"></component>
                    </KeepAlive>
                </RouterView>
            </div>
        </section>
    </main>
</template>

<style scoped lang="scss">
.main-block {
  padding: 60px 0;
  @media (max-width: $mobile) {
    padding: 20px 0;
  }

  &__container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 15px;
  }
  &__nav {
    display: flex;
    align-items: center;
    gap: 20px;
    font-size: 20px;
    a {
      position: relative;
      display: inline-block;
      transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
      &::after {
        content: '';
        height: 2px;
        transition: all 0.3s cubic-bezier(0.5, 0.02, 0.13, 0.5);
        bottom: 0;
        left: 0;
        width: 0;
        position: absolute;
        background: #ccc;
      }
      &.router-link-exact-active {
        &::after {
          width: 100%;
        }
      }
    }
    &:not(:last-child) {
      margin-bottom: 40px;
    }
  }
}
</style>

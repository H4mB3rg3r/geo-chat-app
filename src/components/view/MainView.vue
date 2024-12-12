<template>
  <main>
    <aside class="left">
      <SidePanel />
    </aside>
    
    <section :class="isMobile && isFlex ? 'mobile' : ''">
      <div class="side-panel-mobile">
        <SidePanel v-if="isMobile && route.name === 'home'"/>
      </div>
      <RouterView />
    </section>
  </main>
</template>

<script setup lang="ts">
  import SidePanel from '../SidePanel.vue';
  import { computed, onMounted, ref } from 'vue';
  import { useAppStore } from '../../stores/app';
  import UserForm from '../form/UserForm.vue';
  import { useRoute } from 'vue-router';

  const route = useRoute()
  const isMobile = computed(() => window.innerWidth <= 768);
  const flexPages = ref(['user-detail'])
  
  const isFlex = computed(() => {
    console.log('@__ route.name :: ', route.name)
    return flexPages.value.includes(route.name as string)
  })

  const appStore = useAppStore()

  const loginUser = computed(() => {
    return appStore.user
  })



  onMounted(() => {

  })
</script>

<style scoped>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }

  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }

  .logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
  }

  .side-panel-mobile {
    display: none;
  }
  .mobile {
    display: flex !important;
    justify-content: center;
  }



  
</style>
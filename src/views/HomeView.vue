<template>
  <div class="home">
    <Login/>
  </div>
</template>
<script>
import Login from '@/components/login-form.vue'

export default {
  name: 'HomeView',
  components: {
    Login
  }
}
</script>

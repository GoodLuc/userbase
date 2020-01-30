<template>
  <q-layout>
    <q-page-container>
      <div class="mainAppContainer">
        <router-view />
      </div>
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  name: 'Main',

  data () {
    return {
    }
  }
}
</script>
<style lang="stylus">
  .mainAppContainer { max-width: 1200px; margin: auto; padding: 20px }
</style>
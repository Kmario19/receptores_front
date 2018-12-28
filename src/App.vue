<template>
  <div id="app">
    <div class="container-fluid">
      <div class="row">
        <div class="col-sm-12">
          Graphics
        </div>
      </div>
      <div class="row">
        <div class="col-md-6 track-list">
          <Trackers />
        </div>
        <div class="col-md-6">
          Track Detail
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Trackers from './components/Trackers.vue'

export default {
  name: 'app',
  components: {
    Trackers
  }
}
</script>

<style>

</style>
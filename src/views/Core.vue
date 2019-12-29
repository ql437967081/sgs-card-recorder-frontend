<template>
  <div>
    <CardRecorder :url="backend_url"/>
  </div>
</template>

<script>
import CardRecorder from '@/components/CardRecorder'
export default {
  name: 'Core',
  components: { CardRecorder },
  data () {
    return {
      backend_url: 'http://localhost:8080/card/'
    }
  }
}
</script>

<style scoped>

</style>

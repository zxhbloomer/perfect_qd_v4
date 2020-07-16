<template>
  <div>
    <div v-for="item in permission_topNav" :key="item.index">
      {{ item.meta.name }}
    </div>
  </div>
</template>

<style scoped>
</style>

<script>
import { mapGetters } from 'vuex'

export default {
  data() {
    return {
    }
  },
  computed: {
    ...mapGetters([
      'permission_topNav'
    ])
  },
  watch: {
  },
  created() {
  },
  methods: {

  }
}
</script>


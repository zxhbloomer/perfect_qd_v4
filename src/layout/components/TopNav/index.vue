<template>
  <div>
    <template>
      <el-menu mode="horizontal" @select="handleSelect">
        <el-menu-item v-for="item in permission_topNav" :key="item.index">
          <svg-icon :icon-class="item.meta.icon" />
          <span slot="title">{(item.meta.name)}</span>
        </el-menu-item>
      </el-menu>
      <div v-for="item in permission_topNav" :key="item.index">
        {{ item.meta.name }}
      </div>
    </template>
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
    handleSelect(key, keyPath) {
      console.log(key, keyPath)
    }
  }
}
</script>


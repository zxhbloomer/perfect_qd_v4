<template>
  <div>
    <template>
      <el-menu
        mode="horizontal"
        class="topnav"
        @select="handleSelect"
      >
        <el-menu-item v-for="item in permission_topNav" :key="item.index">
          <svg-icon :icon-class="item.meta.icon" />
          <span slot="title" style="margin-left: 5px">{{ item.meta.name }}</span>
        </el-menu-item>
      </el-menu>
    </template>
  </div>
</template>

<style scoped>
  .topnav {
    background-color: transparent;
    float: left
  }
  .el-menu > .el-menu-item {
    display: inline-block;
    line-height: 50px;
    height: 50px;
    color: #fff;
    vertical-align: middle;
    border-bottom: none;
  }
  .el-menu>.el-menu-item.is-active {
    color: #fff!important;
    background-color: rgba(255, 255, 255, 0.1)!important;
  }
  .el-menu>.el-menu-item:hover {
    color: #fff!important;
    background-color: rgba(0,0,0,.1)!important
  }
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


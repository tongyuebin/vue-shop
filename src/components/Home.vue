<template>
  <div>
    首页
    <el-button type="info" @click="logout">登出</el-button>
  </div>
</template>
<script>
export default {
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>
<style lang="less" scoped>
</style>

<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
  </div>
</template>
<script>
export default {
  name: 'category.vue',
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/mallproduct/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    }
  },
  mounted () {
    this.getMenus()
  }
}
</script>

<style scoped>

</style>

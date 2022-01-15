<template>
  <el-tree :data="menus"
           :props="defaultProps"
           node-key="catId"
           @node-click="nodeClick"
           ref="menuTree">
  </el-tree>
</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      menus: [],
      expandeKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  mounted () {
    this.getMenus()
  },
  methods: {
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/mallproduct/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    nodeClick (data, node, component) {
      console.log('子组件category组件被点击')
      //向父组件发送事件
      this.$emit('tree-node-click', data, node, component)
    }
  }
}
</script>
<style scoped>
</style>

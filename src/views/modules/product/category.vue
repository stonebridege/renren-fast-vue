<template>
  <div>
    <el-tree :data="menus"
             :props="defaultProps"
             :show-checkbox="true"
             :expand-on-click-node="true"
             node-key="catId"
             :default-expanded-keys="expandeKey"
             @node-click="handleNodeClick">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2"
                     type="text"
                     size="mini"
                     @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length===0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>
<script>
export default {
  name: 'category.vue',
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
    },
    append (data) {
      console.log('append' + data)
    },
    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`是否删除【${data.name}】, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/mallproduct/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.getMenus()
          console.log(node)
          console.log(node.parent.data.catId)
          this.expandeKey = [node.parent.data.catId]
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped>

</style>

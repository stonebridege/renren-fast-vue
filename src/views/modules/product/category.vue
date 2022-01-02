<template>
  <div>
    <el-tree :data="menus"
             :props="defaultProps"
             :show-checkbox="true"
             :expand-on-click-node="false"
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
    <el-dialog title="新增节点" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCategory">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'category.vue',
  data () {
    return {
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0},
      dialogVisible: false,
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
      // console.log(data)
    },
    append (data) {
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.dialogVisible = true
    },
    // 添加分类
    addCategory () {
      this.dialogVisible = false
      this.$http({
        url: this.$http.adornUrl('/mallproduct/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(() => {
        this.getMenus()
        this.expandeKey = [this.category.parentCid]
        this.category.name = ''
        this.$message({
          type: 'success',
          message: '保存成功!'
        })
      })
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/mallproduct/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
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

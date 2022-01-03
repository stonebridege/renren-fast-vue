<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" :show-checkbox="true" :expand-on-click-node="false"
             node-key="catId"
             :default-expanded-keys="expandeKey" @node-click="handleNodeClick">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button v-if="node.childNodes.length===0" type="text" size="mini"
                     @click="() => remove(node, data)">Delete</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">Edit</el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitData()">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'category.vue',
  data () {
    return {
      title: '',
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, productUnit: '', icon: ''},
      dialogVisible: false,
      dialogType: '',
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
      this.title = '添加分类'
      this.dialogVisible = true
      this.dialogType = 'add'
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    edit (data) {
      this.title = '修改分类'
      this.dialogType = 'edit'
      this.dialogVisible = true
      // 发送器请求获取当前节点的最新请求
      this.$http({
        url: this.$http.adornUrl(`/mallproduct/category/info/${data.catId}`),
        method: 'post'
      }).then(({data}) => {
        this.category.name = data.category.name
        this.category.catId = data.category.catId
        this.category.icon = data.category.icon
        this.category.productUnit = data.category.productUnit
        this.category.parentCid = data.category.parentCid
        console.log('要回显的数据：' + JSON.stringify(this.category))
      })
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
    editCategory () {
      this.dialogVisible = false
      const {catId, name, icon, productUnit} = this.category
      this.$http({
        url: this.$http.adornUrl('/mallproduct/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.getMenus()
        this.expandeKey = [this.category.parentCid]
        this.$message({
          type: 'success',
          message: '编辑成功!'
        })
      })
    },
    submitData () {
      if (this.dialogType === 'add') {
        this.addCategory()
      } else if (this.dialogType === 'edit') {
        this.editCategory()
      }
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

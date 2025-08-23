<template>
  <div>
    <el-tree
      :data="data"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKeys"
      node-key="catId"
      show-checkbox
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.subList.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(node, data)"
          >
            Add
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      title="添加"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
      :close-on-click-modal="false"
    >
      <el-form ref="form" :model="form" label-width="120px">
        <el-form-item label="name">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="icon">
          <el-input v-model="form.icon"></el-input>
        </el-form-item>
        <el-form-item label="productUnit">
          <el-input v-model="form.productUnit"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="submitForm">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      expandedKeys: [],
      data: [],
      defaultProps: {
        children: 'subList',
        label: 'name',
      },
      dialogVisible: false,
      form: {},
    }
  },
  methods: {
    handleNodeClick(data) {
      console.log(data)
    },
    loadCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/treelist'),
        method: 'get',
      }).then(({ data }) => {
        console.log('data', data)
        this.data = data.list
      })
    },
    append(node, data) {
      console.log('append:data', data)
      this.dialogVisible = true
      this.form.parentCid = data.catId
      this.form.catLevel = data.catLevel + 1
      this.form.showStatus = 1
      this.form.sort = 0
    },

    submitForm() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.form,
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
          })
          this.expandedKeys = [this.form.parentCid]
          this.dialogVisible = false
          this.loadCategory()
        } else {
          this.$message.error(data.msg)
        }
      })
    },

    remove(node, data) {
      console.log('node', node)
      console.log('data', data)

      this.$confirm(`确定要删除【${data.name}】吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: [data.catId],
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
              type: 'success',
            })
            this.expandedKeys = [node.parent.data.catId]
            this.loadCategory()
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
  },
  mounted() {
    console.log('loading....')
    this.loadCategory()
  },
}
</script>

<style scoped></style>

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
            删除
          </el-button>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(node, data)"
          >
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => update(node, data)">
            更新
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :close-on-click-modal="false"
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
    >
      <el-form ref="categoryForm" :model="categoryForm" label-width="120px">
        <el-form-item label="商品名称">
          <el-input v-model="categoryForm.name"></el-input>
        </el-form-item>
        <el-form-item label="商品图标">
          <el-input v-model="categoryForm.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="categoryForm.productUnit"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="submitCategoryForm">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      categoryForm: {
        name: '',
        icon: '',
        productUnit: '',
        catId: '',
      },
      dialogVisible: false,
      expandedKeys: [],
      data: [],
      defaultProps: {
        children: 'subList',
        label: 'name',
      },
    }
  },
  computed: {
    dialogTitle() {
      if (this.categoryForm && this.categoryForm.catId) {
        return '修改商品'
      }
      return '添加商品'
    },
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
      this.dialogVisible = true
      // 清空数据
      this.categoryForm = {}
      // 补全数据
      this.categoryForm.parentCid = data.catId
      this.categoryForm.catLevel = data.catLevel + 1
      this.categoryForm.showStatus = 1
      this.categoryForm.sort = 0
    },

    update(node, data) {
      console.log('update.....')
      this.dialogVisible = true
      // 以免其他信息干扰
      this.categoryForm = {}
      this.categoryForm.catId = data.catId

      // 查出数据库中的信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        console.log(data)
        if (data && data.code === 0) {
          this.categoryForm = data.category
        } else {
          console.log(data.msg)
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
    submitCategoryForm() {
      this.dialogVisible = false
      let url = ''
      if (this.dialogTitle == '添加商品') {
        url = '/product/category/save'
      } else {
        url = '/product/category/update'
      }
      this.$http({
        url: this.$http.adornUrl(url),
        method: 'post',
        data: this.categoryForm,
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
          })
          this.expandedKeys = [this.categoryForm.parentCid]
          this.loadCategory()
        } else {
          this.$message.error(data.msg)
        }
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

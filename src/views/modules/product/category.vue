<template>
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
      </span>
    </span>
  </el-tree>
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
    append(data) {
      console.log('data', data)
      const newChild = { id: id++, label: 'testtest', children: [] }
      if (!data.children) {
        this.$set(data, 'children', [])
      }
      data.children.push(newChild)
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

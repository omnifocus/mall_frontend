<template>
  <el-tree
    :default-expanded-keys="expandedKeys"
    :data="data"
    :props="defaultProps"
    @node-click="handleNodeClick"
    :expand-on-click-node="false"
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
      this.$confirm('This will permanently delete the file. Continue?', 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning',
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl(`/product/category/delete`),
            method: 'post',
            data: [data.catId],
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 500,
                onClose: () => {
                  this.expandedKeys = [node.parent.data.catId]
                  this.loadCategory()
                },
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: 'Delete canceled',
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

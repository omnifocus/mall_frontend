<template>
  <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  data() {
    return {
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
  },
  mounted() {
    console.log('loading....')
    this.loadCategory()
  },
}
</script>

<style scoped></style>

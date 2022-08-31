<template>
  <div>
    <el-form :inline="true" class="demo-form-inline" :model="cForm">
      <el-form-item label="一级分类">
        <el-select v-model="cForm.category1Id" placeholder="请选择" @change="btnHandel">
          <el-option v-for="item1 in list1" :key="item1.id" :label="item1.name" :value="item1.id" />
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select v-model="cForm.category2Id" placeholder="请选择" @change="btnHandel2">
          <el-option v-for="item2 in list2" :key="item2.id" :label="item2.name" :value="item2.id" />
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select v-model="cForm.category3Id" placeholder="请选择" @change="btnHandel3">
          <el-option v-for="item3 in list3" :key="item3.id" :label="item3.name" :value="item3.id" />
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'CategorySelect',
  data() {
    return {
      list1: [],
      list2: [],
      list3: [],
      cForm: {
        category1Id: '',
        category2Id: '',
        category3Id: ''
      }
    }
  },
  mounted() {
    this.getCateList()
  },
  methods: {
    async  getCateList() {
      const res = await this.$API.attr.reqCategory1List()
      if (res.code === 200) {
        this.list1 = res.data
      }
    },
    // 一级分类
    async btnHandel(val1) {
      this.list2 = []
      this.list3 = []
      this.cForm.category2Id = ''
      this.cForm.category3Id = ''

      this.$emit('category', { category: val1, label: 1 })
      const res = await this.$API.attr.reqCategory2List(val1)
      if (res.code === 200) {
        this.list2 = res.data
      }
    },
    // 二级分类
    async btnHandel2(val2) {
      this.list3 = []
      this.cForm.category3Id = ''
      this.$emit('category', { category: val2, label: 2 })
      const res = await this.$API.attr.reqCategory3List(val2)
      if (res.code === 200) {
        this.list3 = res.data
      }
    },
    // 三级分类
    btnHandel3(val3) {
      this.$emit('category', { category: val3, label: 3 })
    }
  }

}
</script>

<style lang="scss" scoped>

</style>

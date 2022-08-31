<template>
  <div>
    <el-card style="margin: 20px 0px">
      <CategorySelect @category="category" />
    </el-card>

    <el-card>
      <div v-show="isShow">
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin: 10px 0px"
          :disabled="!category3Id"
          @click="isShow = false"
        >添加属性</el-button>
        <el-table border :data="list">
          <el-table-column
            label="序号"
            type="index"
            width="80px"
            align="center"
          />
          <el-table-column label="属性名称" width="150px" prop="attrName" />
          <el-table-column label="属性值列表">
            <template slot-scope="{ row }">
              <el-tag
                v-for="item in row.attrValueList"
                :key="item.id"
                type="success"
                style="margin: 0px 10px"
              >{{ item.valueName }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180px">
            <template slot-scope="{ row }">
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-edit"
                @click="updataAttr(row)"
              >修改</el-button>
              <el-button
                type="danger"
                size="mini"
                icon="el-icon-delete"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="!isShow">
        <el-form ref="form" :model="attrInfo" label-width="80px">
          <el-form-item label="属性名">
            <el-input v-model="attrInfo.attrName" placeholder="请输入属性名" />
          </el-form-item>
        </el-form>
        <el-button type="primary" icon="el-icon-plus" :disabled="!attrInfo.attrName" @click="addAttr">添加属性值</el-button>
        <el-button @click="cancelBtn">取消</el-button>
        <el-table border style="margin: 10px 0px" :data="attrInfo.attrValueList">
          <el-table-column label="序号" type="index" width="80px" align="center" />
          <el-table-column label="属性值名称" align="center">
            <template slot-scope="{row,$index}">
              <el-input v-if="row.flag" ref="$index" v-model="row.valueName" placeholder="请输入属性值名称" size="mini" @blur="addFlag(row)" @keyup.native.enter="addFlag(row)" />
              <span v-else @click="kangkang(row,$index)">{{ row.valueName }}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center">

            <template slot-scope="{row,$index}">
              <!-- 这是气泡确认框 -->
              <el-popconfirm :title="`确定要删除${row.valueName}吗?`" @onConfirm="queding($index)">
                <el-button slot="reference" type="danger" icon="el-icon-delete" />
              </el-popconfirm>
            </template>

          </el-table-column>
        </el-table>
        <el-button type="primary" @click="addOrUpdateAttr">保存</el-button>
        <el-button @click="isShow = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'
export default {
  name: 'Attr',
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      list: [],
      isShow: true,
      attrInfo: {
        attrName: '',
        attrValueList: [],
        categoryId: '',
        categoryLevel: ''
      }
    }
  },
  methods: {
    category({ category, label }) {
      if (label === 1) {
        this.category1Id = category
        this.category2Id = ''
        this.category3Id = ''
      } else if (label === 2) {
        this.category2Id = category
        this.category3Id = ''
      } else {
        this.category3Id = category
        this.getList()
      }
    },
    async getList() {
      const { category1Id, category2Id, category3Id } = this
      const res = await this.$API.attr.reqAttrList(category1Id, category2Id, category3Id)
      if (res.code === 200) {
        this.list = res.data
      }
    },
    addAttr() {
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: '',
        flag: true
      })
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus()
      })
    },
    cancelBtn() {
      this.isShow = true
      this.attrInfo = {
        attrName: '',
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: ''
      }
    },
    updataAttr(row) {
      this.isShow = false
      this.attrInfo = cloneDeep(row)
      this.attrInfo.attrValueList.forEach(item => {
        this.$set(item, 'flag', false)
      })
    },
    addFlag(row) {
      console.log(row)
      if (row.valueName.trim() === '') {
        this.$message('输入不正确')
        return
      }
      const a = this.attrInfo.attrValueList.some(item => {
        if (row !== item) {
          return row.valueName === item.valueName
        }
      })
      if (a) return
      row.flag = false
    },
    kangkang(row, index) {
      row.flag = true
      this.$nextTick(() => {
        this.$refs[index].focus()
      })
    },
    queding(val) {
      this.attrInfo.attrValueList.splice(val, 1)
    },
    async addOrUpdateAttr() {
      // 整理参数,然后把参数传给接口,得到服务器返回的结果
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(item => {
        if (item.valueName !== '') {
          return
        }
      })
      try {
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo)
        this.$message({
          type: 'success',
          message: '保存数据成功'
        })
        this.getList()
        this.isShow = true
      } catch (error) {
        console.log(error)
      }
    }
  }
}
</script>

<style lang="scss" scoped></style>

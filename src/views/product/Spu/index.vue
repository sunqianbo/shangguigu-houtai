<template>
  <div>
    <el-card style="margin:20px 0px">
      <CategorySelect :show="scence !== 0" @category="category" />
    </el-card>
    <!-- 展示列表的结构 -->
    <el-card>
      <div v-show="scence===0">
        <el-button type="primary" icon="el-icon-plus" style="margin:20px 0px" :disabled="!category3Id" @click="addSpu">添加SPU</el-button>
        <el-table style="width: 100%" border :data="records">
          <el-table-column prop="prop" label="序号" width="80px" type="index" align="center" />
          <el-table-column prop="spuName" label="SPU名称" width="width" align="center" />
          <el-table-column prop="description" label="SPU描述" width="width" align="center" />
          <el-table-column prop="prop" label="操作" width="width" align="center">
            <template slot-scope="{row}">
              <el-button type="success" icon="el-icon-plus" @click="addsku(row)" />
              <el-button type="warning" icon="el-icon-edit" @click="updateSpu(row)" />
              <el-button type="info" icon="el-icon-info" @click="handel(row)" />
              <el-popconfirm :title="`确定要删除${row.spuName}吗?`" style="margin:0px 10px" @onConfirm="queding(row.id)">
                <el-button slot="reference" type="danger" icon="el-icon-delete" />
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          style="text-align:center;margin:20px 0px"
          :current-page="page"
          :page-sizes="[3,5,10]"
          :page-size="limit"
          layout="prev, pager, next,jumper,->,sizes,total"
          :total="total"
          @current-change="handleCurrentChange"
        />
      </div>
      <SpuForm v-show="scence===1" ref="spu" @changeScene="changeScene" />
      <SkuForm v-show="scence===2" ref="sku" @changeScenes="changeScenes" />
    </el-card>
    <el-dialog :title="`${spu.spuName}的SKU列表`" :visible.sync="dialogVisible" width="width" :before-close="close">
      <el-table v-loading="loading" :data="skuList" style="width: 100%">
        <el-table-column prop="skuName" label="名称" width="width" />
        <el-table-column prop="price" label="价格" width="width" />
        <el-table-column prop="weight" label="重量" width="width" />
        <el-table-column label="默认图片" width="width">
          <template slot-scope="{row}">
            <img :src="row.skuDefaultImg" style="width:100px ; height:100px">
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SkuForm from './SkuForm'
import SpuForm from './SpuForm'
export default {
  name: 'Spu',
  components: {
    SkuForm,
    SpuForm
  },
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      page: 1,
      limit: 3,
      records: [],
      total: 0,
      scence: 0, // 控制三个页面的切换
      dialogVisible: false,
      spu: {},
      skuList: [], // 存储的是SKU列表的数据
      loading: true
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
        this.getSpuList()
      }
    },
    // 添加SPU
    addSpu() {
      this.scence = 1
      this.$refs.spu.addSpu(this.category3Id)
    },
    // 修改SPU
    updateSpu(row) {
      this.scence = 1
      this.$refs.spu.initSpuData(row)
    },
    // 获取spu列表数据的接口
    async getSpuList() {
      const { page, limit, category3Id } = this
      const res = await this.$API.spu.reqSpuList(page, limit, category3Id)
      if (res.code === 200) {
        this.records = res.data.records
        this.total = res.data.total
      }
    },
    handleCurrentChange(val) {
      this.page = val
      this.getSpuList()
    },
    changeScene({ scence, flag }) {
      // flag这个形参为了区分保存按钮是添加还是修改
      // 切换结构（场景）
      this.scence = scence
      // 子组件通知父组件切换场景，需要再次获取SPU列表的数据进行展示
      if (flag === '修改') {
        this.getSpuList(this.page)
      } else {
        this.getSpuList()
      }
    },
    async queding(val) {
      const res = await this.$API.spu.reqDeleteSpu(val)
      if (res.code === 200) {
        this.$message({
          type: 'success',
          message: '删除数据成功'
        })
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1)
      }
    },
    // 添加sku
    addsku(row) {
      this.scence = 2
      this.$refs.sku.getData(this.category1Id, this.category2Id, row)
    },
    changeScenes(val) {
      this.scence = val
    },
    async  handel(row) {
      this.dialogVisible = true
      this.spu = row
      const res = await this.$API.sku.reqSkuList(row.id)

      if (res.code === 200) {
        this.skuList = res.data
        this.loading = false
      }
    },
    close() {
      this.skuList = []
      this.dialogVisible = false
      this.loading = true
    }
  }
}
</script>

<style lang="scss" scoped>

</style>

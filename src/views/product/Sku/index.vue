<template>
  <div>
    <el-table :data="records" style="width: 100%" border>
      <el-table-column label="序号" width="80px" type="index" />
      <el-table-column prop="skuName" label="名称" width="width" />
      <el-table-column prop="skuDesc" label="描述" width="width" />
      <el-table-column prop="prop" label="默认图片" width="130px">
        <template slot-scope="{row}">
          <img :src="row.skuDefaultImg" style="width:100px;height:100px">
        </template>
      </el-table-column>
      <el-table-column prop="weight" label="重量(KG)" width="100px" />
      <el-table-column prop="price" label="价格(元)" width="100px" />
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{row}">
          <el-button v-if="row.isSale === 0" type="danger" icon="el-icon-sort-down" size="mini" @click="done(row)">下架</el-button>
          <el-button v-else type="success" icon="el-icon-sort-up" size="mini" @click="up(row)">上架</el-button>
          <el-button type="primary" icon="el-icon-edit" size="mini" />
          <el-button type="info" icon="el-icon-info" size="mini" @click="getSkuInfo(row)" />
          <el-button type="danger" icon="el-icon-delete" size="mini" />
        </template>
      </el-table-column>
    </el-table>
    <!--  @size-change="handleSizeChange"  @current-change="handleCurrentChange" -->
    <el-pagination
      style="text-align: center;margin-top:20px"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      layout=" prev, pager, next, jumper , -> ,sizes,total"
      :total="total"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
    />
    <el-drawer :title="sku.skuName" :visible.sync="show" :show-close="false" :before-close="handleClose">
      <el-row>
        <el-col :span="5"><div class="grid-content bg-purple" />名称</el-col>
        <el-col :span="16"><div class="grid-content bg-purple-light" />{{ skuInfo.skuName }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5"><div class="grid-content bg-purple" />描述</el-col>
        <el-col :span="16"><div class="grid-content bg-purple-light" />{{ skuInfo.skuDesc }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5"><div class="grid-content bg-purple" />价格</el-col>
        <el-col :span="16"><div class="grid-content bg-purple-light" />{{ skuInfo.price }}元</el-col>
      </el-row>
      <el-row>
        <el-col :span="5"><div class="grid-content bg-purple" />平台属性</el-col>
        <el-col :span="16"><div class="grid-content bg-purple-light" />
          <el-tag v-for="item in skuInfo.skuAttrValueList" :key="item.id" type="success" style="margin:5px 10px">
            {{ item.attrName }}
          </el-tag>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="5"><div class="grid-content bg-purple" />商品图片</el-col>
        <el-col :span="16"><div class="grid-content bg-purple-light" />
          <el-carousel height="150px">
            <el-carousel-item v-for="v in skuInfo.skuImageList " :key="v.id">
              <img :src="v.imgUrl" alt="">
            </el-carousel-item>
          </el-carousel>
        </el-col>
      </el-row>
    </el-drawer>
  </div>
</template>

<script>
export default {
  name: 'Sku',
  data() {
    return {
      page: 1,
      limit: 10,
      records: [],
      total: 0,
      skuInfo: {}, // 存储SKU信息
      show: false,
      sku: {}
    }
  },
  mounted() {
    this.getSkuList()
  },
  methods: {
    async getSkuList() {
      const res = await this.$API.sku.reqSkuList2(this.page, this.limit)
      if (res.code === 200) {
        this.records = res.data.records
        this.total = res.data.total
      }
    },
    handleCurrentChange(val) {
      this.page = val
      this.getSkuList()
    },
    handleSizeChange(val) {
      this.limit = val
      this.getSkuList()
    },
    async done(row) {
      const res = await this.$API.sku.reqCancel(row.id)
      if (res.code === 200) {
        this.$message({
          type: 'success',
          message: '下架成功'
        })
        row.isSale = 1
      }
    },
    async up(row) {
      const res = await this.$API.sku.reqSale(row.id)
      if (res.code === 200) {
        this.$message({
          type: 'success',
          message: '上架成功'
        })
        row.isSale = 0
      }
    },
    // 获取SKU详情的方法
    async getSkuInfo(sku) {
      this.sku = sku
      // 展示抽屉
      this.show = true
      // 获取SKU数据
      const result = await this.$API.sku.reqSkuById(sku.id)
      if (result.code === 200) {
        this.skuInfo = result.data
      }
    }
  }
}
</script>

<style lang="scss" scoped>
   .el-col.el-col-5{
     font-size: 18px;
     text-align: right;
   }
    .el-col{
     margin:10px 10px;
   }
</style>

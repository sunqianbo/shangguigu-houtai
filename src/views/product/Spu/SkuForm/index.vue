<template>
  <div>
    <el-form label-width="80px" :model="skuInfo">
      <el-form-item label="SKU名称">
        {{ spu.spuName }}
      </el-form-item>
      <el-form-item label="SPU名称">
        <el-input v-model="skuInfo.skuName" placeholder="sku名称" />
      </el-form-item>
      <el-form-item label="价格(元)">
        <el-input v-model="skuInfo.price" />
      </el-form-item>
      <el-form-item label="重量(千克)">
        <el-input v-model="skuInfo.weight" placeholder="重量(千克)" />
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input v-model="skuInfo.skuDesc" type="textarea" />
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form ref="form" :inline="true" label-width="80px">
          <el-form-item v-for="item in attrInfoList" :key="item.id" :label="item.attrName">
            <el-select v-model="item.attrIdAndValueId" placeholder="请选择">
              <el-option v-for="itemvalue in item.attrValueList" :key="itemvalue.id" :label="itemvalue.valueName" :value="`${item.id}:${itemvalue.id}`" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form ref="form" :inline="true" label-width="80px">
          <el-form-item v-for="sale in spuSaleAttrList" :key="sale.id" :label="sale.saleAttrName">
            <el-select v-model="sale.attrIdAndValueId" placeholder="请选择">
              <el-option v-for="salevalue in sale.spuSaleAttrValueList" :key="salevalue.id" :label="salevalue.saleAttrValueName" :value="`${sale.id}:${salevalue.id}`" />
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table style="width: 100%" border :data="spuImageList" @selection-change="handleSelectionChange">
          <el-table-column prop="prop" type="selection" width="80px" align="center" />
          <el-table-column prop="prop" label="图片" width="width">
            <template slot-scope="{row}">
              <img :src="row.imgUrl" style="width:100px ; height:100px">
            </template>
          </el-table-column>
          <el-table-column prop="imgName" label="名称" width="width" />
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{row}">
              <el-button v-if="row.isDefault === 0" type="primary" @click="changeDefault(row)">设置默认</el-button>
              <el-button v-else>默认</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="save">保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>

  </div>

</template>

<script>
export default {
  name: 'SkuForm',
  data() {
    return {
      // 存储图片的信息
      spuImageList: [],
      // 存储销售是属性
      spuSaleAttrList: [],
      // 存储平台属性的数据
      attrInfoList: [],
      spu: {},
      // 收集sku数据的字段
      skuInfo: {
        // 第一类收集的数据：父组件给的数据
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        // 第二类：需要通过数据双向绑定v-model收集
        skuName: '',
        price: 0,
        weight: '',
        skuDesc: '',
        // 第三类：需要自己书写代码
        // 默认图片
        skuDefaultImg: '',
        // 收集图片的字段
        skuImageList: [
          // {
          //   imgName: "string",
          //   imgUrl: "string",
          //   isDefault: "string",
          //   spuImgId: 0,
          // },
        ],
        // 平台属性
        skuAttrValueList: [
          // {
          //   attrId: 0,
          //   valueId: 0,
          // },
        ],
        // 销售属性
        skuSaleAttrValueList: [
          // {
          //   saleAttrId: 0,
          //   saleAttrValueId: 0,
          // },
        ]

      },
      // 收集图片的数据字段:但是需要注意，收集的数据目前缺少isDefault字段，将来提交给服务器数据的时候，需要整理参数
      imageList: []
    }
  },
  methods: {
    async  getData(category1Id, category2Id, row) {
      this.skuInfo.category3Id = row.category3Id
      this.skuInfo.spuId = row.id
      this.skuInfo.tmId = row.tmId
      this.spu = row
      // 获取图片的数据
      const res1 = await this.$API.sku.reqSpuImageLIst(row.id)
      if (res1.code === 200) {
        const kangkang = res1.data
        kangkang.forEach(v => {
          v.isDefault = 0
        })
        this.spuImageList = kangkang
      }
      // 获取销售属性的数据
      const res2 = await this.$API.sku.reqSpuSaleAttrList(row.id)
      if (res2.code === 200) {
        this.spuSaleAttrList = res2.data
      }
      // 获取平台属性的数据
      const res3 = await this.$API.sku.reqAttrInfoList(category1Id, category2Id, row.category3Id)
      if (res3.code === 200) {
        this.attrInfoList = res3.data
      }
    },
    handleSelectionChange(val) {
      this.imageList = val
    },
    // 排他思像
    changeDefault(row) {
      this.spuImageList.forEach(v => {
        v.isDefault = 0
      })
      row.isDefault = 1
    },
    cancel() {
      this.$emit('changeScenes', 0)
      Object.assign(this._data, this.$options.data())
    },
    async save() {
      // 平台属性
      const arr1 = []
      this.attrInfoList.forEach(v => {
        if (v.attrIdAndValueId) {
          const [attrId, valueId] = v.attrIdAndValueId.split(':')
          arr1.push({ attrId, valueId })
        }
      })
      this.skuInfo.skuAttrValueList = arr1
      // 销售属性
      const arr2 = []
      this.spuSaleAttrList.forEach(v => {
        if (v.attrIdAndValueId) {
          const [saleAttrId, saleAttrValueId] = v.attrIdAndValueId.split(':')
          arr2.push({ saleAttrId, saleAttrValueId })
        }
      })
      this.skuInfo.skuSaleAttrValueList = arr2
      this.skuInfo.skuImageList = this.imageList.map(v => {
        return {
          imgName: v.imgName,
          imgUrl: v.imgUrl,
          isDefault: v.isDefault,
          spuImgId: v.id
        }
      })
      const res = await this.$API.sku.reqAddSku(this.skuInfo)
      if (res.code === 200) {
        this.$message({ type: 'success', message: '添加SKU成功' })
        this.$emit('changeScenes', 0)
      }
    }
  }
}
</script>

<style>
  .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 150px;
    margin: 0;
  }

  .el-carousel__item:nth-child(2n) {
     background-color: #99a9bf;
  }

  .el-carousel__item:nth-child(2n+1) {
     background-color: #d3dce6;
  }
</style>

<style scoped>
   .el-row .el-col-5{
      font-size:18px;
      text-align:right;
   }
   .el-col{
     margin:10px 10px;
   }

   >>>.el-carousel__button{
    width:10px;
    height:10px;
    background:red;
    border-radius:50%;
  }
</style>

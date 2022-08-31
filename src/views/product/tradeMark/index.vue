<template>
  <div>
    <el-button
      type="primary"
      style="margin: 10px 0px"
      icon="el-icon-plus"
      @click="addShowDialog"
    >添加</el-button>
    <el-table style="width: 100%" border :data="list">
      <el-table-column type="index" label="序号" width="80px" align="center" />
      <el-table-column
        prop="tmName"
        label="品牌名称"
        width="width"
        align="center"
      />
      <el-table-column
        prop="prop"
        label="品牌Logo"
        width="width"
        align="center"
      >
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px">
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width" align="center">
        <template slot-scope="{ row }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="putShowDialog(row)"
          >修改</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteTradeMark(row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="margin-top: 20px; textAlign: center"
      layout="prev, pager, next, jumper,->, sizes,total"
      :total="total"
      :page-size="limit"
      :current-page="page"
      :pager-count="7"
      :page-sizes="[3, 5, 10]"
      @current-change="getPageJump"
    />
    <el-dialog :title="tmForm.id?'修改品牌':'添加品牌'" :visible.sync="dialogFormVisible">
      <el-form ref="refTmForm" :model="tmForm" :rules="refTmForm">
        <el-form-item label="活动名称" prop="tmName">
          <el-input v-model="tmForm.tmName" style="width:80%" />
        </el-form-item>
        <el-form-item label="品牌logo" prop="logoUrl">
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" />
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancelBtn">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'TradeMark',
  data() {
    const validatePass = (rule, value, callback) => {
      if (value.length < 2 || value.length > 10) {
        return callback('品牌名称2-10位')
      } else {
        return callback()
      }
    }
    return {
      page: 1, // 分页器第几页
      limit: 3, // 当前页展示的数据
      list: [],
      total: 0,
      dialogFormVisible: false,
      tmForm: {
        tmName: '',
        logoUrl: ''
      },
      refTmForm: {
        tmName: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
          { validator: validatePass, trigger: 'blur' }
        ],
        logoUrl: [{ required: true, message: '请选择品牌的图片' }]
      }
    }
  },
  mounted() {
    this.getPageList()
  },
  methods: {
    async getPageList() {
      const { page, limit } = this
      const res = await this.$API.trademark.reqTradeMarkList(page, limit)
      if (res.code === 200) {
        this.list = res.data.records
        this.total = res.data.total
      }
    },
    getPageJump(val) {
      this.page = val
      this.getPageList()
    },
    // 添加时弹出对话框
    addShowDialog() {
      this.dialogFormVisible = true
      this.tmForm = {
        tmName: '',
        logoUrl: ''
      }
    },
    // 修改时弹出对话框
    putShowDialog(row) {
      this.dialogFormVisible = true
      this.tmForm = { ...row }
    },
    handleAvatarSuccess(res, file) {
      console.log(res)
      console.log(file)
      this.tmForm.logoUrl = res.data
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    addOrUpdateTradeMark() {
      this.$refs.refTmForm.validate(async(success) => {
        if (success) {
          const res = await this.$API.trademark.reqAddOrUpdateTradeMark(this.tmForm)
          if (res.code === 200) {
            this.$message({
              type: 'success',
              message: this.tmForm.id ? '修改品牌成功' : '添加品牌成功'
            })
            this.dialogFormVisible = false
            this.getPageList(!this.tmForm.id ? this.page = 1 : this.page)
          }
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    cancelBtn() {
      this.dialogFormVisible = false
    },
    // 删除按钮
    deleteTradeMark(row) {
      // 弹框
      this.$confirm(`你确定删除${row.tmName}?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        // 当用户点击确定按钮的时候会出发
        // 向服务器发请求
        const result = await this.$API.trademark.reqDeleteTradeMark(row.id)
        // 如果删除成功
        if (result.code === 200) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          // 再次获取品牌列表数据
          this.getPageList(this.list.length > 1 ? this.page : this.page - 1)
        }
      })
        .catch(() => {
          // 当用户点击取消按钮的时候会触发
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    }
  }

}
</script>

<style lang="scss" scoped>
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>

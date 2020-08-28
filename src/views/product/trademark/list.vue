<template>
  <div>
    <el-button type="primary" icon="el-icon-plus" size="medium" @click="trademarkAdd">添加</el-button>

    <el-table :data="trademarkList" border style="width: 100%; margin: 20px 0">
      <el-table-column label="序号" type="index" width="80" />
      <el-table-column prop="tmName" label="品牌名称" width="" />
      <el-table-column prop="logoUrl" label="品牌Logo" width="">
        <template slot-scope="{row}">
          <img :src="row.logoUrl" height="50px">
        </template>
      </el-table-column>
      <el-table-column prop="" label="操作" width="">
        <template slot-scope="scope">
          <el-button
            size="mini"
            @click="trademarkModify(scope.row)"
          >修改</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="trademarkDelete(scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      :current-page="page"
      :total="records.total"
      :page-size="limit"
      :page-sizes="pageSize"
      :background="true"
      :pager-count="5"
      layout=" prev, pager, next, ->, jumper, total, sizes"
      @size-change="handleSizeChange"
      @current-change="getTrademarkList"
    />

    <el-dialog :title="form.id ? '修改品牌' : '添加品牌'" :visible.sync="dialogVisible">
      <el-form :model="form">
        <el-form-item label="品牌名称" label-width="">
          <el-input v-model="form.tmName" autocomplete="off" class="el-input" />
        </el-form-item>
        <el-form-item label="品牌图片" label-width="">
          <el-upload
            class="upload-demo"
            drag
            action="/dev-api/admin/product/fileUpload"
            :on-success="uploadSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <i class="el-icon-upload" />
            <div v-if="form.logoUrl === undefined" class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
            <div v-else-if="form.logoUrl"><img :src="form.logoUrl"></div>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="trademarkddOrupdate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Trademark',
  data() {
    return {
      page: 1,
      limit: 3,
      records: {},
      pageSize: [5, 10, 20, 50],
      trademarkList: [],

      dialogVisible: false,
      form: {}
    }
  },
  // computed: {
  //   trademarkList() {
  //     return this.records.records
  //   }
  // },
  mounted() {
    this.getTrademarkList()
  },
  methods: {
    handleSizeChange(limit) {
      this.limit = limit
      this.getTrademarkList()
    },
    async getTrademarkList(page = 1) {
      this.page = page
      const result = await this.$API.trademark.getTrademark(this.page, this.limit)
      if (result.code === 200) {
        this.trademarkList = result.data.records
        this.records = result.data
      }
    },
    // 点击添加按钮，弹出对话框
    trademarkAdd() {
      // 为了解决添加后取消，再点击添加数据仍然存在的bug
      this.form = {}
      this.dialogVisible = true
    },
    // 点击修改按钮，弹出对话框（和添加）
    trademarkModify(row) {
      // const { id, logoUrl, tmName } = row
      // this.form = { id, logoUrl, tmName }
      // this.form = row
      // form和row(trademarkList当中的对象) 是同一个对象（地址一样） 修改form就相当于修改row
      // 浅拷贝一个新的对象  必须和row不是同一个
      this.form = { ...row }
      this.dialogVisible = true
    },
    // 上传成功后的处理
    uploadSuccess(res) {
      // console.log(res,file) // res是上传成功后图片所在服务器上的路径，真是的线上路径，我们要的是它
      // console.log(URL.createObjectURL(file.raw)) //本地图片的路径，我们要的不是它
      // 真正的收集上传成功的图片路径
      this.form.logoUrl = res.data
    },
    // 上传之前的处理
    // 其实就是让你在上传之前对文件进行一下子限制
    beforeAvatarUpload(file) {
      const fileTypes = ['image/jpeg', 'image/png']
      const isJPGOrPNG = fileTypes.indexOf(file.type) !== -1
      const isLt500K = file.size / 1024 < 500

      if (!isJPGOrPNG) {
        this.$message.error('上传头像图片只能是 JPG 格式或者 PNG格式!')
      }
      if (!isLt500K) {
        this.$message.error('上传头像图片大小不能超过 500KB!')
      }
      return isJPGOrPNG && isLt500K // 只有都为true才符合我们的限制需求
    },
    async trademarkddOrupdate() {
      const trademark = this.form
      console.log(this.form)
      // const { tmName, logoUrl } = this.form
      // if (tmName && logoUrl) {
      try {
        const result = await this.$API.trademark.addOrUpdate(trademark)
        if (result.code === 200) {
          // this.$alert("上传成功")
          // 1.关闭dialog
          this.dialogVisible = false
          // 2.重新请求拿列表数据
          this.getTrademarkList()
          // 3. 提示成功
          this.$message.success(`${trademark.id ? '修改 ' : '添加'}成功`)
        } else {
          this.$alert('上传失败')
        }
      } catch (err) {
        // 发送ajax请求失败
        this.$message.error(err.message)
      }
      // }
    },
    // 删除
    async trademarkDelete(row) {
      this.$confirm(`你确定删除${row.tmName}吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        const result = await this.$API.trademark.deleteTrademark(row.id)
        if (result.code === 200) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.getTrademarkList(this.trademarkList.length > 1 ? this.page : this.page - 1)
        } else {
          this.$message({
            type: 'success',
            message: '删除失败!'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '删除已取消'
        })
      })
    }
  }
}
</script>

<style scoped>
.el-input{
  width: 200px;
}
.el-upload__tip{
  margin-left: 150px;
}
</style>

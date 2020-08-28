<template>
  <el-form :inline="true" :model="form" class="demo-form-inline">
    <el-form-item label="一级分类">
      <el-select v-model="form.category1Id" placeholder="活动区域" @change="category1Handler">
        <el-option v-for="category1 in category1List" :key="category1.id" :label="category1.name" :value="category1.id" />
      </el-select>
    </el-form-item>
    <el-form-item label="二级分类">
      <el-select v-model="form.category2Id" placeholder="活动区域" @change="category2Handler">
        <el-option v-for="category2 in category2List" :key="category2.id" :label="category2.name" :value="category2.id" />
      </el-select>
    </el-form-item>
    <el-form-item label="三级分类">
      <el-select v-model="form.category3Id" placeholder="活动区域" @change="category3Handler">
        <el-option v-for="category3 in category3List" :key="category3.id" :label="category3.name" :value="category3.id" />
      </el-select>
    </el-form-item>
  </el-form>
  <!-- <el-form inline>

    {
    "id": 1,
    "name": "图书、音像、电子书刊"
    },

    <el-form-item label="一级分类">
      <el-select
        v-model="category1Id"
        placeholder="请选择"
        :disabled="disabled"
        @change="handleCategory1Change"
      >
        <el-option v-for="c1 in category1List" :key="c1.id" :label="c1.name" :value="c1.id" />
      </el-select>
    </el-form-item>
    <el-form-item label="二级分类">
      <el-select
        v-model="category2Id"
        placeholder="请选择"
        :disabled="disabled"
        @change="handleCategory2Change"
      >
        <el-option v-for="c2 in category2List" :key="c2.id" :label="c2.name" :value="c2.id" />
      </el-select>
    </el-form-item>
    <el-form-item label="三级分类">
      <el-select
        v-model="category3Id"
        placeholder="请选择"
        :disabled="disabled"
        @change="handleCategory3Change"
      >
        <el-option v-for="c3 in category3List" :key="c3.id" :label="c3.name" :value="c3.id" />
      </el-select>
    </el-form-item>
  </el-form> -->
</template>

<script>
export default {
  name: 'CategorySelector',

  data() {
    return {
      // category1List: [], // 一级分类列表
      // category2List: [], // 二级分类列表
      // category3List: [], // 三级分类列表
      // category1Id: '', // 选择的一级分类id
      // category2Id: '', // 选择的二级分类id
      // category3Id: '', // 选择的三级分类id
      // disabled: false, // 是否禁用select
      form: {
        category1Id: '', // 选择的一级分类id
        category2Id: '', // 选择的二级分类id
        category3Id: '' // 选择的三级分类id
      },
      category1List: [], // 一级分类列表
      category2List: [], // 二级分类列表
      category3List: [] // 三级分类列表
    }
  },

  mounted() {
    // 异步获取一级分类列表显示
    this.getCategory1List()
  },

  methods: {
    // 异步获取一级分类列表显示
    async getCategory1List() {
      const result = await this.$API.category.getCategorys1()
      if (result.code === 200) {
        this.category1List = result.data
      }
    },

    // 异步获取二级分类列表显示
    async category1Handler(id) {
      this.form.category2Id = ''
      this.form.category3Id = ''
      this.category2List = []
      this.category3List = []
      // 通知清楚attr列表数据
      this.$emit('searchCategory', { categoryId: id, level: 1 })
      const result = await this.$API.category.getCategorys2(id)
      if (result.code === 200) {
        this.category2List = result.data
      }
    },

    // 异步获取三级分类列表显示
    async category2Handler(id) {
      this.form.category3Id = ''
      this.category3List = []
      // 通知清楚attr列表数据
      this.$emit('searchCategory', { categoryId: id, level: 2 })

      const result = await this.$API.category.getCategorys3(id)
      if (result.code === 200) {
        this.category3List = result.data
      }
    },

    category3Handler(id) {
      this.$emit('searchCategory', { categoryId: id, level: 3 })
    }
    /*
    异步获取一级分类列表显示
    */
    // async getCategory1List() {
    //   const result = await this.$API.category.getCategorys1()
    //   const category1List = result.data
    //   this.category1List = category1List
    // },

    // /*
    // 什么时候分发: change事件回调中
    // 事件名: categoryChange
    // 携带的数据: 分类Id和分类级别(1/2/3)   {categoryId: 12, level: 1/2/3}
    // */

    // /*
    // 选中的一级分类ID发生变化的事件回调
    // */
    // async handleCategory1Change(category1Id) {
    //   // 分发分类ID发生改变的事件
    //   this.$emit('categoryChange', { categoryId: category1Id, level: 1 })

    //   // 重置二/三分类数据
    //   this.category2List = []
    //   this.category2Id = ''
    //   this.category3List = []
    //   this.category3Id = ''

    //   // 请求获取二级分类列表显示
    //   const result = await this.$API.category.getCategorys2(category1Id)
    //   this.category2List = result.data
    // },
    // /*
    // 选中的二级分类ID发生变化的事件回调
    // */
    // async handleCategory2Change(category2Id) {
    //   // 分发分类ID发生改变的事件
    //   this.$emit('categoryChange', { categoryId: category2Id, level: 2 })

    //   // 重置三分类数据
    //   this.category3List = []
    //   this.category3Id = ''

    //   // 请求获取三级分类列表显示
    //   const result = await this.$API.category.getCategorys3(category2Id)
    //   this.category3List = result.data
    // },
    // /*
    // 选中的三级分类ID发生变化的事件回调
    // */
    // handleCategory3Change(category3Id) {
    //   // 分发分类ID发生改变的事件
    //   this.$emit('categoryChange', { categoryId: category3Id, level: 3 })
    // }
  }
}
</script>

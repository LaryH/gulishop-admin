<template>
  <div>
    <el-card>
      <CategorySelector @searchCategory="searchCategory" />
    </el-card>
    <el-card style="margin-top:20px">
      <div v-show="isShowList">
        <el-button type="primary" icon="el-icon-plus" :disabled="!category3Id" @click="showAddDiv">添加属性</el-button>
        <el-table border style="width:100%" :data="attrList">
          <el-table-column label="序号" type="index" width="80" />
          <el-table-column prop="attrName" label="属性名称" width="150" />
          <el-table-column prop="" label="属性值列表">
            <template slot-scope="{row}">
              <div><el-tag v-for="attr in row.attrValueList" :key="attr.id" type="success">
                {{ attr.valueName }}
              </el-tag></div>
            </template>
          </el-table-column>
          <el-table-column prop="" label="操作" width="150">
            <template slot-scope="{row}">
              <HintButton type="warning" icon="el-icon-edit" @click="showUpdateDiv(row)" />
              <HintButton type="danger" icon="el-icon-delete" />
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div v-show="!isShowList">
        <el-form ref="form" :model="form" :inline="true">
          <el-form-item label="属性名称">
            <el-input v-model="form.attrName" placeholder="请输入属性名称" />
          </el-form-item>
        </el-form>
        <el-button type="primary" icon="el-icon-plus" :disabled="!form.attrName" @click="addAttrValue">添加属性值</el-button>
        <el-button @click="isShowList=true">取消</el-button>
        <el-table border style="margin:20px 0" :data="form.attrValueList">
          <el-table-column label="序号" type="index" width="80" />
          <el-table-column label="属性值名称" prop="valueName">
            <template slot-scope="{row}">
              <el-input v-model="row.valueName" placeholder="请输入属性值名称" />
            </template>
          </el-table-column>
          <el-table-column label="操作" />
        </el-table>
        <el-button type="primary">保存</el-button>
        <el-button @click="isShowList=true">取消</el-button>
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
      attrList: [],
      isShowList: true,

      // {
      //   "attrName": "string",
      //   "attrValueList": [
      //     {
      //       "attrId": 0,
      //       "id": 0,
      //       "valueName": "string"
      //     }
      //   ],
      //   "categoryId": 0,
      //   "categoryLevel": 0,
      //   "id": 0
      // }

      form: {
        attrName: '',
        attrValueList: [],
        categoryId: '',
        categoryLevel: 3
      }
    }
  },
  methods: {
    searchCategory({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId
        this.category2Id = ''
        this.category3Id = ''
        this.attrList = []
      } else if (level === 2) {
        this.category2Id = categoryId
        this.category3Id = ''
        this.attrList = []
      } else {
        this.category3Id = categoryId
        // 发请求拿数据
        this.getAttrInfoList()
      }
    },
    async getAttrInfoList() {
      const result = await this.$API.attr.getAttrInfoList(this.category1Id, this.category2Id, this.category3Id)
      if (result.code === 200) {
        this.attrList = result.data
      }
    },
    showAddDiv() {
      this.form = {
        attrName: '',
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: 3
      }
      this.isShowList = false
    },
    addAttrValue() {
      this.form.attrValueList.push({
        attrId: this.form.id, // 添加id为undefined,修改有id
        valueName: ''
      })
    },
    showUpdateDiv(row) {
      this.isShowList = false
      // 展示同时修改数据时,需要用深浅拷贝,对象或数组中包含对象时,需要深拷贝
      // this.form = {...row}
      // this.form = JSON.parse(JSON.stringify(row))
      this.form = cloneDeep(row)
    }
  }
}
</script>


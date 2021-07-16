<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="4">
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否删除模板 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>

        <!-- 排序模板 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level==0" size="mini">一级</el-tag>
          <el-tag v-if="scope.row.cat_level==1" size="mini" type="success">二级</el-tag>
          <el-tag v-if="scope.row.cat_level==2" size="mini" type="warning">三级</el-tag>
        </template>

        <!-- 操作模板 -->
        <template slot="opt">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
      ></el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        ref="addCateFormRef"
        :model="addCateForm"
        :rules="addCateFormRules"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父级分类">
          <el-cascader
            expandTrigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChangeed"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      addCateDialogVisible: false,
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      cateList: [],
      parentCateList: [],
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
          width: '200px',
        },
        {
          label: '是否删除',
          prop: 'cat_deleted',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          prop: 'order',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          prop: 'opt',
          type: 'template',
          template: 'opt'
        },
      ],
      addCateForm: {
        cat_id: '',
        cat_pid: 0,
        cat_name: '',
        cat_level: 1
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称 ', trigger: 'blur' },
        ],
        cat_id: [
          { required: true, message: '请输入分类名称 ', trigger: 'blur' },
        ],
      },
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      selectedKeys: [],

    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    handleSizeChange (newSize) {
      console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateList = res.data.result
    },
    showAddCateDialog () {
      this.getParentCateList()
      console.log(1234)
      this.addCateDialogVisible = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error(res.meta.status)
      this.parentCateList = res.data
    },
    parentCateChangeed () {
      console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        // 父级分类的id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 当前分类的层级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的id
        this.addCateForm.cat_pid = 0
        // 当前分类的层级
        this.addCateForm.cat_level = 0
      }
    },
    addCate () {
      console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.addCateDialogVisible = false
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }
  },
}
</script>
<style lang="less" scoped>
</style>
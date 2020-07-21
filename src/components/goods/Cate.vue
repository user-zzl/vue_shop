<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加分类按钮区 -->
        <el-row>
            <el-col>
                <el-button
                    type="primary"
                    @click="showAddCateDialog"
                >添加分类</el-button>
            </el-col>
        </el-row>
        <!-- 表格 -->
        <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        class="treeTable">
          <!-- '是否有效' 栏 -->
          <template slot="isok" slot-scope="scope">
              <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: #67C23A"></i>
              <i class="el-icon-error" v-else style="color: #F56C6C"></i>
          </template>
          <!-- '排序' 栏 -->
          <template slot="order" slot-scope="scope">
              <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
              <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
              <el-tag type="warning" size="mini" v-else>三级</el-tag>
          </template>
          <!-- '操作' 栏 -->
          <template slot="opt" slot-scope="scope">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCateDialog(scope.row.cat_id)">编辑</el-button>
              <el-button type="primary" icon="el-icon-delete" size="mini" @click="removeCate(scope.row.cat_id)">删除</el-button>
          </template>
        </tree-table>
        <!-- 分页区 -->
        <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
        </el-pagination>
    </el-card>
    <!-- 弹框区 -->
    <!-- 添加分类弹框 -->
    <el-dialog
    title="提示"
    :visible.sync="addCateDialogVisible"
    width="50%"
    @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm"
      :rules="addCateFormRules"
      ref="addCateFormRef"
      label-width="100px">
          <el-form-item label="分类名称：" prop="cat_name">
              <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类：" prop="cat_level">
              <!-- 级联选择器 -->
              <el-cascader
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cacsaderProps"
              @change="parentCateChanged"
              clearable></el-cascader>
          </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
      <!-- 编辑分类弹框 -->
      <el-dialog
      title="修改用户"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClosed">
        <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="70px">
          <el-form-item label="分类名称">
            <el-input v-model="editCateForm.cat_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editCateInfo">确 定</el-button>
        </span>
      </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类数据列表，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }, {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok'
      }, {
          label: '排序',
          type: 'template',
          template: 'order'
      }, {
          label: '操作',
          type: 'template',
          template: 'opt'
      }],
      // 控制添加分类弹框显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
          // 将要添加的分类的名称
          cat_name: '',
          // 分类的等级， 默认要添加的是1级分类
          cat_level: '0',
          // 父级分类的ID
          cat_pid: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
          cat_name: [
              { required: true, message: '请输入分类名称', trigger: 'blur' }
          ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cacsaderProps: {
          // 次级菜单的展开方式
          expandTrigger: 'hover',
          // 指定选项的值
          value: 'cat_id',
          // 指定选项标签
          label: 'cat_name',
          // 指定选项的子选项
          children: 'children',
          // 允许选中任意选项
          checkStrictly: 'true'
      },
      // 选中父级分类的ID数组
      selectedKeys: [],
      // 控制编辑分类框的显示与隐藏
      editCateDialogVisible: false,
      // 编辑分类表单数据对象
      editCateForm: {},
      // 编辑分类表单的验证规则对象
      editCateFormRules: {
          cat_name: [
              { required: true, message: '请输入分类名称', trigger: 'blur' }
          ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类失败！')
      }
      // 把数据列表赋值给cateList
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听 pagesize 改变
    handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
    },
    // 监听 pagenum 改变
    handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
    },
    // 点击按钮，弹出添加分类弹框
    showAddCateDialog() {
        // 获取父级分类的数据列表
        this.getParentCateList()
        this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
        const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
        if (res.meta.status !== 200) {
            return this.$message.error('获取父级分类数据失败！')
        }
        this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged() {
        // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
        // 反之，就说明没有选中如何父类分类
        if (this.selectedKeys.length > 0) {
          // 父级分类的ID
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          // 为当前分类的等级赋值
          this.addCateForm.cat_level = this.selectedKeys.length
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }
    },
    // 点击按钮，添加分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听弹框的关闭事件
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 点击编辑按钮，弹出编辑分类弹框
    async showEditCateDialog(id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类失败！')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    // 监听编辑弹框的关闭事件
    editCateDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    // 点击确定，提交修改表单
    editCateInfo() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
          cat_name: this.editCateForm.cat_name
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新失败！')
        }
        this.$message.success('更新成功！')
        this.getCateList()
        this.editCateDialogVisible = false
      })
    },
    // 删除角色
    async removeCate(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功！')
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}
</style>
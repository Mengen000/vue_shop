<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0" size="mini">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level===1" size="mini" type="success">二级</el-tag>
          <el-tag v-else size="mini" type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form
        ref="addCataFormRef"
        :model="addCataForm"
        :rules="addCataFormRules"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCataForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- options用来指定数据原 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
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
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: "分类名称",
          prop: "cat_name"
        },
        {
          label: "是否有效",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "isok"
        },
        {
          label: "排序",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "order"
        },
        {
          label: "操作",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "opt"
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCataForm: {
        // 要添加的分类的名称
        cat_name: "",
        // 父级分类的id
        cat_pid: 0,
        // 当前要添加的分类等级 默认1级
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCataFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps:{
        expandTrigger:'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children',
        checkStrictly:true
      },
      // 选中的父级分类Id数组
      selectedKeys:[]
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败");
      }
      // 把数据列表，赋值给catelist
      this.catelist = res.data.result;
      // 为总数据条数赋值
      this.total = res.data.total;
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    // 点击按钮 展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据
      this.getParentCateList();
      // 在展示出对话框
      this.addCateDialogVisible = true;
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: {
          type: 2
        }
      });
      if (res.meta.status !== 200) {
        return this.$$message.error("获取父级分类数据失败！");
      }
      this.parentCateList = res.data;
    },
    // 选择项发生变化触发这个函数
    parentCateChanged(){
      console.log(this.selectedKeys)
      // 如果selecttedKeys 数组中的length大于0，证明选中的父级分类
      // 反之，就说明没有选中任何父级分类
      if(this.selectedKeys.length>0){
        // 父级分类id
        this.addCataForm.cat_pid=this.selectedKeys[this.selectedKeys.length-1]
        // 为当前分类的等级赋值
        this.addCataForm.cat_level=this.selectedKeys.length;
      }else{
        // 父级分类id
        this.addCataForm.cat_pid=0;
        // 为当前分类的等级赋值
        this.addCataForm.cat_level=0;
      }
    },
    // 点击按钮，添加新的分类
    addCate(){
      this.$refs.addCataFormRef.validate(async valid=>{
        if(!valid) return;
        const{data:res} =await this.$http.post('categories',this.addCataForm);
        if(res.meta.status!==201){
          return this.$message.error("添加分类失败！");
        }
        this.$message.success("添加分类成功！");
        this.getCateList();
        this.addCateDialogVisible=false;
      })
    },
    // 监听对话框的关闭时间
    addCateDialogClosed(){
      this.$refs.addCataFormRef.resetFields();
      this.selectedKeys=[];
      this.addCataForm.cat_level=0;
      this.addCataForm.cat_pid=0;
    }
  }
};
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}
</style>
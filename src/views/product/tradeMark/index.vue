<template>
  <div>
    <!-- 按钮 -->
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 10px 0px"
      @click="showDialog"
    >
      添加
    </el-button>

    <!-- 
      表格组件 
        data:表格组件将来需要展示的数据 ---- 数组类型
        border:是给表格添加边框的
        column属性是列
        label:显示标题
        width:对应列的宽度
        align:标题的对齐方式
        prop:对应列的字段名
        注意1： elmentUI当中的table组件，展示的数据是以一列一列进行展示数据
    -->
    <el-table style="width: 100%" border :data="list">
      <el-table-column
        prop="prop"
        label="序号"
        type="index"
        width="80"
        align="center"
      >
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称" width="width">
      </el-table-column>
      <el-table-column prop="logoUrl" label="品牌LOGO" width="width">
        <template slot-scope="{ row, $index }">
          <!-- 结构出row,$index ，名字不能乱写，要和对象里的名一样。
                row表示数组里的每一条数据，$index表示序列
            -->
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 100px" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作" width="width">
        <template slot-scope="{ row, $index }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTradeMark(row)"
            >修改</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteTradeMark(row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <!-- 
      分页器 
        当前第几页，数据总条数，每一页展示条数，连续页码数              
        @current-change="" 
        @size-cahnge=""
        current-page 代表的是当前第几页
        total: 代表分页器一共需要展示数据条数
        page-size:代表的是每一页需要展示多少条数据
        page-sizes:代表可以设置每一页展示多少条数据
        layout: 可以实现分页器布局
        page-count :按钮的数量 如果是9 中间连续页码就是7


    -->
    <el-pagination
      style="margin-top: 20px; text-align: center"
      :total="total"
      :current-page="page"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
      :page-size="limit"
      :page-sizes="[3, 5, 10]"
      :page-count="7"
      layout="prev,pager,next,jumper,->,sizes,total"
    >
    </el-pagination>

    <!-- 对话框 
      :visible.sync:控制对话框显示与隐藏用的
      Form 组件提供了表单验证的功能，只需要通过 rules 属性传入约定的验证规则，
        并将 Form-Item 的 prop 属性设置为需校验的字段名即可。
    -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <!-- form表单 
          :model属性，这个属性的作用是，把表单的数据收集到那个对象上，将来表单验证，也需要这个属性
      
      -->
      <el-form style="width: 80%" :model="tmForm" :rules="rules" ref="ruleForm">
        <el-form-item label="品牌名称" label-width="100px" prop="tmName">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌LOGO" label-width="100px" prop="logoUrl">
          <!-- 
            这里收集数据不能使用v-model,因为不是表单元素
            action:设置图片上传的地址
            这里要记得补一下action路径，让他走代理，但注意商品走的是brand
            :on-success="handleAvatarSuccess" 可以监测到图片上传成功，当图片上传成功，会执行一次
            :before-upload="beforeAvatarUpload" 可以在上传图片之前，会执行一次
           -->
          <el-upload
            class="avatar-uploader"
            action="/brand/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "tradeMark",
  data() {
    // 自定义校验规则
    var validateTmName = (rule, value, callback) => {
      // 规则
      if (value.length < 2 || value.length > 10) {
        callback(new Error("品牌名称2-10位"));
      } else {
        callback();
      }
    };
    return {
      // 代表的分页器第几页
      page: 1,
      // 当前页数展示数据条数
      limit: 3,
      // 总共数据条数
      total: 0,
      // 列表展示的数据
      list: [],
      // 对话框显示与隐藏控制的属性
      dialogFormVisible: false,
      // 上传图片用到的属性
      // imageUrl: "",
      // 收集品牌的信息:对象身上的属性，不能瞎写，需要看文档中
      tmForm: {
        tmName: "",
        logoUrl: "",
      },
      // 表单验证规则
      rules: {
        // 品牌名称的验证规则
        // required：必须要校验的字段（前面的五角星）   message:提示信息  trigger:用户行为设置（事件的设置：blur(失焦)，change）
        //
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          // 自定义校验规则 : 取一个名称validateTmName,然后在data中书写对应名称的校验规则
          { validator: validateTmName, trigger: "blur" },
        ],
        // 品牌的logo验证规则
        logoUrl: [{ required: true, message: "请选择品牌的图片" }],
      },
    };
  },
  // 组件挂载完毕发请求
  mounted() {
    // console.log(this.$API);
    //获取列表数据方法
    this.getPageList();
  },
  methods: {
    // 获取品牌列表的数据
    async getPageList() {
      // 解构出参数
      const { limit, page } = this;
      // 获取品牌列表的接口
      // 当你向服务器发请求的时候，这个函数需要带参数，因此在data中初始化两个字段，代表给服务器传递参数
      let result = await this.$API.trademark.reqTradeMarkList(page, limit);
      // console.log(result);
      if (result.code == 200) {
        // 分别是展示数据总条数与列表展示的数据
        this.list = result.data.records;
        this.total = result.data.total;
      }
    },
    handleCurrentChange(pager) {
      // 修改参数
      this.page = pager;
      this.getPageList();
    },
    //当分页器某一页需要展示数据条数发生变化的时候会触发
    handleSizeChange(limit) {
      this.limit = limit;
      this.getPageList();
    },
    // 点击添加品牌的按钮
    showDialog() {
      // 点击显示
      this.dialogFormVisible = true;
      this.tmForm = {
        tmName: "",
        logoUrl: "",
      };
    },
    // 修改某一个品牌
    updateTradeMark(row) {
      // 显示对话框
      this.dialogFormVisible = true;
      // row ：当前用户选中这个品牌的信息
      //console.log(row);
      // 将已有的品牌信息赋值给tmForm展示
      // 将服务器返回品牌的信息直接赋值给了tmForm 进行展示
      // 也就是tmForm 存储即为服务器返回品牌信息
      this.tmForm = { ...row };
    },
    // 上传图片相关的回调
    // 图片上传成功
    handleAvatarSuccess(res, file) {
      // res:上传成功之后返回前端的数据
      // res.data是上传成功后图片在服务器的真实地址，不是本地地址
      // file:上传成功之后服务器返回前端数据
      this.tmForm.logoUrl = res.data;
    },
    // 图片上传之前
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    // 添加品牌或者修改品牌
    addOrUpdateTradeMark() {
      // 当全部验证字段通过后，再去书写业务逻辑
      this.$refs.ruleForm.validate(async (success) => {
        // 如果全部字段符合条件
        if (success) {
          // 隐藏对话框
          this.dialogFormVisible = false;
          // 发请求(添加品牌|修改品牌)
          let result = await this.$API.trademark.reqAddOrUpdateTradeMark(
            this.tmForm
          );
          if (result.code == 200) {
            // 弹出信息:添加品牌成功、修改品牌成功
            this.$message({
              message: this.tmForm.id ? "修改品牌成功" : "添加品牌成功",
              type: "success",
            });
            // 添加或者修改品牌成功之后再次获取品牌列表展示
            this.getPageList(this.tmForm.id ? this.page : 1);
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    // 删除品牌
    deleteTradeMark(row) {
      // 弹框
      this.$confirm(`是否删除${row.tmName}?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          // 当用户点击确定按钮的时候会触发
          // 向服务器发送请求
          let result = await this.$API.trademark.reqDeleteTradeMark(row.id);
          // 如果删除成功
          if (result.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            // 再次获取品牌列表数据
            // 我的写法是不对的，永远都是走的后面那个判断
            // this.list.length <1 ? this.getPageList(this.page-1) : this.getPageList(this.page)
            this.getPageList(this.list.length > 1 ? this.page : this.page-1)
          }
        })
        .catch(() => {
          // 当用户点击取消按钮的时候触发
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
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
<template>
  <div>
    <!-- inline:代表的是行内表单，代表一行可以放置多个表单元素 -->
    <el-form :inline="true" class="demo-form-inline" :model="cForm">
      <el-form-item label="一级分类">
        <el-select placeholder="请选择" v-model="cForm.category1Id" @change="handler1" :disabled="show">
            <!-- v-model表示的是收集到的选项框的东西，而选项框被选中的又是选项的value值 -->
          <el-option :label="c1.name" :value="c1.id" v-for="(c1,index) in list1" :key="c1.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select placeholder="请选择" v-model="cForm.category2Id" @change="handler2" :disabled="show">
          <el-option :label="c2.name" :value="c2.id" v-for="(c2,index) in list2" :key="c2.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select placeholder="请选择" v-model="cForm.category3Id" @change="handler3" :disabled="show">
          <el-option :label="c3.name" :value="c3.id" v-for="(c3,index) in list3" :key="c3.id"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: "CategorySelect",
  props:['show'],
  data(){
    return {
        // 一级分类的数据
        list1:[],
        // 二级分类的数据
        list2:[],
        // 三级分类的数据
        list3:[],
        // 收集相应的一级二级三级分类的id
        cForm:{
            category1Id:'',
            category2Id:'',
            category3Id:''
        }
    }
  },
  // 组件挂载完毕：向服务器发请求，获取响应的一级分类的数据
  mounted(){
    // 获取一级分类的数据的方法
    this.getCategory1List()
  },
  methods:{
    // 获取一级分类的请求
    async getCategory1List(){
        // 获取一级分类的请求，不需要携带参数
        let result = await this.$API.attr.reqCategory1List()
        if(result.code == 200){
            this.list1 = result.data
        }
    },
    // 一级分类的select事件回调（当一级分类的option发生变化的时候获取相应的二级分类数据）
    async handler1(){
      // 改变的时候二三级清除数据
        this.list2 = []
        this.list3 = []
        this.cForm.category2Id = ''
        this.cForm.category3Id = ''
        
        // 结构出一级分类的id
        const { category1Id } = this.cForm
        // 通过一级分类的id，获取二级分类的数据
        let result = await this.$API.attr.reqCategory2LIst(category1Id)
        if(result.code == 200){
          this.list2 = result.data
        }
        this.$emit('getCategoryId',category1Id,1)
    },
    // 二级分类的select事件回调（当二级分类的option发生变化的时候获取相应的三级分类数据）
    async handler2(){
      // 改变的时候三级分类清空
      this.list3 = []
      this.cForm.category3Id = ''
      // 解构出二级分类的id
      const {category2Id} = this.cForm
      // 通过二级分类的id，获取三级分类的数据
      let result = await this.$API.attr.reqCategory3LIst(category2Id)
      if(result.code == 200){
        this.list3 = result.data
      }
      this.$emit('getCategoryId',category2Id,2)
    },
    // // 三级分类的select事件回调
    handler3(){
      const {category3Id} = this.cForm
      this.$emit('getCategoryId',category3Id,3)
    }
  }
};
</script>

<style>
</style>
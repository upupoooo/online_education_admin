<template>
  <div class="app-container">
    <el-form label-width="120px">
      <el-form-item label="魔法师名称">
        <el-input v-model="teacher.name"/>
      </el-form-item>
      <el-form-item label="魔法师排序">
        <el-input-number v-model="teacher.sort" controls-position="right" sort="0"/>
      </el-form-item>
      <el-form-item label="魔法师头衔">
        <el-select v-model="teacher.level" clearable placeholder="请选择">
          <!--
            数据类型一定要和取出的json中的一致，否则没法回填
            因此，这里value使用动态绑定的值，保证其数据类型是number
          -->
          <el-option :value="1" label="高级魔法师"/>
          <el-option :value="2" label="首席魔法师"/>
        </el-select>
      </el-form-item>
      <el-form-item label="魔法师资历">
        <el-input v-model="teacher.career"/>
      </el-form-item>
      <el-form-item label="魔法师简介">
        <el-input v-model="teacher.intro" :rows="10" type="textarea"/>
      </el-form-item>

      <!-- 魔法师头像：TODO -->
      <!-- 魔法师头像 -->
      <el-form-item label="魔法师头像">

          <!-- 头衔缩略图 -->
          <pan-thumb :image="teacher.avatar"/>
          <!-- 文件上传按钮 -->
          <el-button type="primary" icon="el-icon-upload" @click="imagecropperShow=true">更换头像
          </el-button>

          <!--
      v-show：是否显示上传组件
      :key：类似于id，如果一个页面多个图片上传控件，可以做区分
      :url：后台上传的url地址
      @close：关闭上传组件
      @crop-upload-success：上传成功后的回调 -->
          <image-cropper
                        v-show="imagecropperShow"
                        :width="300"
                        :height="300"
                        :key="imagecropperKey"
                        :url="BASE_API+'/eduoss/file/upload'"
                        field="file"
                        @close="close"
                        @crop-upload-success="cropSuccess"/>

      </el-form-item>

      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>

import teacher from '@/api/edu/teacher'
import ImageCropper from '@/components/ImageCropper'
import PanThumb from '@/components/PanThumb'

export default {
  components: { ImageCropper, PanThumb },
  data() {
    return {
      teacher: {
        name: '',
        sort: 0,
        level: 1,
        career: '',
        intro: '',
        avatar: ''
      },
      //上传弹框组件是否显示
      imagecropperShow:false,
      imagecropperKey:0, //上传组件key值
      BASE_API:process.env.BASE_API, //获取dev.env.js里面的地址
      saveBtnDisabled: false // 保存按钮是否禁用,
    }
  },
  created(){ //页面渲染之前执行的
    this.init()
  },

  watch:{  //监听
    $route(to,from){  //路由变化方式，路由发生变化，方法就会执行
      this.init()
    }

  },

  methods: {
    //关闭上传弹框的方法
    close(){
      this.imagecropperShow=false
      //上传组件初始化
      this.imagecropperKey=this.imagecropperKey+1
    },
    //上传成功的方法
    cropSuccess(data){
      this.imagecropperShow=false
      //上传之后接口返回图片地址
      this.teacher.avatar=data.url
      this.imagecropperKey=this.imagecropperKey+1
    },
    init(){
      if(this.$route.params && this.$route.params.id){  //判断路径中有id就修改
        //从路径中获取到id值
        const id=this.$route.params.id
        //调用根据id查询的方法
        this.getInfo(id)
      }else{
        this.teacher={}
      }
    },
    //根据魔法师id查询的方法，数据回显
    getInfo(id){
      teacher.getTeacherInfo(id)
      .then(response=>{
        this.teacher=response.data.teacher
      })
    },
    saveOrUpdate() {
      //判断是修改还是添加
      //根据teacher是否有id
      if(!this.teacher.id){
        //添加
        this.saveData()
      }else{
        //修改
        this.updateTeacher()
      }
    },
    //修改魔法师的方法
    updateTeacher(){
      teacher.updateTeacher(this.teacher)
        .then(response=>{
          return this.$message({
            type:'success',
            message:'修改成功'
          })
        }).then(response=>{
          this.$router.push({path:'/teacher/list'})
        }).catch((response)=>{
          this.$message({
            type:'error',
            message:'修改失败'
          })
        })
    },
    // 保存
    saveData() {
      teacher.addTeacher(this.teacher)
      .then(response=>{
        return this.$message({
          type:'success',
          message:'添加成功'
        })
      }).then(response=>{
        this.$router.push({path:'/teacher/list'})
      }).catch((response)=>{
        this.$message({
          type:'error',
          message:'添加失败'
        })
      })
    }
  }
}
</script>
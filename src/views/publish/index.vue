<template>
  <div class="publish-container">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>{{$route.query.id ? '修改文章' : '发布文章'}}</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-form :model="article" :rules="formRules" label-width="60px" ref="publish-form">
        <el-form-item label="标题" prop="title">
          <el-input v-model="article.title"></el-input>
        </el-form-item>
        <el-form-item label="内容" prop="content">
          <!-- <el-input type="textarea" v-model="article.content"></el-input> -->
          <el-tiptap v-model="article.content" :extensions="extensions" height="400" placeholder="请输入文章内容"></el-tiptap>
        </el-form-item>
        <el-form-item label="封面">
          <el-radio-group v-model="article.cover.type">
            <el-radio :label="1">单图</el-radio>
            <el-radio :label="3">三图</el-radio>
            <el-radio :label="0">无图</el-radio>
            <el-radio :label="-1">自动</el-radio>
          </el-radio-group>
          <!-- 对于发布文章这个页面,需要把选择图片的地址放到 article.cover.images 数组中-->
          <template v-if="article.cover.type > 0">
            <!-- 子组件 -->
            <!--
              v-model="article.cover.images[index]"
              就相当与给子组件传递了名字 叫做 value 的数据
              :value = "article.cover.images[index]""
              默认监听 input 事件,当事件发送,它会让绑定数据 = 事件参数
              @input = "article.cover.images[index]=事件参数"
            -->
            <upload-cover v-for="(cover, index) in article.cover.type" :key="index"
              v-model="article.cover.images[index]">
            </upload-cover>
            <!-- <upload-cover v-for="(cover, index) in article.cover.type" :key="index"
              :cover-image="article.cover.images[index]" @update-cover="onUpdateCover(index,$event)">
            </upload-cover> -->
          </template>
        </el-form-item>
        <el-form-item label="频道" prop="channel_id">
          <el-select v-model="article.channel_id" placeholder="请选择频道">
            <el-option v-for="(item, id) in channels" :key="id" :label="item.name" :value="id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onPublish(false)">发表</el-button>
          <el-button @click="onPublish(true)">存入草稿</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
// 获取文的数据
import {
  getArticleChannels,
  addArticle,
  appointArticle,
  updateArticle
} from '@/api/article'

import {
  // 需要的 extensions
  Doc,
  Text,
  Paragraph,
  Heading,
  Bold,
  Image,
  Underline,
  Italic,
  Strike,
  ListItem,
  BulletList,
  OrderedList,
  TodoItem,
  TodoList, // (与 TodoItem 一起使用)
  HorizontalRule, // 分割线
  Fullscreen, // 全屏
  CodeBlock, // 代码块
  TextColor // 文字颜色
} from 'element-tiptap'

// 上传图片的请求
import {
  uploadImage
} from '@/api/images'

// 引入封面的组件
import UploadCover from './components/upload-cover'

export default {
  name: 'PublishIndex',

  components: {
    UploadCover
  },

  data () {
    return {
      // 从后台获取到的文章频道的数据
      channels: [],
      // 发布文章-表单绑定的数据
      article: {
        title: '', // 标题
        content: '', // 内容
        cover: { // 封面
          type: 0, // 封面的类型： -1-自动；0-无图； 1-1张； 3-3张
          images: [] // 封面图片的地址
        },
        channel_id: null
      },
      extensions: [
        new Doc(),
        new Text(),
        new Paragraph(),
        new Heading({ // 标题
          level: 5
        }),
        new Bold({ //
          bubble: true
        }), // 在气泡菜单中渲染菜单按钮
        new Underline({ // 下划线
          bubble: true,
          menubar: false
        }), // 在气泡菜单而不在菜单栏中渲染菜单按钮
        new Italic(), // 斜体
        new Strike(), // 删除线
        new Image({
          // 默认把图片生成 base64,字符串和内容存储在一起,如果需要自定义上传图片
          uploadRequest (file) {
            // 如果接口要求 Content-Type 是 multipart/form-data,则请求体必须使用 FormData 对象
            const fd = new FormData()
            fd.append('image', file)
            // 第一个 return 是返回 Promise 对象
            return uploadImage(fd).then(res => {
              // 返回图片的地址
              // 这个 return 是返回最后结果
              return res.data.data.url
            })
          }
        }), // 图片
        new ListItem(),
        new BulletList(), // 无序列表
        new OrderedList(), // 有序列表
        new TodoItem(),
        new TodoList(), // 任务框
        new HorizontalRule(), // 分割线
        new Fullscreen(), // 全屏
        new CodeBlock(), // 代码块
        new TextColor() // 文字颜色
      ],
      // form 表单的验证规则
      formRules: {
        // 标题的验证规则
        title: [{
          required: true,
          message: '请输入文章标题',
          trigger: 'blur'
        },
        {
          min: 5,
          max: 30,
          message: '长度在 5 到 30 个字符',
          trigger: 'blur'
        }
        ],
        // 内容的验证规则
        content: [{
          validator (rule, value, callback) {
            console.log('hhhhhh')
            if (value === '<p></p>') {
              callback(new Error('请输入文章内容'))
            } else {
              callback()
            }
          }
        }, {
          required: true,
          message: '请输入文章内容',
          trigger: 'blur'
        }],
        // 频道的验证规则
        channel_id: [{
          required: true,
          message: '请选择文章频道'
        }]
      }
    }
  },

  created () {
    // 页面初始化，获取文章频道数据
    this.loadChannels()

    // 由于发布文章 和修改 页面 基本一直，所以公用了一个组件
    // 所以这里要判断: 如果路径参数中有 id,则根据id 展示对应的文章内容
    // 根据 id 点击编辑按钮跳转页面,$route.query 可以看到 id
    if (this.$route.query.id) {
      // 要编辑文章,所以就要加载获取这个文章
      this.loadArticle()
    }
  },
  methods: {
    // 获取文章频道数据
    loadChannels () {
      getArticleChannels().then(res => {
        // console.log(res);
        this.channels = res.data.data.channels
      })
    },

    // 文章发表请求(和修改公用一个组件)
    // 判断是否 是修改文章,则执行修改操作;否则执行发布操作
    onPublish (draft = false) {
      // 文章提交之前,表单验证
      this.$refs['publish-form'].validate(valid => {
        if (!valid) {
          // 验证失败
          return
        }
        // 验证通过
        if (this.$route.query.id) {
          updateArticle(this.$route.query.id, this.article, draft).then(res => {
            console.log(res)
            this.$message({
              message: `${draft ? '存入草稿' : '发布'}成功`,
              type: 'success'
            })
            this.$router.push('/article')
          })
        } else {
          addArticle(this.article, draft).then(res => {
            console.log(res)
            this.$message({
              message: `${draft ? '存入草稿' : '发布'}成功`,
              type: 'success'
            })
            this.$router.push('/article')
          })
        }
      })
    },
    // 修改文章: 加载文章内容
    loadArticle () {
      // 获取指定文章
      appointArticle(this.$route.query.id).then(res => {
        // console.log(res);
        this.article = res.data.data
      })
    },
    // 接收子组件传递过来的值
    onUpdateCover (index, url) {
      console.log(index, url)
      // 如果需要传递三种图片,那么需要根据索引来传递,但是传递过来的参数就没了
      this.article.cover.images[index] = url
    }
  }
}

</script>
<style scoped>
</style>

<template>
	 <div class="panel">
    <panel-title :title="$route.meta.title">
    	<el-button @click.stop="on_refresh" size="small">
        <i class="fa fa-refresh"></i>
      </el-button>
      <router-link :to="{name: 'styleGoods', params: {gid: gid}}" tag="span">
        <el-button type="primary" icon="plus" size="small">添加数据</el-button>
        </router-link>
    </panel-title>
    <div class="panel-body">
    <h3 style="marginBottom:30px;">父商品名称:{{fatherGoods.name}}</h3>
    <el-row type="flex" class="row-bg" justify="space-around">
  <el-col :span="6" v-for="item in styleGoods" :key="item">
  <el-card :body-style="{ padding: '0px' }" class="wrap">
    <el-carousel height="300px">
      <el-carousel-item v-for="(item2,index) in item.img" :key="index">
        <img :src="item2.url" alt="">
      </el-carousel-item>
    </el-carousel>
  	<div style="padding: 14px;">
       <div class="name" style="marginBottom:10px;">{{item.name}}</div>
       <el-row v-for="(stock, size) in item.size" :key="stock" style="height:50px;">
       <el-col :span="5" style="lineHeight:40px;">{{size}}</el-col>
       <el-col :span="19" style="textAlign: right;">
        <el-input-number v-model="item.size[size]"  size="small" :min="1" :max="99999" label="描述文字"></el-input-number>
      </el-col>
       </el-row>
       <el-row type="flex" class="row-bg" justify="space-between">
  <el-col :span="12">
    <router-link :to="{name: 'styleGoods', params: {item: item, gid: gid, sid: item.sid}}" tag="span">
    <el-button type="primary" size="small" icon="el-icon-edit"></el-button>
    </router-link>
  </el-col>
  <el-col :span="12">
    <el-button type="danger" size="small" icon="el-icon-delete" @click="delete_data(item)"></el-button>
  </el-col>
  </el-row>
  </div>
  </el-card>
  	
  </el-col>
  </el-row>
    </div>
    </div>
</template>
<script>
import {panelTitle, bottomToolBar} from '@/components'
export default {
  data () {
    return {
      styleGoods: [],
      fatherGoods: [],
      gid: this.$route.params.gid,
      sid: this.$route.params.sid,
      title: '',
      load_data: false,
      dialog: false,
      // 批量选择数组
      batch_select: []
    }
  },
  watch: {
    styleGoods: {
      handler (val) {
        this.editStyleGoods(val)
      },
      deep: true
    }
  },
  mounted () {
    this.getStyleGoods()
  },
  methods: {
    getStyleGoods () {
      this.load_data = true
      this.$fetch.getPro.list({
        id: this.$route.params.gid
      }).then((res) => {
        this.load_data = false
        if (res.status) {
          this.fatherGoods = res.data[0]
          this.styleGoods = res.data[0].style
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    editStyleGoods (val) {
      console.log(val)
    },
    // 刷新
    on_refresh () {
      this.getStyleGoods()
    },
    // 单个删除
    delete_data (item) {
    // 后台哥哥硬要我加个数组
      let data = []
      data[0] = item
      this.$confirm('此操作将删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      .then(() => {
        this.load_data = true
        this.$fetch.getPro.sDel(data)
          .then((res) => {
            console.log(res)
            if (res.status) {
              this.getStyleGoods()
              this.$message.success(res.msg)
            } else {
              this.$message.error(res.msg)
            }
          })
          .catch(() => {
            this.load_data = false
          })
      })
      .catch(() => {
      })
    }
  },
  components: {
    panelTitle,
    bottomToolBar
  }
}
</script>

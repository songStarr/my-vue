<template>
  <div class="panel">
    <panel-title :title="$route.meta.title">
      <el-button @click.stop="on_refresh" size="small">
        <i class="fa fa-refresh"></i>
      </el-button>
      <router-link :to="{name: 'goodsAdd'}" tag="span">
        <el-button type="primary" icon="plus" size="small">添加数据</el-button>
      </router-link>
    </panel-title>
    <div class="panel-body">
    <div class="search">
      <search @search="submit_search"></search>
    </div>
      <el-table
        :data="goods"
        v-loading="load_data"
        element-loading-text="拼命加载中"
        border
        @selection-change="on_batch_select"
        style="width: 100%;">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          prop="id"
          label="id"
          width="80"
          sortable>
        </el-table-column>
        <el-table-column
          prop="name"
          label="商品名称"
          width="120"
          show-overflow-tooltip
          sortable>
        </el-table-column>
        <el-table-column
          prop="type.type_name"
          label="商品分类"
          width="120"
          sortable>
        </el-table-column>
        <el-table-column
          label="商品图片"
          width="100">
           <template scope="props">
           <img :src="props.row.img[0].url"/>
           <router-link :to="{name: 'viewStyleGoods', params: {gid: props.row.id}}" tag="span">
              <el-button type="primary" size="small" icon="view">查看</el-button>
            </router-link>
          </template>
        </el-table-column>
        <el-table-column
          prop="stock"
          label="库存"
          width="90"
          sortable>
        </el-table-column>
        <el-table-column
          label="创建时间"
          width="150"
          prop="date"
          sortable>
           <template scope="props">
        <el-icon name="time"></el-icon>
        <span style="margin-left: 10px">{{ props.row.date }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="是否上架"
          width="100">
           <template scope="props">
           <el-switch
            v-model="props.row.status"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="changeStatus(props.row)">
          </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="是否推广"
          width="100">
          <template scope="props">
         <el-switch
            v-model="props.row.spread"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="changeSpread(props.row)">
          </el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="子商品"
          width="170">
          <template scope="props">
            <router-link :to="{name: 'viewChildGoods', params: {gid: props.row.id}}" tag="span">
              <el-button type="primary" size="small" icon="view">查看</el-button>
            </router-link>
             <router-link :to="{name: 'childGoods', params: {gid: props.row.id}}" tag="span">
            <el-button type="danger" size="small" icon="edit">添加</el-button>
            </router-link>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="170">
          <template scope="props">
            <router-link :to="{name: 'goodsEdit', params: {id: props.row.id}}" tag="span">
              <el-button type="primary" size="small" icon="edit">修改</el-button>
            </router-link>
            <el-button type="danger" size="small" icon="delete" @click="delete_data(props.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <bottom-tool-bar>
        <el-button
          type="danger"
          icon="delete"
          size="small"
          :disabled="batch_select.length === 0"
          @click="on_batch_del"
          slot="handler">
          <span>批量删除</span>
        </el-button>
        <div slot="page">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-sizes="[5, 10, 15, 20]"
            :page-size="length"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
          </el-pagination>
        </div>
      </bottom-tool-bar>
    </div>
  </div>
</template>
<script type="text/javascript">
  import {panelTitle, bottomToolBar, search} from '@/components'

  export default{
    data () {
      return {
        goods: null,
        // 当前页码
        currentPage: 1,
        // 数据总条目
        total: 0,
        // 每页显示多少条数据
        length: 5,
        // 分类 0为全部
        type: 0,
        // 搜索内容
        str: '',
        // 请求时的loading效果
        load_data: true,
        // 批量选择数组
        batch_select: []
      }
    },
    components: {
      panelTitle,
      bottomToolBar,
      search
    },
    created () {
      this.get_table_data()
      this.$store.dispatch('GET_TYPE')
    },
    watch: {
      goods: {
        handler (newGoods) {
          // console.log(newGoods)
        },
        deep: true
      }
    },
    methods: {
      // 刷新
      on_refresh () {
        this.get_table_data()
      },
      // 获取数据
      get_table_data () {
        this.load_data = true
        console.log(this.$fetch)
        this.$fetch.getPro.list({
          page: this.currentPage,
          length: this.length,
          type: this.type,
          str: this.str
        })
          .then((res) => {
            console.log(res)
            this.goods = res.data
            this.currentPage = res.page
            this.total = res.total
            this.load_data = false
          })
          .catch(() => {
            this.load_data = false
          })
      },
      // 修改status
      changeStatus (item) {
        console.log(item)
        // item.status = item.status ? 0 : 1
        this.editGoods(item)
      },
      // 修改spread
      changeSpread (item) {
        // item.spread = item.spread ? 0 : 1
        this.editGoods(item)
      },
      // 修改商品
      editGoods (item) {
        this.$fetch.getPro.edit(item)
          .then((res) => {
            if (res.status) {
              this.$message({
                message: res.msg,
                type: 'success'
              })
            } else {
              this.$message({
                type: 'error',
                message: res.msg
              })
            }
            this.on_submit_loading = false
          })
          .catch(() => {
            this.on_submit_loading = false
          })
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
            this.$fetch.getPro.del(data)
              .then((res) => {
                console.log(res)
                if (res.status) {
                  this.get_table_data()
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
      },
      // 搜索
      submit_search (value) {
        this.type = value.type ? value.type : 0
        this.str = value.value
        this.get_table_data()
      },
      formatter (row, column) {
        return row.id
      },
      filterTag (value, row) {
        return row.hot === value
      },
      // 页码选择
      handleCurrentChange (val) {
        this.currentPage = val
        this.get_table_data()
      },
      // 页码长度
      handleSizeChange (val) {
        this.length = val
        this.get_table_data()
      },
      // 批量选择
      on_batch_select (val) {
        this.batch_select = val
      },
      // 批量删除
      on_batch_del () {
        this.$confirm('此操作将批量删除选择数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            this.load_data = true
            this.$fetch.getPro.del(this.batch_select)
              .then((res) => {
                this.load_data = false
                if (res.status) {
                  this.$message.success(res.msg)
                  this.get_table_data()
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
    }
  }
</script>

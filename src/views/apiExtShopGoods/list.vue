<template>
  <div class="app-container">

    <div class="filter-container">
      <el-input clearable @keyup.enter.native="fetchData" style="width: 200px;" class="filter-item" placeholder="商品编号"
                v-model="searchData.id">
      </el-input>
      <el-input clearable @keyup.enter.native="fetchData" style="width: 200px;" class="filter-item" placeholder="条码编号"
                v-model="searchData.barCode">
      </el-input>
      <el-select clearable style="width: 200px" class="filter-item" v-model="searchData.shopId" placeholder="所属店铺">
        <el-option
          v-for="item in shopData"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-select clearable style="width: 200px" class="filter-item" v-model="searchData.categoryIdSel"
                 placeholder="商品分类">
        <el-option
          v-for="item in categoryData"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-input clearable @keyup.enter.native="fetchData" style="width: 200px;" class="filter-item" placeholder="商品名称"
                v-model="searchData.nameLike">
      </el-input>
      <el-input clearable @keyup.enter.native="fetchData" style="width: 200px;" class="filter-item" placeholder="商品特色"
                v-model="searchData.characteristicLike">
      </el-input>
      <el-select clearable style="width: 200px" class="filter-item" v-model="searchData.status" placeholder="商品状态">
        <el-option label="上架" value="0"></el-option>
        <el-option label="下架" value="1"></el-option>
      </el-select>
      <el-select clearable style="width: 200px" class="filter-item" v-model="searchData.recommendStatus"
                 placeholder="是否推荐">
        <el-option label="不推荐" value="0"></el-option>
        <el-option label="推荐" value="1"></el-option>
      </el-select>
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="fetchData">搜索</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" @click="handleCreate" type="success"
                 icon="el-icon-edit">添加
      </el-button>
    </div>

    <el-table :data="list" v-loading.body="listLoading" element-loading-text="Loading" border fit highlight-current-row
              empty-text="暂无数据" @selection-change="handleSelectionChange">
      <el-table-column prop="id" width="90px" label="商品编号"></el-table-column>
      <el-table-column prop="shopName" label="店铺"></el-table-column>
      <el-table-column prop="categoryName" label="分类"></el-table-column>
      <el-table-column prop="name" label="标题"></el-table-column>
      <el-table-column label="图片" min-width="100px">
        <template slot-scope="scope" v-if="scope.row.pic">
          <img :src="scope.row.pic" width="100" height="100">
        </template>
      </el-table-column>
      <el-table-column prop="statusStr" label="状态"></el-table-column>
      <el-table-column prop="recommendStatusStr" label="推荐状态"></el-table-column>
      <el-table-column prop="stores" label="总库存"></el-table-column>
      <el-table-column prop="originalPrice" label="原价"></el-table-column>
      <el-table-column prop="minPrice" label="最低价"></el-table-column>
      <el-table-column prop="linkPhone" label="商品数据">
        <template slot-scope="scope">
          <p>浏览量 : {{scope.row.views}}</p>
          <p>收藏量 : {{scope.row.numberFav}}</p>
          <p>订单数 : {{scope.row.numberOrders}}</p>
          <p>好评数 : {{scope.row.numberGoodReputation}}</p>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="handleUpdate(scope.row.id)">编辑</el-button>
          <el-button type="text" @click="delData(scope.row.id)" style="color:red">删除</el-button>
        </template>
      </el-table-column>
    </el-table>


  </div>
</template>

<script>
  import {delData, fetchDataList, getShopData, getShopGoodsCategoryData} from '@/api/apiExtShopGoods'
  import {Message} from 'element-ui'
  import {mapGetters} from 'vuex'

  export default {
    computed: {
      ...mapGetters([
        'centerUserBase'
      ])
    },
    data() {
      return {
        categoryData: [],
        shopData: [{label: '全部', value: ''},
          {label: '未归属任何店铺', value: 0}],
        rules: {
          goodsId: [
            {required: true, message: '不能为空'},
            {type: 'integer', message: '必须为数字'}
          ],
          number: [
            {required: true, message: '不能为空'},
            {type: 'integer', message: '必须为数字'}
          ],
          originalPrice: [
            {required: true, message: '不能为空'},
            {type: 'number', message: '必须为数字'}
          ],
          minPrice: [
            {required: true, message: '不能为空'},
            {type: 'number', message: '必须为数字'}
          ],
          helpPriceMin: [
            {required: true, message: '不能为空'},
            {type: 'number', message: '必须为数字'}
          ],
          helpPriceMax: [
            {required: true, message: '不能为空'},
            {type: 'number', message: '必须为数字'}
          ],
          status: [
            {required: true, message: '不能为空'}
          ],
          dateAddStr: [
            {required: true, message: '不能为空'}
          ],
          dateEndStr: [
            {required: true, message: '不能为空'}
          ]
        },
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end])
            }
          }]
        },
        dateAdd: '',
        dateUpdate: '',

        searchData: {
          id: undefined,
          barCode: undefined,
          status: undefined,
          nameLike: undefined,
          shopId: undefined,
          categoryIdSel: undefined,
          characteristicLike: undefined,
          recommendStatus: undefined,
          categoryValue: ''
        },

        pushData: {
          dialogTitle: undefined,
          dialogFormVisible: false,

          id: undefined,
          type: undefined,
          status: undefined,
          nameLike: undefined,
          addressLike: undefined,
          dateAddStr: undefined,
          dateEndStr: undefined
        },

        multipleSelection: [],
        list: null,
        listLoading: true,
        statisticsData: {}
      }
    },
    created() {
      this.pushDataTmp = Object.assign({}, this.pushData);

      this.fetchData();

      this.getShopData();
      this.getShopGoodsCategoryData();
    },
    mounted() {

    },
    methods: {
      getShopData() {
        getShopData().then(response => {
          if (response.code === 0) {
            response.data.result.forEach((a) => {
              let v = {};
              v.value = a.id;
              v.label = a.name;
              this.shopData.push(v);
            })
          }
        })
      },
      getShopGoodsCategoryData() {
        getShopGoodsCategoryData().then(response => {
          const data = [];
          if (response.code === 0) {
            response.data.forEach((a) => {
              let v = {}, s = '';
              v.value = a.id;
              if (a.level > 1) {
                for (let i = 2; i < a.level; i++) {
                  s += '　　　　'
                }
                s += '╚═══'
              }
              v.label = s + a.name;
              data.push(v)
            })
          }
          this.categoryData = data
        })
      },
      handleDateAdd(val) {
        this.searchData.dateAddBegin = val ? val[0] : undefined;
        this.searchData.dateAddEnd = val ? val[1] : undefined
      },
      handleDateUpdate(val) {
        this.searchData.dateUpdateBegin = val ? val[0] : undefined;
        this.searchData.dateUpdateEnd = val ? val[1] : undefined
      },
      handleSelectionChange(val) {
        this.multipleSelection = val
      },
      fetchData() {
        this.list = null;
        this.listLoading = true;
        fetchDataList(this.searchData).then(response => {
          if (response.code === 0) {
            this.list = response.data.result;
            this.totalRow = response.data.totalRow
          }
          this.listLoading = false
        })
      },
      handleCreate() {
        this.$router.push({path: '/user/apiExtShopGoods/add'})
      },
      handleUpdate(id) {
        this.$router.push({path: '/user/apiExtShopGoods/add', query: {id: id}})
      },
      delData(id) {
        this.$confirm('删除无法恢复, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          delData(id).then(() => {
            Message({
              message: '删除成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.fetchData()
              }
            })
          })
        }).catch(() => {
        })
      }
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss">
  .filter-container {
    padding-bottom: 10px;
    .filter-item {
      display: inline-block;
      vertical-align: middle;
      margin-bottom: 10px;
    }
  }

  .el-range-editor.el-input__inner {
    padding: 7px 11px;
  }


</style>

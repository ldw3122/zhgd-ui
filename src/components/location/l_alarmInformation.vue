<template>
  <div class="content-warp">
    <!-- 搜索栏 -->
    <div class="search-warp">
      <div class="search">
        <div class="search-item">
          <span>人员姓名：</span>
          <el-input v-model="searchUserName" placeholder="请输入姓名" clearable></el-input>
        </div>
        <div class="search-item date-time">
          <span>报警时间：</span>
          <el-date-picker
            v-model="searchTime"
            type="daterange"
            align="center"
            unlink-panels
            value-format="yyyy-MM-dd"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            placeholder="请输入时间">
          </el-date-picker>
        </div>
        <div class="search-item">
          <span>报警类型：</span>
          <el-select v-model="searchWarningType" clearable>
            <el-option
              v-for="item in warningTypeList"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </div>
      </div>
      <div class="search-button">
        <div class="confirm" @click="search(),pageNum=1">搜索</div>
      </div>
    </div>
    <!-- 表格主体 -->
    <div class="table-warp">
      <el-table
        :data="tableData"
        style="width:100%"
        :header-cell-style="headerStyle"
        :row-style="bodyStyle"
        :cell-style="cellStyle"
      >
        <el-table-column type="selection" width="40"></el-table-column>
        <el-table-column prop="userName" label="人员姓名"></el-table-column>
        <el-table-column prop="warningTime" label="报警时间"></el-table-column>
        <el-table-column label="报警原因">
          <template slot-scope="scope">
            {{getWarningName(scope.row.warningType)}}
          </template>
        </el-table-column>
        <el-table-column prop="areaName" label="所属工区"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <p class="examine" @click="examine(scope.row.userName)">查看</p>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page="pageNum"
        :page-size="pageSize"
        layout="total, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>
  </div>
</template>

<script>
import mixin from "@/utils/mixin.js"
export default {
  mixins: [mixin],
  data() {
    return {
      searchUserName: '', // 搜索的人名
      searchTime: '', // 搜索时间
      searchWarningType: '', // 搜索报警
      warningTypeList: [
        {
          label: 'SOS报警',
          value: 0
        },
        {
          label: '摔跌报警',
          value: 1
        },
        {
          label: '久站不动报警',
          value: 2
        },
        {
          label: '低电量报警',
          value: 3
        }
      ], // 报警搜索列表
      tableData: [], //列表数据
      total: 0, // 总条目数
      pageSize: 15, // 每页显示条数
      pageNum: 1, // 当前页数
      firstSearch: false, // 是否使用搜索搜索
    }
  },
  mounted() {
    this.getDay()
    this.getList()
  },
  methods: {
    headerStyle() {
      return {
        backgroundColor: '#142a6a',
        color: '#2f6dee',
        fontSize: '.16rem',
        borderBottomColor: 'rgba(0,0,0,0)'
      }
    },
    bodyStyle() {
      return {
        backgroundColor: 'rgba(0,0,0,0)',
        color: '#fff'
      }
    },
    cellStyle() {
      return {
        borderBottomColor: '#142a6a!important'
      }
    },

    // 返回报警名称
    getWarningName(id) {
      for (let i = 0; i < this.warningTypeList.length; i++) {
        if (this.warningTypeList[i].value == id) {
          return this.warningTypeList[i].label
        }
      }
    },

    // 翻页
    handleCurrentChange(val) {
      this.pageNum = val
      this.search()
    },

    //查看详情
    examine(id) {
      this.$router.push({
        path: '/location/l_search',
        query: {orderId: id}
        })
    },

    //获取列表数据
    getList() {
      if (this.$route.query.userName && (this.$route.query.warningType || this.$route.query.warningType === 0)) {
        this.searchUserName = this.$route.query.userName
        this.searchWarningType = this.$route.query.warningType
        this.search()
      } else {
        this.$axios
          .post(`/api/pcEquipmentWarning/getWarningList?projectId=${this.projectId}&pageSize=${this.pageSize}&pageNum=${this.pageNum}`)
          .then(res => {
            if (res.data.code == 0) {
              this.tableData = res.data.data
              this.total = res.data.total
            }
          })
      }
    },

    //搜索
    search() {
      let start = ''
      let end = ''
      if (this.searchTime) {
        start = this.searchTime[0] + ' 00:00:00'
        end = this.searchTime[1] + ' 23:59:59'
      }
      this.$axios
        .post(`/api/pcEquipmentWarning/getWarningList?projectId=${this.projectId}&username=${this.searchUserName}&warningType=${this.searchWarningType}&pageSize=${this.pageSize}&pageNum=${this.pageNum}&startTime=${start}&endTime=${end}`)
        .then(res => {
          if (res.data.code == 0) {
            this.tableData = res.data.data
            this.total = res.data.total
          }
        })
    }
  }
}
</script>

<style lang="less" scoped>
.content-warp {
    width: 15rem;
    height: 9.39rem;
    background-image: url("../../../static/images/location-bigBorder.png");
    background-position: center center;
    background-repeat: no-repeat;
    background-size: contain;
    .search-warp {
      width: 100%;
      height: .8rem;
      padding: .2rem;
      margin-top: .1rem;
      box-sizing: border-box;
      display: flex;
      .search {
        flex: 1;
        justify-content: space-between;
        flex-wrap: nowrap;
        display: flex;
        .search-item {
          color: #f4f4f4;
          font-size: .16rem;
          display: inline-block;
          .el-input {
            width: auto;
          }
          /deep/input {
            width: 2.6rem;
            border: 1px solid #3375fe;
            border-radius: 5px;
            background: #1d1f3b;
            color: #f4f4f4;
          }
        }
        .date-time {
          .el-input__inner {
            background: #1d1f3b;
            border: 1px solid #3375fe;
          }
          /deep/input {
            border: none;
          }
          /deep/span {
            color: #f4f4f4;
          }
          /deep/input {
            width: 1.45rem!important;
          }
        }
      }
      .search-button {
        position: relative;
        width: 2.6rem;
        .confirm {
          position: absolute;
          right: 0;
          width: 1.6rem;
          height: .4rem;
          background: #3273fb;
          border-radius: 5px;
          color: #fff;
          line-height: .4rem;
          text-align: center;
          font-size: .18rem;
          cursor: pointer;
        }
      }
      /deep/.el-input__inner {
        height: .4rem;
        line-height: .4rem;
      }
      /deep/.el-input__icon {
        line-height: .4rem;
      }
      /deep/.el-date-editor--daterange.el-input, .el-date-editor--daterange.el-input__inner, .el-date-editor--timerange.el-input, .el-date-editor--timerange.el-input__inner {
        width: 3.5rem;
      }
      /deep/.el-date-editor .el-range-separator {
        padding: 0;
        line-height: .32rem;
      }
      /deep/.el-date-editor .el-range__icon, /deep/.el-date-editor .el-range__close-icon {
        line-height: .32rem;
      }
      /deep/.el-input, /deep/.el-date-editor .el-range-input, /deep/.el-date-editor .el-range-separator {
        font-size: .14rem;
      }
    }
    .table-warp {
      display: block;
      width: 100%;
      height: calc(100% - .8rem);
      position: relative;
      /deep/.el-table-column--selection .cell {
        padding-right: 0!important;
      }
      /deep/.el-table, .el-table__expanded-cell {
        background-color: rgba(0,0,0,0)!important;
      }
      /deep/.el-table__row:hover>td {
        background: #142a6a !important;
      }
      .el-table::before {
        height: 0px;
      }
      .examine {
        cursor: pointer;
      }
      .el-pagination {
        position: absolute;
        bottom: .3rem;
        right: .1rem;
        /deep/button {
          background: rgba(0,0,0,0) !important;
        }
        /deep/.el-pager li {
          background: rgba(0,0,0,0) !important;
          color: #878788;
        }
        /deep/.el-pager li.active {
          color: #409EFF;
        }
        /deep/.el-pagination__editor.el-input .el-input__inner {
          background: rgba(0,0,0,0);
          border: none;
        }
      }
      /deep/.el-table td, /deep/.el-table th {
        padding: .12rem 0;
      }
      /deep/.el-table .cell {
        font-size: .14rem;
        line-height: .23rem;
      }
    }
}
</style>
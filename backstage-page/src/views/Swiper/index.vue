<template>
  <div class="swiper-wrap">
    <el-card v-loading="loading">
      <div slot="header">轮播图列表</div>
      <el-table :data="formData" border stripe highlight-current-row height="400" v-loadmore='loadmore'>
        <el-table-column prop="avatar" width='100' label="轮播图头图" align='center'>
          <template slot-scope="scope">
            <img class="table-item-img" :src="scope.row.img" alt="">
          </template>
        </el-table-column>
        <el-table-column prop="title" width='120' label="轮播图标题" align='center' show-overflow-tooltip></el-table-column>
        <el-table-column prop="newsId" width='100' label="新闻头图" align='center'>
          <template slot-scope="scope">
            <img class="table-item-img" :src="scope.row.newsId.img" alt="">
          </template>
        </el-table-column>
        <el-table-column prop="newsId.title" width='200' label="新闻标题" align='center' show-overflow-tooltip></el-table-column>
        <el-table-column sortable prop="create_time" width='180' label="创建时间" align='center'></el-table-column>
        <el-table-column sortable prop="sort" width='80' label="权重" align='center'></el-table-column>
        <el-table-column prop="status" width='100' label="显示与否" align='center' :filters="[{ text: '显示', value:1}, { text: '不显示', value:0 }]" :filter-method="filterTag" filter-placement="bottom-end">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status === 1 ? 'primary' : 'success'" disable-transitions>{{scope.row.status == 1 ?'显示':'不显示'}}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width='180' align='center'>
          <template slot-scope="scope">
            <el-button type="primary" size='mini' @click="editclick(scope.row._id)">编辑</el-button>
            <el-button type="danger" size='mini' @click="delclick(scope.row._id)">删除</el-button>
          </template>
        </el-table-column>
        <template slot="append">
          <div v-if="loadSign" class="loading" v-loading="loadSign"></div>
          <div v-else class="nopage-wrap">
            <div class="nopage">没有数据了</div>
            <div class="no-line"></div>
          </div>
        </template>
      </el-table>
      <!-- <el-pagination
              class="flr pagination"
              background
              small
              layout="prev, pager, next"
              @current-change='handleCurrentChange'
              :page-size="4"
              :total="total">
            </el-pagination> -->
    </el-card>
  </div>
</template>

<script>
// import  { Scrollbar } from 'element-ui';
import getDate from "../../unitls/date.js";
export default {
  name: "swiper",
  //   components:{ scrollbar },
  data() {
    return {
      formData: [],
      page: 1,
      page_size: 4,
      total: null,
      loading: true,
      loadSign: true,
    };
  },
  methods: {
    loadmore() {
      // if (this.loadSign) {
      //   this.loadSign = false;
      //   this.page++;
      //   if (this.page > 10) {
      //     return;
      //   }
      //   setTimeout(() => {
      //     this.loadSign = true;
      //   }, 1000);
      // }
      if (this.loadSign) {
        this.page++;
        this.getData();
      } else {
        this.$message({
          message: "没有数据了！",
          type: "info",
          duration: 1000,
          center: true
        });
      }
    },
    delclick(id) {
      this.$confirm("此操作将永久删除轮播图！！！, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$axios.del(`/swiper/delete?id=${id}`).then(res => {
            // console.log(res);
            if (res.code == 200) {
              this.$message({
                type: "success",
                message: res.msg,
                center: true
              });
              setTimeout(() => {
                this.formData = []
                this.getData();
              }, 1000);
            } else {
              this.$message({
                type: "error",
                message: "error:" + res.msg,
                center: true
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    editclick(id) {
      this.$router.push({ name: "editSwiper", params: { id } });
    },
    // handleCurrentChange(val) {
    //   this.page = val;
    //   this.getData();
    // },
    filterTag(value, row) {
      return row.status === value;
    },
    getData() {
      this.loading = true;
      this.$axios.get(`/swiper/getSwiper?page=${this.page}&page_size=${this.page_size}`)
        .then(res => {
          // console.log(res);
          if (res.code == 200) {
            res.data.forEach(item => {
              item.create_time = getDate(item.create_time);
              return item;
            });
            // this.formData = res.data;
            this.formData.push.apply(this.formData, res.data);
            this.total = res.count;
            this.loading = false;
            if(res.count < this.page_size){
              this.loadSign  = false
            }else if (res.data.length == 0) {
              this.loadSign = false;
            } 

          } else {
            this.$message({
              message: res.msg,
              type: "error",
              duration: 1000,
              center: true
            });
            this.loading = false;
          }
        })
        .catch(err => {
          this.$message({
            message: "网络链接错误！",
            type: "error",
            duration: 1000,
            center: true
          });
          this.loading = false;
        });
    }
  },
  created() {
    this.getData();
  }
};
</script>

<style scoped lang='scss'>
.pagination {
  margin: 10px 80px;
}
/deep/ .el-table td {
  padding: 9px 0;
}
/deep/ .el-table {
  overflow-x: hidden;
}
.loading {
  height: 50px;
}
// 使用position: relative;设置位置后，只设置一个组件的z-index，没有效果，需要将需显示的组件都设置z-index属性
.nopage-wrap {
  position: relative;
  text-align: center;
  height: 50px;
  .nopage {
    z-index: 10;
    position: absolute;
    top: 50%;
    left: 50%;
    height: 1px;
    transform: translate(-50%, -50%);
    line-height: 50px;
    height: 50px;
    width: 120px;
    background-color: #fff;
    font-size: 16px;
    font-weight: 600;
  }
  .no-line {
    z-index: 1;
    width: 80%;
    position: absolute;
    top: 50%;
    left: 50%;
    height: 1px;
    transform: translate(-50%, -50%);
    background-color: #ccc;
  }
}
</style>
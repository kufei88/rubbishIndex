<template>
  <div>
    <!-- 自定义查询模块 -->
    <Input
      search
      enter-button="查询"
      placeholder="请输入查询关键字，如姓名、联系方式"
      style="width:300px;margin-bottom:10px;float:left;"
      @on-search="handleSearch"
      v-model="searchData"
    />
    <!-- 删除用户按钮 -->
    <Button type="error" @click="handleDelete" style="float:right;margin-left:10px">删除</Button>

    <div style="clear:both"></div>

    <!-- 用户表格显示 -->
    <Table
      border
      highlight-row
      :columns="dataColumns"
      :data="pageData"
      ref="currentRowTable"
      height="522"
      @on-current-change="currentChange"
    ></Table>

    <!-- 分页功能 -->
    <span>记录总共 {{this.dataCount}} 条</span>
    <Page
      :total="dataCount"
      @on-change="changePage"
      show-sizer
      :current="pageCurrent"
      @on-page-size-change="changePageNumber"
      :page-size-opts="[10,20,50,100]"
      align="center"
    />
  </div>
</template>

<script>
import axios from "@/libs/api.request";
export default {
  data() {
    return {
      isSelectRow: false, // 是否选中状态
      deleteData: {}, // 需要删除数据
      searchData: "", // 查询搜索字段
      // 表格显示的列名数据
      dataColumns: [
        {
          type: "index",
          width: 60,
          align: "center",
          indexMethod(row) {
            return row._index + 1 + (row.pageCurrent - 1) * row.pageSize;
          }
        },
        {
          title: "姓名",
          key: "userName",
          align: "center"
        },
        {
          title: "年龄",
          key: "age",
          align: "center"
        },

        {
          title: "性别",
          key: "sex",
          width: 80,
          align: "center"
        },
        {
          title: "联系电话",
          key: "telephone",
          align: "center"
        },
        {
          title: "工作单位",
          key: "workspace",
          align: "center"
        },
        {
          title: "地区",
          key: "region",
          align: "center"
        },
        {
          title: "添加时间",
          key: "insertTime",
          align: "center"
        }
      ],

      pageCurrent: 1, // 当前页数
      pageStart: 0, // 记录开始位置
      dataCount: 0, // 后台读取的总记录长度
      pageSize: 10, // 每页显示多少条
      pageData: [] // table绑定的数据
    };
  },
  methods: {
    // 查询点击事件
    handleSearch(value) {
      this.searchData = value;
      this.pageCurrent = 1;
      this.getRequestData(this.pageCurrent);
    },

    // 选中行改变点击事件
    currentChange(currentRow, index) {
      this.deleteData = currentRow;
      this.isSelectRow = true;
    },

    // 改变每页条数
    changePageNumber(index) {
      this.pageSize = index;
      if (this.pageCurrent === 1) {
        this.changePage(this.pageCurrent);
      }
    },

    // 分页
    changePage(index) {
      // 获得当前页数，以及发送数据请求
      this.pageCurrent = index;
      this.getRequestData(index);
    },

    // 从后台查询数据,index为当前页码
    getRequestData(index) {
      let _this = this;
      this.pageStart = (index - 1) * this.pageSize;
      axios
        .request({
          url: "user/getUserList",
          method: "get",
          params: {
            search: this.searchData,
            start: this.pageStart,
            count: this.pageSize
          }
        })
        .then(function(response) {
          _this.pageData = response.data.userList;
          _this.dataCount = response.data.dataCount;
          _this.addPageCurrentAndPageSize(_this.pageData);
        });
    },

    // 添加当前页和页码数据
    addPageCurrentAndPageSize(updatePageData) {
      for (var key in updatePageData) {
        updatePageData[key].pageCurrent = this.pageCurrent;
        updatePageData[key].pageSize = this.pageSize;
      }
    },

    // 删除某用户信息
    handleDelete() {
      if (this.isSelectRow == true) {
        this.$Modal.confirm({
          title: "删除提示",
          content: "<p>是否确认将该用户删除？</p>",
          onOk: () => {
            let _this = this;
            let _data = this.deleteData;
            axios
              .request({
                url: "user/deleteUser",
                method: "post",
                headers: {
                  "Content-Type": "application/json;charset=UTF-8"
                },
                data: _data
              })
              .then(function(response) {
                if (response.data == 1) {
                  _this.$Message.success("删除成功");
                  // 判断是否pageData的数据长度<=1,然后判断是否第1页,是则页数减1;
                  if (_this.pageData.length <= 1) {
                    if (_this.pageCurrent != 1) {
                      _this.pageCurrent = _this.pageCurrent - 1;
                    }
                  }
                  _this.getRequestData(_this.pageCurrent);
                } else {
                  _this.$Message.error("删除失败");
                }
                _this.isSelectRow = false;
              });
          },
          onCancel: () => {}
        });
      } else {
        this.$Message.error("请先选择记录");
      }
    }
  },
  mounted() {
    this.getRequestData(this.pageCurrent);
  }
};
</script>
<style lang="scss" scoped>
</style>
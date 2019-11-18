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
    <!-- 删除成绩记录按钮 -->
    <Button type="error" @click="handleDelete" style="float:right;margin-left:10px">删除</Button>

    <!-- 显示历史记录按钮 -->
    <Button type="warning" @click="handleHistory" style="float:right;margin-left:10px">历史记录</Button>

    <div style="clear:both"></div>

    <!-- 成绩表格显示 -->
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

    <!-- 历史记录弹窗 -->
    <Modal
      :closable="false"
      v-model="isShowHistory"
      :mask-closable="false"
      :scrollable="true"
      width="70"
      title="历史成绩记录"
    >
      <p style="margin-top:10px;margin-bottom:10px;display:flex">
        <span style="font-size:14px;flex:1">
          <strong>姓名:</strong>
          {{selectRowData.userName}}
        </span>

        <span style="font-size:14px;margin-left:35px;flex:2">
          <strong>联系方式:</strong>
          {{selectRowData.telephone}}
        </span>
      </p>

      <Table
        border
        highlight-row
        :columns="showColumns"
        :data="historyData"
        ref="showTable"
        height="400"
      ></Table>

      <!-- 确定取消框 -->

      <div slot="footer">
        <Button type="primary" size="large" @click="handleSubmit()">确定</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
import axios from "@/libs/api.request";
export default {
  data() {
    return {
      // 历史成绩表格列名
      showColumns: [
        {
          type: "index",
          width: 60,
          align: "center",
          indexMethod(row) {
            return row._index + 1;
          }
        },
        {
          title: "厨余垃圾分数",
          key: "kitchenWasteFraction",
          align: "center"
        },
        {
          title: "有害垃圾分数",
          key: "harmfulWasteFraction",
          align: "center"
        },
        {
          title: "可回收垃圾分数",
          key: "recyclableWasteFraction",
          align: "center"
        },
        {
          title: "其他垃圾分数",
          key: "otherWasteFraction",
          align: "center"
        },
        {
          title: "总评分",
          key: "scoreFraction",
          align: "center"
        },
        {
          title: "添加时间",
          key: "insertTime",
          align: "center"
        }
      ],
      selectRowData: {}, // 被选中的数据
      isShowHistory: false, // 是否显示历史数据
      historyTelephone: "", // 被选中的查询电话号码
      historyData: [], // 历史成绩数据

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
          title: "联系电话",
          key: "telephone",
          align: "center"
        },
        {
          title: "厨余垃圾分数",
          key: "kitchenWasteFraction",
          align: "center"
        },
        {
          title: "有害垃圾分数",
          key: "harmfulWasteFraction",
          align: "center"
        },
        {
          title: "可回收垃圾分数",
          key: "recyclableWasteFraction",
          align: "center"
        },
        {
          title: "其他垃圾分数",
          key: "otherWasteFraction",
          align: "center"
        },
        {
          title: "总评分",
          key: "scoreFraction",
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
    // 对话框的确认点击事件
    handleSubmit() {
      this.isShowHistory = false;
    },
    // 历史成绩点击事件
    handleHistory() {
      if (this.isSelectRow == true) {
        this.isShowHistory = true;
        // 发送请求到后台获取全部数据
        let _this = this;
        this.historyTelephone = this.selectRowData.telephone;
        axios
          .request({
            url: "grade/getHistoryGrade",
            method: "post",
            headers: {
              "Content-Type": "application/json;charset=UTF-8"
            },
            data: _this.historyTelephone
          })
          .then(function(response) {
            _this.historyData = response.data;
          });
      } else {
        this.$Message.error("请先选择记录");
      }
    },
    // 查询点击事件
    handleSearch(value) {
      this.searchData = value;
      this.pageCurrent = 1;
      this.getRequestData(this.pageCurrent);
    },

    // 选中行改变点击事件
    currentChange(currentRow, oldCurrentRow) {
      this.deleteData = currentRow;
      this.selectRowData = currentRow;

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
          url: "grade/getGradeList",
          method: "get",
          params: {
            search: this.searchData,
            start: this.pageStart,
            count: this.pageSize
          }
        })
        .then(function(response) {
          _this.pageData = response.data.gradeList;
          _this.dataCount = response.data.dataCount;
          _this.addPageCurrentAndPageSize(_this.pageData);
        });
    },

    // 添加当前页和页码数据（序号生成）
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
          content: "<p>是否确认将该记录删除？</p>",
          onOk: () => {
            let _this = this;
            let _data = this.deleteData;
            axios
              .request({
                url: "grade/deleteGrade",
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
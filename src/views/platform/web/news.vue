<template>
  <div class="font16 hgt_full" v-cloak>
    <div class="flex_column hgt_full">
      <div class="flex_1">
        <el-table
          ref="refElTabel"
          tooltip-effect="light"
          :data="newsListTable"
          border
          style="width: 100%" 
        >
          <el-table-column prop="Id" label="ID" width="50"></el-table-column>
          <el-table-column prop="Title" label="新闻标题" :show-overflow-tooltip="true">
            <template slot-scope="scope">
              <span
                class="color-1f85aa font-w6 cursor"
                @click="openMoreOptationDialog(scope.$index, scope.row)"
              >{{ scope.row.Title }}</span>
            </template>
          </el-table-column>
          <el-table-column label="新闻类型" width="100">
            <template slot-scope="scope">
              <span>{{common.FormatSelect(newsKindOptions,scope.row.KindID)}}</span>
            </template>
          </el-table-column>
          <el-table-column label="是否公共" width="100">
            <template slot-scope="scope">
              <span v-if="scope.row.Platform==0">公共新闻</span>
              <span v-else>校区新闻</span>
            </template>
          </el-table-column>
          <el-table-column prop="Creattime" :formatter="TimeFormatter" label="发布时间" width="130"></el-table-column>
          <el-table-column prop="AuthorLabel" label="发布人" width="100"></el-table-column>
          <el-table-column label="操作" width="100" fixed="right">
            <template slot-scope="scope">
              <el-button type="danger" @click="deleteNewsRow(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="between-center m-v-15">
        <el-button type="primary" @click="newsAdd">添加新闻</el-button>
        <el-pagination
          background
          @current-change=" currentPageChange"
          :current-page.sync="nowPage"
          :page-size="rows"
          layout="total,prev, pager, next, jumper"
          :total="allRows"
        ></el-pagination>
      </div>
    </div>
    <!-- 弹出框 -->
    <div>
      <my-dialog title="新闻详情编辑" :showLeft="false" :visible.sync="newsFormDialog">
        <div slot="right_content">
          <newsFormData
            ref="newsForm"
            :kindList="newsKindOptions"
            :platform="currentPlatform"
            :formItemData="currentItemData"
            @updateRowData="updateNewsList"
          ></newsFormData>
        </div>
      </my-dialog>
    </div>
  </div>
</template>

<script>
import myDialog from "@/components/myDialog/myDialog";
import common from "@/utils/common";
import newsFormData from "@/views/platform/web/component/newsFormData";
import { GetPlatformNews, deleNewsRow } from "@/api/news";
export default {
  name: "newsList",
  components: {
    myDialog,
    newsFormData
  },
  data() {
    return {
      common,
      // 新闻类型的选项
      newsKindOptions: [
        {
          value: 1,
          Label: "新闻"
        },
        {
          value: 2,
          Label: "学校展示"
        },
        {
          value: 3,
          Label: "最新活动"
        }
      ],
      // 新闻的数据列表
      newsListTable: [],
      // 数据总条数
      allRows: 0,
      // 当前页数
      nowPage: 1,
      // 每页数据的总条
      rows: 30,
      // 显示隐藏模态框
      newsFormDialog: false,
      // 模态框获得的单条数据
      currentItemData: null,
      // 当前索引
      currentNewsIndex: null,
      currentPlatform: 0,
      documentHeight:500,
    };
  },
  methods: {
    // 获取新闻的数据列表
    async GetPlatformNews() {
      let offsetRow = (this.nowPage - 1) * this.rows;
      let newParams = {
        needPublic: false,
        content: 1,
        limit: this.rows,
        offset: offsetRow
      };
      let res = await GetPlatformNews(this.currentPlatform, newParams);
      if (res.code == 200) {
        this.newsListTable = [];
        if (res.data) {
          this.newsListTable = res.data;
        }
        this.allRows = res.title;
      }
    },
    // 分页获取数据
    currentPageChange(val) {
      this.nowPage = val;
      this.GetPlatformNews();
    },
    // 删除新闻数据
    deleteNewsRow: function(index, row) {
      this.$confirm("你确定要删除吗？删了之后找不回来哦", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          this.currentNewsIndex = index;
          let res = await deleNewsRow(row.Id);
          if (res.code == 200) {
            this.GetPlatformNews();
          }
        })
        .catch(() => {});
    },
    // 打开编辑弹窗获取用户数据
    openMoreOptationDialog(index, row) {
      this.newsFormDialog = true;
      this.currentNewsIndex = index;
      this.currentItemData = row;
    },
    // 显示列表的时候格式化时间
    TimeFormatter(row, column, cellValue) {
      return this.common.dateFormat(cellValue, 2);
    },
    // 点击新增新闻
    newsAdd() {
      this.newsFormDialog = true;
      this.currentItemData = {
        icon: "/upload/icon/defaultnews.png",
        Id: 0,
        Downfile: "",
        Title: "",
        Description: "",
        Content: "没有写任何内容",
        KindId: 1
      };
    },
    // 编辑或者添加之后更新表格数据-新闻列表
    updateNewsList(rowData, isType) {
      // isType编辑还是添加
      if (isType == 1) {
        this.$set(this.newsListTable, this.currentNewsIndex, rowData);
      } else if (isType == 0) {
        this.newsListTable.unshift(rowData);
      }
      this.newsFormDialog = false;
    }
  },

  mounted() {
    let paths = this.$router.currentRoute.path.split("/");
    this.currentPlatform = parseInt(paths[paths.length - 1]);
    if (isNaN(this.currentPlatform)) {
      this.currentPlatform = 0;
    }
    
    console.log(document.body.clientHeight,"   this.documentHeight:",   this.documentHeight)
    this.GetPlatformNews();
  }
};
</script>
<style scope >
.selectStyle {
  border: none;
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  color: #606266;
  font-size: 14px;
}
.selectStyle::-ms-expand {
  display: none;
}
</style>


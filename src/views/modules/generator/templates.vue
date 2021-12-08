<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('generator:templates:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('generator:templates:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="模版名称"
      >
      </el-table-column>
      <el-table-column
        prop="industryId"
        header-align="center"
        align="center"
        label="模版行业"
      >
      </el-table-column>
      <el-table-column
        prop="type"
        header-align="center"
        align="center"
        label="设计类型"
      >
      </el-table-column>
      <el-table-column
        prop="loginDetail"
        header-align="center"
        align="center"
        label="模版登录地址"
      >
      </el-table-column>
      <el-table-column
        prop="designerId"
        header-align="center"
        align="center"
        label="责任设计师"
      >
      </el-table-column>
      <el-table-column
        prop="startedDate"
        header-align="center"
        align="center"
        label="开始日期"
      >
      </el-table-column>
      <el-table-column
        prop="finishedDate"
        header-align="center"
        align="center"
        label="完成日期"
      >
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态更新"
      >
      </el-table-column>
      <el-table-column
        prop="prodReady"
        header-align="center"
        align="center"
        label="是否能上 Prod"
      >
        <template slot-scope="scope">
          <el-switch
            :active-value="1"
            :inactive-value="0"
            v-model="scope.row.prodReady"
            disabled>
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        prop="prodLink"
        header-align="center"
        align="center"
        label="模版上线链接"
      >
      </el-table-column>
      <el-table-column
        prop="notes"
        header-align="center"
        align="center"
        label="备注"
      >
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.id)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      :designerList="designerList"
      :industryList="industryList"
      @refreshDataList="getDataList"
    ></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./templates-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: "",
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      designerList: [],
      industryList: [],
    };
  },
  components: {
    AddOrUpdate,
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // transform list
    transformList (list) {
      const getIndustryNameById = id => {
        const filterResult = this.industryList.filter(item => item.industryId === id)
        return filterResult.length > 0 ? filterResult[0].name : ''
      }
      const getDesignerNameById = id => {
        const filterResult = this.designerList.filter(item => item.designerId === id)
        return filterResult.length > 0 ? filterResult[0].name : ''
      }
      return list.map(item => {
        return {
          ...item,
          startedDate: item.startedDate.split(' ') ? item.startedDate.split(' ')[0] : '',
          finishedDate: item.finishedDate.split(' ') ? item.finishedDate.split(' ')[0] : '',
          industryId: getIndustryNameById(item.industryId),
          designerId: getDesignerNameById(item.designerId)
        }
      })
    },
    // get meta data (designId, industryId)
    getMetaData () {
      const promise = new Promise((resolve, reject) => {
        // use cache if we can
        if (this.designerList.length > 0 && this.industryList.length > 0) {
            resolve(true);
        }
        
        // call endpoint
        const designerPromise = this.$http({
          url: this.$http.adornUrl("/generator/templatedesigner/list"),
          method: "get",
          params: this.$http.adornParams({
            page: 1,
            limit: 100,
          }),
        })

        const industryPromise = this.$http({
          url: this.$http.adornUrl("/generator/templateindustry/list"),
          method: "get",
          params: this.$http.adornParams({
            page: 1,
            limit: 100,
          }),
        })

        Promise.all([designerPromise, industryPromise]).then((values) => {
          console.log(values);
          const {data: designPromRes} = values[0];
          const {data: industryPromRes} = values[1];
          this.designerList = designPromRes.page.list;
          this.industryList = industryPromRes.page.list;
          resolve(true);
        });

      });
      return promise;
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/generator/templates/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(async ({ data }) => {
        if (data && data.code === 0) {
          await this.getMetaData();
          // todo: transform the list
          this.dataList = this.transformList(data.page.list);
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/generator/templates/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>

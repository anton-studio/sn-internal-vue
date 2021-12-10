<template>
  <div class="mod-home">
    <h3>数据看板</h3>

    <el-row :gutter="20">
      <el-col :span="6">
        <el-card>
          <div class="stat-box">
            <div class="stat-box--metricName">总销售额</div>
            <div class="stat-box--metricValue">¥8888</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div class="stat-box">
            <div class="stat-box--metricName">总客户数</div>
            <div class="stat-box--metricValue">88</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div class="stat-box">
            <div class="stat-box--metricName">总上线模版</div>
            <div class="stat-box--metricValue">{{ totalProdTemplates }}</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div class="stat-box">
            <div class="stat-box--metricName">未转化客户</div>
            <div class="stat-box--metricValue">3</div>
          </div>
        </el-card>
      </el-col>
      <el-col :span="24">
        <el-card>
          <div id="J_chartBarTemplates" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div id="J_chartPieBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div id="J_chartPieBoxDesigner" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as echarts from "echarts";
export default {
  data() {
    return {
      chartLine: null,
      chartBarTemplates: null,
      chartPie: null,
      chartPieDesigner: null,
      chartScatter: null,
      templateList: [],
      designerList: [],
      industryList: [],
    };
  },
  mounted() {
    this.fetchMetaDateForPlot();
    this.initChartBarTemplates();
    this.initChartPie();
    this.initChartPieDesigner();
  },
  activated() {
    // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
    if (this.chartBarTemplates) {
      this.chartBarTemplates.resize();
    }
    if (this.chartPie) {
      this.chartPie.resize();
    }
    if (this.chartPieDesigner) {
      this.initChartPieDesigner.resize();
    }
  },
  methods: {
    fetchMetaDateForPlot() {
      const promise = new Promise((resolve, reject) => {
        // use cache if we can
        if (
          this.designerList.length > 0 &&
          this.industryList.length > 0 &&
          this.templateList.length > 0
        ) {
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
        });

        const industryPromise = this.$http({
          url: this.$http.adornUrl("/generator/templateindustry/list"),
          method: "get",
          params: this.$http.adornParams({
            page: 1,
            limit: 100,
          }),
        });

        const templatePromise = this.$http({
          url: this.$http.adornUrl("/generator/templates/list"),
          method: "get",
          params: this.$http.adornParams({
            page: 1,
            limit: 100,
          }),
        });

        Promise.all([designerPromise, industryPromise, templatePromise]).then(
          (values) => {
            const { data: designPromRes } = values[0];
            const { data: industryPromRes } = values[1];
            const { data: templatePromRes } = values[2];
            this.designerList = designPromRes.page.list;
            this.industryList = industryPromRes.page.list;
            this.templateList = templatePromRes.page.list;
            resolve(true);
          }
        );
      });
      return promise;
    },
    getIndustryNameById(id) {
      const filterResult = this.industryList.filter(
        (item) => item.industryId === id
      );
      return filterResult.length > 0 ? filterResult[0].name : "";
    },
    getDesignerNameById(id) {
      const filterResult = this.designerList.filter(
        (item) => item.designerId === id
      );
      return filterResult.length > 0 ? filterResult[0].name : "";
    },
    async initChartBarTemplates() {
      await this.fetchMetaDateForPlot();

      const allTemplate = [...this.templateList];

      const designingGroup = {};
      const prodReadyGroup = {};
      for (let i = 0; i < allTemplate.length; i++) {
        const template = allTemplate[i];
        if (designingGroup[template.industryId] == undefined) {
          designingGroup[template.industryId] = 0;
          prodReadyGroup[template.industryId] = 0;
        }
        if (template.prodReady == 1) {
          prodReadyGroup[template.industryId] += 1;
          continue;
        }
        if (template.status == "设计中") {
          designingGroup[template.industryId] += 1;
          continue;
        }
      }
      const industryNames = Object.keys(designingGroup).map((id) =>
        this.getIndustryNameById(Number(id))
      );
      const designingValue = Object.keys(designingGroup).map(
        (key) => designingGroup[key]
      );
      const prodReadyValue = Object.keys(prodReadyGroup).map(
        (key) => prodReadyGroup[key]
      );

      var option = {
        title: {
          text: "模版行业覆盖",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
        },
        legend: {},
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: {
          type: "value",
          boundaryGap: [0, 0.01],
        },
        yAxis: {
          type: "category",
          data: [...industryNames],
        },
        series: [
          {
            name: "设计中",
            type: "bar",
            data: [...designingValue],
          },
          {
            name: "已上线",
            type: "bar",
            data: [...prodReadyValue],
          },
        ],
      };
      this.chartBarTemplates = echarts.init(
        document.getElementById("J_chartBarTemplates")
      );
      this.chartBarTemplates.setOption(option);
      window.addEventListener("resize", () => {
        this.chartBarTemplates.resize();
      });
    },
    initChartPie() {
      var option = {
        title: {
          text: "客户分类",
          subtext: "",
          left: "center",
        },
        tooltip: {
          trigger: "item",
        },
        // legend: {
        //   orient: "vertical",
        //   left: "left",
        // },
        series: [
          {
            name: "客户类型",
            type: "pie",
            radius: "50%",
            data: [
              { value: 1048, name: "注册未购买" },
              { value: 735, name: "购买未上线" },
              { value: 580, name: "已上线(客户运营)" },
              { value: 484, name: "已上线(慢慢来运营)" },
            ],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
          },
        ],
      };
      this.chartPie = echarts.init(document.getElementById("J_chartPieBox"));
      this.chartPie.setOption(option);
      window.addEventListener("resize", () => {
        this.chartPie.resize();
      });
    },
    async initChartPieDesigner() {
      await this.fetchMetaDateForPlot();

      const allTemplate = [...this.templateList];

      const designingGroup = {};
      for (let i = 0; i < allTemplate.length; i++) {
        const template = allTemplate[i];
        if (template.prodReady == 1) {
          if (designingGroup[template.designerId] == undefined) {
            designingGroup[template.designerId] = 0;
          }
          designingGroup[template.designerId] += 1;
        }
      }
      const designingValue = Object.keys(designingGroup).map((key) => ({
        name: this.getDesignerNameById(Number(key)), // designer
        value: designingGroup[key],
      }));

      var option = {
        title: {
          text: "上线模版数",
          subtext: "",
          left: "center",
        },
        tooltip: {
          trigger: "item",
        },
        // legend: {
        //   orient: "vertical",
        //   left: "left",
        // },
        series: [
          {
            name: "设计师",
            type: "pie",
            // radius: "50%",
            radius: ['40%', '70%'],
            data: [...designingValue],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)",
              },
            },
          },
        ],
      };
      this.chartPie = echarts.init(
        document.getElementById("J_chartPieBoxDesigner")
      );
      this.chartPie.setOption(option);
      window.addEventListener("resize", () => {
        this.chartPie.resize();
      });
    },
  },
  computed: {
    totalProdTemplates: function () {
      return this.templateList.filter((template) => template.prodReady == 1)
        .length;
    },
  },
};
</script>

<style lang="scss">
.mod-home {
  line-height: 1.5;
  > .el-row {
    margin-top: -10px;
    margin-bottom: -10px;
    .el-col {
      padding-top: 10px;
      padding-bottom: 10px;
    }
  }
  .chart-box {
    min-height: 400px;
  }
  #J_chartBarTemplates {
    min-height: 600px;
  }

  .stat-box {
    text-align: center;

    .stat-box--metricName {
      color: rgba(0, 0, 0, 0.54);
    }

    .stat-box--metricValue {
      font-size: 36px;
    }
  }
}
</style>


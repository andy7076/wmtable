<template>
  <div class="wm-table">
    <div class="query-panel" v-if="query">
      <el-row>
        <el-col
          class="query-panel-col"
          v-for="(column, index) in queryColumns"
          :key="index"
          :xs="24"
          :sm="12"
          :md="8"
          :lg="6"
        >
          <el-input
            size="small"
            v-if="column.query.type === 'text'"
            v-model="queryForm[column.prop]"
            :placeholder="column.query.placeholder"
          ></el-input>
          <el-select
            style="width: 100%"
            v-if="column.query.type === 'select'"
            v-model="queryForm[column.prop]"
            :placeholder="column.query.placeholder"
            size="small"
          >
            <el-option
              v-for="(column, index) in column.query.columns"
              :key="index"
              :label="column.label"
              :value="column.value"
            >
            </el-option>
          </el-select>
        </el-col>
        <el-col class="query-operate" :xs="24" :sm="12" :md="8" :lg="6">
          <el-button
            type="primary"
            size="small"
            icon="el-icon-search"
            @click="btnConfirm"
            >查询</el-button
          >
          <el-button
            plain
            icon="el-icon-refresh-right"
            size="small"
            class="query-confirm"
            @click="resetForm"
            >重置</el-button
          >
        </el-col>
      </el-row>
    </div>
    <div class="table-panel" v-if="panel">
      <el-row>
        <slot name="table-panel"></slot>
      </el-row>
    </div>
    <el-table
      v-loading="loading"
      :data="data.list || []"
      :stripe="stripe"
      :border="border"
      :row-class-name="rowClassName"
      :height="height"
      :rowKey="rowKey"
      :max-height="maxHeight"
    >
      <el-table-column
        :key="index"
        :prop="column.prop || ''"
        :label="column.label || ''"
        :fixed="column.fixed || false"
        :width="column.width || ''"
        v-for="(column, index) in columns"
      >
        <slot
          v-if="column.customRender"
          slot-scope="scope"
          :name="column.customRender"
          :record="scope.row"
          :index="scope.$index"
          :text="scope.row[scope.column.property]"
        >
        </slot>
        <slot slot-scope="scope" v-else>{{
          scope.row[scope.column.property]
        }}</slot>
      </el-table-column>
    </el-table>
    <el-pagination
      layout="prev, pager, next"
      :page-size="pageSize"
      :pager-count="11"
      :total="data.total || 0"
      class="pagination"
      @current-change="handleCurrentChange"
    >
    </el-pagination>
  </div>
</template>

<script>
/**
 * Filter collapse以及时间过滤功能 表单修改为form,以便增加form的validate功能
 * 图表操作选项区域
 * Table待完善 多级表头 单选 多选 排序 筛选 展开行 树形数据与懒加载 表尾合计行 合并行或列 自定义索引
 * pagination选择器 每页数量选择
 */
export default {
  name: "wmTable",
  props: {
    columns: {
      type: Array,
      required: true,
    },
    data: {
      type: Object,
      required: true,
    },
    pagination: {
      type: Boolean,
      default: false,
    },
    stripe: {
      type: Boolean,
      default: false,
    },
    border: {
      type: Boolean,
      default: false,
    },
    rowClassName: {
      type: String,
      default: undefined,
    },
    height: {
      type: [String, Number],
      default: undefined,
    },
    rowKey: {
      type: [String, Function],
      default: undefined,
    },
    maxHeight: {
      type: [String, Number],
      default: undefined,
    },
    onRequest: {
      type: Function,
      default: () => {},
    },
    loading: {
      type: Boolean,
      default: false,
    },
    query: {
      type: Boolean,
      default: false,
    },
    panel: {
      type: Boolean,
      default: false,
    },
  },
  watch: {
    data(data) {
      this.handleDataMatchColumns(data.list);
    },
  },
  data() {
    return {
      currentPage: 1,
      pageSize: 10,
      queryColumns: [],
      queryForm: {},
    };
  },
  mounted() {
    this.reload();

    const { data, columns } = this;
    data.list && this.handleDataMatchColumns(data.list);

    const queryColumns = [];
    columns.forEach((column) => {
      const { query, prop } = column;
      if (query) {
        queryColumns.push(column);
        if (query.type === "select") {
          this.queryForm[prop] = query.default || "";
        }
      }
    });
    this.queryColumns = queryColumns;
  },
  methods: {
    btnConfirm() {
      this.reload();
    },
    reload() {
      const { currentPage, pageSize, queryForm } = this;
      this.onRequest({ currentPage, pageSize, filter: queryForm });
    },
    resetForm() {
      this.queryForm = {};
    },
    handleDataMatchColumns(data) {
      const { columns } = this;
      columns.forEach((column) => {
        const { prop, customTextRender } = column;
        if (customTextRender) {
          if (typeof customTextRender !== "function") {
            throw new TypeError("customRender的类型为函数类型");
          }
          data.forEach((item) => {
            item[prop] = customTextRender(item[prop], item);
          });
        }
      });
    },
    handleCurrentChange() {
      this.reload();
    },
  },
};
</script>

<style lang="less" scoped>
.wm-table {
  .query-panel {
    .query-panel-col {
      padding-right: 24px;
      margin-bottom: 16px;
    }

    .query-operate {
      display: flex;
      align-items: center;
      margin-bottom: 16px;

      .query-confirm {
        margin-right: 8px;
      }
    }
  }

  .table-panel {
    margin-bottom: 16px;
  }

  .pagination {
    float: right;
    margin: 16px 0;
  }
}
</style>
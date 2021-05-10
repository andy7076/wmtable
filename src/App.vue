<template>
  <div id="app">
    <el-card class="card-box">
      <wm-table
        :columns="columns"
        :data="sourceData"
        :stripe="false"
        :border="false"
        :rowKey="(record) => record.id"
        :onRequest="getUserList"
        :loading="loading"
        :query="true"
        :panel="true"
        ref="wmTable"
      >
        <template slot="table-panel">
          <el-button
            size="small"
            type="primary"
            icon="el-icon-plus"
            @click="addUser"
            >新增用户</el-button
          >
        </template>
        <template slot="status" slot-scope="scope">
          <el-tag size="small" :type="scope.text | statusTagType">{{
            scope.text | statusNameFilter
          }}</el-tag>
        </template>
        <template slot="createTime" slot-scope="scope">
          <el-tag size="small">{{ scope.text }}</el-tag>
        </template>
        <template slot="action" slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.index, scope.record)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.index, scope.record)"
            >删除</el-button
          >
        </template>
      </wm-table>
    </el-card>
  </div>
</template>

<script>
import WmTable from "./components/WmTable";
import dayjs from "dayjs";
import getRandomFruitsName from "random-fruits-name";
import { nanoid } from "nanoid";

export default {
  name: "App",
  components: {
    WmTable,
  },
  filters: {
    statusNameFilter(status) {
      switch (status) {
        case 0:
          return "状态1";
        case 1:
          return "状态2";
        case 2:
          return "状态3";
      }
    },
    statusTagType(status) {
      switch (status) {
        case 0:
          return "success";
        case 1:
          return "info";
        case 2:
          return "warning";
      }
    },
  },
  data() {
    return {
      columns: [
        {
          prop: "id",
          label: "id",
          width: "280",
          query: {
            type: "text",
            placeholder: "请填写用户id",
          },
        },
        {
          prop: "name",
          label: "姓名",
          width: "280",
        },
        {
          prop: "age",
          label: "年龄",
        },
        {
          prop: "status",
          label: "状态",
          customRender: "status",
          query: {
            type: "select",
            default: 0,
            placeholder: "请选择状态",
            columns: [
              {
                value: 0,
                label: "全部",
              },
              {
                value: 0,
                label: "状态1",
              },
              {
                value: 1,
                label: "状态2",
              },
              {
                value: 2,
                label: "状态3",
              },
            ],
          },
        },
        {
          prop: "createTime",
          label: "创建时间",
          width: "160",
          customRender: "createTime",
          customTextRender: (text) => dayjs(text).format("YYYY-MM-DD HH:mm"),
        },
        {
          prop: "action",
          label: "操作",
          fixed: "right",
          customRender: "action",
          width: "160",
        },
      ],
      sourceData: {},
      loading: false,
    };
  },
  methods: {
    async getUserList({ currentPage, pageSize, filter }) {
      this.loading = true;
      console.log(currentPage, pageSize, filter);
      await this.delayTime(1000);
      const list = [];
      for (let i = 0; i < 10; i++) {
        list.push({
          id: nanoid(),
          name: getRandomFruitsName(),
          age: Math.floor(Math.random() * 100),
          status: Math.floor(Math.random() * 3),
          createTime: new Date().getTime(),
        });
      }
      this.data = {
        code: 0,
        data: {
          total: 80,
          currentPage,
          list,
        },
        message: "",
      };
      this.sourceData = this.data.data;
      this.loading = false;
    },
    addUser() {
      this.$refs.wmTable.reload();
    },
    handleEdit(index, record) {
      console.log(index, record);
    },
    handleDelete(index, record) {
      console.log(index, record);
    },
    delayTime(time) {
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve();
        }, time);
      });
    },
  },
};
</script>

<style lang="less" scoped>
.card-box {
}
</style>
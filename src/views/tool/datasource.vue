<template>
  <basic-container>
    <avue-crud :option="option"
               :table-loading="loading"
               :data="data"
               :page.sync="page"
               :permission="permissionList"
               :before-open="beforeOpen"
               v-model="form"
               ref="crud"
               @row-update="rowUpdate"
               @row-save="rowSave"
               @row-del="rowDel"
               @search-change="searchChange"
               @search-reset="searchReset"
               @selection-change="selectionChange"
               @current-change="currentChange"
               @size-change="sizeChange"
               @refresh-change="refreshChange"
               @on-load="onLoad">
      <template slot="menuLeft">
        <el-button type="danger"
                   size="small"
                   icon="el-icon-delete"
                   plain
                   v-if="permission.datasource_delete"
                   @click="handleDelete">删 除
        </el-button>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
  import {getList, getDetail, add, update, remove} from "@/api/tool/datasource";
  import {mapGetters} from "vuex";
  import func from "@/util/func";

  export default {
    data() {
      return {
        form: {},
        query: {},
        loading: true,
        page: {
          pageSize: 10,
          currentPage: 1,
          total: 0
        },
        selectionList: [],
        option: {
          height: 'auto',
          calcHeight: 30,
          dialogWidth: 900,
          tip: false,
          searchShow: true,
          searchMenuSpan: 6,
          border: true,
          index: true,
          viewBtn: true,
          selection: true,
          dialogClickModal: false,
          column: [
            {
              label: '数据类型',
              type: 'radio',
              value: 1,
              span: 24,
              width: 120,
              searchLabelWidth: 100,
              row: true,
              display: false,
              dicData: [
                {
                  label: 'jdbc',
                  value: 1,
                },
              ],
              dataType: 'number',
              prop: 'category',
              rules: [
                {
                  required: true,
                  message: '请选择分类',
                  trigger: 'blur',
                },
              ],
            },
            {
              label: "名称",
              prop: "name",
              width: 120,
              span: 24,
              search: true,
              rules: [{
                required: true,
                message: "请输入数据源名称",
                trigger: "blur"
              }]
            },
            {
              label: 'YAML',
              prop: 'shardingConfig',
              span: 24,
              minRows: 5,
              hide: true,
              display: false,
              type: 'textarea',
              rules: [
                {
                  required: true,
                  message: '请输入YAML配置',
                  trigger: 'blur',
                },
              ],
            },
            {
              label: "驱动类",
              prop: "driverClass",
              type: 'select',
              span: 24,
              dicData: [
                {
                  label: 'com.mysql.cj.jdbc.Driver',
                  value: 'com.mysql.cj.jdbc.Driver',
                }, {
                  label: 'org.postgresql.Driver',
                  value: 'org.postgresql.Driver',
                }, {
                  label: 'oracle.jdbc.OracleDriver',
                  value: 'oracle.jdbc.OracleDriver',
                }
              ],
              width: 200,
              display: true,
              rules: [{
                required: true,
                message: "请输入驱动类",
                trigger: "blur"
              }]
            },
            {
              label: "用户名",
              prop: "username",
              width: 120,
              display: true,
              rules: [{
                required: true,
                message: "请输入用户名",
                trigger: "blur"
              }]
            },
            {
              label: "密码",
              prop: "password",
              hide: true,
              display: true,
              rules: [{
                required: true,
                message: "请输入密码",
                trigger: "blur"
              }]
            },
            {
              label: "连接地址",
              prop: "url",
              span: 24,
              display: true,
              rules: [{
                required: true,
                message: "请输入连接地址",
                trigger: "blur"
              }]
            },
            {
              label: "备注",
              prop: "remark",
              span: 24,
              minRows: 3,
              hide: true,
              type: "textarea"
            },
          ]
        },
        data: []
      };
    },
    watch: {
      'form.category'() {
        const category = func.toInt(this.form.category);
        this.$refs.crud.option.column.filter(item => {
          if (item.prop === 'driverClass') {
            item.display = category === 1;
          }
          if (item.prop === 'url') {
            item.display = category === 1;
          }
          if (item.prop === 'username') {
            item.display = category === 1;
          }
          if (item.prop === 'password') {
            item.display = category === 1;
          }
          if (item.prop === 'shardingConfig') {
            item.display = category === 2;
          }
        });
      },
    },
    computed: {
      ...mapGetters(["permission"]),
      permissionList() {
        return {
          addBtn: this.vaildData(this.permission.datasource_add, false),
          viewBtn: this.vaildData(this.permission.datasource_view, false),
          delBtn: this.vaildData(this.permission.datasource_delete, false),
          editBtn: this.vaildData(this.permission.datasource_edit, false)
        };
      },
      ids() {
        let ids = [];
        this.selectionList.forEach(ele => {
          ids.push(ele.id);
        });
        return ids.join(",");
      }
    },
    methods: {
      rowSave(row, done, loading) {
        add(row).then(() => {
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          done();
        }, error => {
          window.console.log(error);
          loading();
        });
      },
      rowUpdate(row, index, done, loading) {
        update(row).then(() => {
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          done();
        }, error => {
          window.console.log(error);
          loading();
        });
      },
      rowDel(row) {
        this.$confirm("确定将选择数据删除?", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            return remove(row.id);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: "操作成功!"
            });
          });
      },
      handleDelete() {
        if (this.selectionList.length === 0) {
          this.$message.warning("请选择至少一条数据");
          return;
        }
        this.$confirm("确定将选择数据删除?", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            return remove(this.ids);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: "操作成功!"
            });
            this.$refs.crud.toggleSelection();
          });
      },
      beforeOpen(done, type) {
        if (["edit", "view"].includes(type)) {
          getDetail(this.form.id).then(res => {
            this.form = res.data.data;
          });
        }
        done();
      },
      searchReset() {
        this.query = {};
        this.onLoad(this.page);
      },
      searchChange(params, done) {
        this.query = params;
        this.page.currentPage = 1;
        this.onLoad(this.page, params);
        done();
      },
      selectionChange(list) {
        this.selectionList = list;
      },
      selectionClear() {
        this.selectionList = [];
        this.$refs.crud.toggleSelection();
      },
      currentChange(currentPage) {
        this.page.currentPage = currentPage;
      },
      sizeChange(pageSize) {
        this.page.pageSize = pageSize;
      },
      refreshChange() {
        this.onLoad(this.page, this.query);
      },
      onLoad(page, params = {}) {
        this.loading = true;
        getList(page.currentPage, page.pageSize, Object.assign(params, this.query)).then(res => {
          const data = res.data.data;
          this.page.total = data.total;
          this.data = data.records;
          this.loading = false;
          this.selectionClear();
        });
      }
    }
  };
</script>

<style>
</style>

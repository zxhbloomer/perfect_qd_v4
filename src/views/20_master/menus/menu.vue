<template>
  <div class="app-container">
    <el-form
      ref="minusForm"
      :inline="true"
      :model="dataJson.searchForm"
      label-position="getLabelPosition()"
      class="floatRight"
    >
      <el-form-item label="">
        <el-input v-model.trim="dataJson.searchForm.name" clearable placeholder="名称" />
      </el-form-item>
      <el-form-item label="">
        <select-dict v-model="dataJson.searchForm.visible" :para="CONSTANTS.DICT_SYS_VISIBLE_TYPE" init-placeholder="请选择菜单类型" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" plain icon="el-icon-search" @click="handleSearch">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-popover:popover type="primary" plain icon="perfect-icon-reset" @click="doResetSearch">重置</el-button>
      </el-form-item>
    </el-form>
    <el-button-group v-show="!meDialogStatus.dialogStatus">
      <el-button type="primary" icon="el-icon-circle-plus-outline" :loading="settings.listLoading" @click="handleInsert">新增菜单组</el-button>
      <el-button :disabled="!settings.btnShowStatus.showAddSubNode" type="primary" icon="el-icon-edit-outline" :loading="settings.listLoading" @click="handleAddSubNode">添加子菜单-结点</el-button>
      <el-button :disabled="!settings.btnShowStatus.showUpdate" type="primary" icon="el-icon-edit-outline" :loading="settings.listLoading" @click="handleUpdate">修改</el-button>
      <el-button :disabled="!settings.btnShowStatus.showExport" type="primary" icon="el-icon-circle-close" :loading="settings.listLoading" @click="handleRealyDelete">物理删除</el-button>
    </el-button-group>
    <el-table
      ref="multipleTable"
      v-loading="settings.listLoading"
      :data="dataJson.listData"
      :element-loading-text="'正在拼命加载中...'"
      element-loading-background="rgba(255, 255, 255, 0.5)"
      :height="settings.tableHeight"
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
      stripe
      border
      fit
      highlight-current-row
      style="width: 100%"
      default-expand-all
      row-key="id"
      @row-click="handleRowClick"
      @current-change="handleCurrentChange"
      @selection-change="handleSelectionChange"
    >
      <el-table-column header-align="center" type="index" width="45" />
      <el-table-column header-align="center" show-overflow-tooltip min-width="150" prop="name" label="菜单名称" />
      <!-- <el-table-column header-align="center" show-overflow-tooltip min-width="150" prop="code" label="菜单编号" /> -->
      <!-- <el-table-column header-align="center" show-overflow-tooltip min-width="80" prop="type_name" label="菜单类型" /> -->
      <el-table-column header-align="center" label="按钮">
        <el-table-column
          v-for="button_column in dataJson.menu_buttons"
          :key="button_column.code"
          align="center"
          :prop="button_column.code"
          :label="button_column.name"
          min-width="25"
        >
          <template v-slot="column_lists">
            <div v-for="item in column_lists.row.function_info" :key="item.id">
              <!-- 存在按钮数据时 -->
              <div v-if="item.code === button_column.code">
                〇
              </div>
            </div>
            <!-- 不存在按钮数据时 -->
            <div v-if="column_lists.row.function_info.length === 0">
              -
            </div>
          </template>
        </el-table-column>
      </el-table-column>
    </el-table>

    <!-- 新增菜单组 dialog -->
    <edit-group-dialog
      v-if="popSettings.one.visible"
      :visible="popSettings.one.visible"
      :dialog-status="popSettings.one.props.dialogStatus"
      :data="popSettings.one.props.data"
      @closeMeOk="handleEditGroupDialogCloseMeOk"
      @closeMeCancel="handleEditGroupDialogCloseMeCancel"
    />

    <!-- 添加子菜单-结点 dialog editSubNodeDialog-->
    <edit-sub-node-dialog
      v-if="popSettings.two.visible"
      :visible="popSettings.two.visible"
      :dialog-status="popSettings.two.props.dialogStatus"
      :data="popSettings.two.props.data"
      @closeMeOk="handleEditSubNodeDialogCloseMeOk"
      @closeMeCancel="handleEditSubNodeDialogCloseMeCancel"
    />

    <iframe id="refIframe" ref="refIframe" scrolling="no" frameborder="0" style="display:none" name="refIframe">x</iframe>
  </div>
</template>

<style scoped>
  .floatRight {
    float: right;
  }
  .floatLeft {
    float: left;
  }
  .el-form-item .el-select {
    width: 100%;
  }
  .grid-content {
    border-radius: 2px;
    min-height: 36px;
    margin-bottom: 10px;
  }
  .bg-purple-light {
    background: #e5e9f2;
  }
</style>
<style >
  .el-input-group__append_select{
    color: #FFFFFF;
    background-color: #1890ff;
    border-color: #1890ff;
  }
  .el-input-group__append_reset{
    color: #FFFFFF;
    background-color: #F56C6C;
    border-color: #F56C6C;
  }
</style>

<script>
import constants_program from '@/common/constants/constants_program'
import { getListApi, deleteApi, realDeleteSelectionApi } from '@/api/20_master/menus/menu'
import resizeMixin from './menuResizeHandlerMixin'
import elDragDialog from '@/directive/el-drag-dialog'
import SelectDict from '@/components/00_dict/select/SelectDict'
import editGroupDialog from '@/views/20_master/menus/dialog/editGroup'
import editSubNodeDialog from '@/views/20_master/menus/dialog/editSubNode'
import deepCopy from 'deep-copy'

export default {
  name: constants_program.P_MENU, // 页面id，和router中的name需要一致，作为缓存
  components: { SelectDict, editGroupDialog, editSubNodeDialog },
  directives: { elDragDialog },
  mixins: [resizeMixin],
  props: {
    // 自己作为弹出框时的参数
    meDialogStatus: {
      type: Boolean,
      default: false
    },
    dataModel: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      dataJson: {
        // 级联选择器数据
        cascader: {
          data: null,
          value: ''
        },
        // 查询使用的json
        searchForm: {
          // 翻页条件
          pageCondition: deepCopy(this.PARAMETERS.PAGE_CONDITION),
          // 查询条件
          name: '',
          code: '',
          visible: 'null',
          is_del: '0' // 未删除
        },
        // 分页控件的json
        paging: deepCopy(this.PARAMETERS.PAGE_JSON),
        // table使用的json
        listData: null,
        // 按钮json
        menu_buttons: null,
        // 单条数据 json
        currentJson: null,
        // 当前表格中的索引，第几条
        rowIndex: 0,
        // 当前选中的行（checkbox）
        multipleSelection: []
      },
      // 页面设置json
      settings: {
        // 表格排序规则
        sortOrders: deepCopy(this.PARAMETERS.SORT_PARA),
        // 按钮状态
        btnShowStatus: {
          showUpdate: false,
          // 添加子菜单-结点：按钮
          showAddSubNode: false,
          showCopyInsert: false,
          showExport: false
        },
        // loading 状态
        listLoading: true,
        tableHeight: this.setUIheight(),
        duration: 4000
      },
      popSettings: {
        // 弹出编辑页面
        one: {
          visible: false,
          props: {
            id: undefined,
            data: {},
            dialogStatus: ''
          }
        },
        two: {
          visible: false,
          props: {
            id: undefined,
            data: {},
            dialogStatus: ''
          }
        }
      }
    }
  },
  computed: {
  },
  // 监听器
  watch: {
    // 选中的数据，使得导出按钮可用，否则就不可使用
    'dataJson.multipleSelection': {
      handler(newVal, oldVal) {
        if (newVal.length > 0) {
          this.settings.btnShowStatus.showExport = true
        } else {
          this.settings.btnShowStatus.showExport = false
        }
      }
    },
    // 当前行的选中
    'dataJson.currentJson': {
      handler(newVal, oldVal) {
        if (this.dataJson.currentJson.id !== undefined) {
          // this.settings.btnShowStatus.doInsert = true
          this.settings.btnShowStatus.showAddSubNode = true
          this.settings.btnShowStatus.showUpdate = true
          this.settings.btnShowStatus.showCopyInsert = true
          this.settings.btnShowStatus.showExport = true
        } else {
          // this.settings.btnShowStatus.doInsert = false
          this.settings.btnShowStatus.showUpdate = false
          this.settings.btnShowStatus.showCopyInsert = false
          this.settings.btnShowStatus.showExport = false
        }
      },
      deep: true
    }
  },
  created() {
    this.initShow()
  },
  mounted() {
    // 描绘完成
  },
  methods: {
    initShow() {
      // 初始化查询
      this.getDataList()
    },
    // 下拉选项控件事件
    handleSelectChange(val) {
    },
    // 获取行索引
    getRowIndex(row) {
      const _index = this.dataJson.listData.lastIndexOf(row)
      return _index
    },
    // 行点击
    handleRowClick(row) {
      this.dataJson.rowIndex = this.getRowIndex(row)
    },
    handleSearch() {
      // 查询
      this.dataJson.searchForm.pageCondition.current = 1
      this.dataJson.paging.current = 1
      this.getDataList()
      // 清空选择
      this.dataJson.multipleSelection = []
      this.$refs.multipleTable.clearSelection()
      // this.dataJson.currentJson.id = undefined
    },
    handleRowUpdate(row, _rowIndex) {
      this.dataJson.rowIndex = _rowIndex
    },
    // 删除操作
    handleDel(row) {
      let _message = ''
      const _value = row.is_del
      const selectionJson = []
      selectionJson.push({ 'id': row.id })
      if (_value === true) {
        _message = '是否要删除选择的数据？'
      } else {
        _message = '是否要复原该条数据？'
      }
      // 选择全部的时候
      this.$confirm(_message, '确认信息', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确认',
        cancelButtonText: '取消'
      }).then(() => {
        // loading
        this.settings.listLoading = true
        deleteApi(selectionJson).then((_data) => {
          this.$notify({
            title: '更新处理成功',
            message: _data.message,
            type: 'success',
            duration: this.settings.duration
          })
        }, (_error) => {
          this.$notify({
            title: '更新处理失败',
            message: _error.message,
            type: 'error',
            duration: this.settings.duration
          })
          row.is_del = !row.is_del
        }).finally(() => {
          this.settings.listLoading = false
        })
      }).catch(action => {
        row.is_del = !row.is_del
      })
    },
    // 点击按钮 新增
    handleInsert() {
      // 新增
      this.popSettings.one.props.dialogStatus = this.PARAMETERS.STATUS_INSERT
      this.popSettings.one.visible = true
    },
    // 点击按钮 更新
    handleUpdate() {
      // 没有选择任何数据的情况
      if (this.dataJson.currentJson.id === undefined) {
        this.showErrorMsg('请选择一条数据')
        return
      }
      switch (this.dataJson.currentJson.type) {
        // 根节点编辑
        case this.CONSTANTS.DICT_SYS_MENU_TYPE_ROOT:
          this.popSettings.one.props.data = deepCopy(this.dataJson.currentJson)
          this.popSettings.one.visible = true
          this.popSettings.one.props.dialogStatus = this.PARAMETERS.STATUS_UPDATE
          break
      }
    },
    // 点击按钮 添加子菜单-结点
    handleAddSubNode() {
      this.popSettings.two.props.dialogStatus = this.PARAMETERS.STATUS_INSERT
      this.popSettings.two.props.data = deepCopy(this.dataJson.currentJson)
      this.popSettings.two.visible = true
    },
    handleCurrentChange(row) {
      this.dataJson.currentJson = Object.assign({}, row) // copy obj
      this.dataJson.currentJson.index = this.getRowIndex(row)

      // 设置dialog的返回
      this.$store.dispatch('popUpSearchDialog/selectedDataJson', Object.assign({}, row))
    },
    handleSortChange(column) {
      // 服务器端排序
      if (column.order === 'ascending') {
        this.dataJson.searchForm.pageCondition.sort = column.prop
      } else if (column.order === 'descending') {
        this.dataJson.searchForm.pageCondition.sort = '-' + column.prop
      }
      this.getDataList()
    },
    getDataList() {
      this.dataJson.searchForm.pageCondition.current = this.dataJson.paging.current
      this.dataJson.searchForm.pageCondition.size = this.dataJson.paging.size
      // 查询逻辑
      this.settings.listLoading = true
      getListApi(this.dataJson.searchForm).then(response => {
        // 增加对象属性，columnTypeShowIcon，columnNameShowIcon
        const recorders = response.data.menu_data
        this.dataJson.listData = recorders
        this.dataJson.menu_buttons = response.data.menu_buttons
        this.dataJson.paging = response.data.menu_data
        this.dataJson.paging.records = {}
      }).finally(() => {
        this.settings.listLoading = false
      })
    },
    // 重置查询区域
    doResetSearch() {
      this.dataJson.searchForm = this.$options.data.call(this).dataJson.searchForm
    },
    // 获取row-key
    getRowKeys(row) {
      return row.id
    },
    // table选择框
    handleSelectionChange(arr) {
      arr.forEach(function(val, index, arr) {
        console.log(val, index, arr)
      })
      this.dataJson.multipleSelection = arr
    },
    // 删除按钮
    handleRealyDelete() {
      // 没有选择任何数据的情况
      if (this.dataJson.tempJson.id === undefined) {
        this.showErrorMsg('请选择一条数据')
        return
      }
      // 选择全部的时候
      this.$confirm('请注意：将会删除当前节点以及子节点数据！！', '确认信息', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(() => {
        this.settings.listLoading = true
        this.handleRealDeleteData()
      }).catch(action => {
        // 右上角X
        if (action !== 'close') {
          // 当前页所选择的数据导出
        }
      })
    },
    // 选中数据删除
    handleRealDeleteData() {
      // loading
      this.settings.listLoading = true
      const tempData = Object.assign({}, this.dataJson.tempJson)
      // 开始删除
      realDeleteSelectionApi(tempData).then((_data) => {
        this.$notify({
          title: '删除成功',
          message: _data.message,
          type: 'success',
          duration: this.settings.duration
        })
        this.getDataList()
        this.dataJson.multipleSelection = []
        // loading
        this.settings.listLoading = false
      }, (_error) => {
        this.$notify({
          title: '删除错误',
          message: _error.message,
          type: 'error',
          duration: this.settings.duration
        })
        this.settings.listLoading = false
      })
    },
    // -----------------新增菜单组 start------------------
    handleEditGroupDialogCloseMeOk(val) {
      switch (this.popSettings.one.props.dialogStatus) {
        case this.PARAMETERS.STATUS_INSERT:
          this.doInsertEditGrouplCallBack(val)
          break
        case this.PARAMETERS.STATUS_UPDATE:
          this.doUpdateEditGroupCallBack(val)
          break
      }
    },
    handleEditGroupDialogCloseMeCancel() {
      this.popSettings.one.visible = false
    },
    // 处理插入回调
    doInsertEditGrouplCallBack(val) {
      if (val.return_flag) {
        this.popSettings.one.visible = false

        // 设置到table中绑定的json数据源
        this.dataJson.listData.push(val.data.data)
        this.$notify({
          title: '新增菜单组处理成功',
          message: val.data.message,
          type: 'success',
          duration: this.settings.duration
        })
      } else {
        this.$notify({
          title: '新增菜单组处理失败',
          message: val.error.message,
          type: 'error',
          duration: this.settings.duration
        })
      }
    },
    // 处理更新回调
    doUpdateEditGroupCallBack(val) {
      if (val.return_flag) {
        this.popSettings.one.visible = false

        // 设置到table中绑定的json数据源
        this.dataJson.listData.splice(this.dataJson.rowIndex, 1, val.data.data)
        // 设置到currentjson中
        this.dataJson.currentJson = Object.assign({}, val.data.data)
        this.$notify({
          title: '更新处理成功',
          message: val.data.message,
          type: 'success',
          duration: this.settings.duration
        })
      } else {
        this.$notify({
          title: '更新处理失败',
          message: val.error.message,
          type: 'error',
          duration: this.settings.duration
        })
      }
    },
    // -----------------新增菜单组 end------------------
    // -----------------添加子菜单-结点 start------------------
    handleEditSubNodeDialogCloseMeOk(val) {
      switch (this.popSettings.two.props.dialogStatus) {
        case this.PARAMETERS.STATUS_INSERT:
          this.doInsertEditSubNodelCallBack(val)
          break
        case this.PARAMETERS.STATUS_UPDATE:
          this.doUpdateEditSubNodeCallBack(val)
          break
      }
    },
    handleEditSubNodeDialogCloseMeCancel() {
      this.popSettings.two.visible = false
    },
    // 处理插入回调
    doInsertEditSubNodelCallBack(val) {
      if (val.return_flag) {
        this.popSettings.two.visible = false
        this.getDataList()
        this.$notify({
          title: '添加子菜单-结点处理成功',
          message: val.data.message,
          type: 'success',
          duration: this.settings.duration
        })
      } else {
        this.$notify({
          title: '添加子菜单-结点处理失败',
          message: val.error.message,
          type: 'error',
          duration: this.settings.duration
        })
      }
    },
    // 处理更新回调
    doUpdateEditSubNodeCallBack(val) {
      if (val.return_flag) {
        this.popSettings.two.visible = false

        // 设置到table中绑定的json数据源
        this.dataJson.listData.splice(this.dataJson.rowIndex, 1, val.data.data)
        // 设置到currentjson中
        this.dataJson.currentJson = Object.assign({}, val.data.data)
        this.$notify({
          title: '更新处理成功',
          message: val.data.message,
          type: 'success',
          duration: this.settings.duration
        })
      } else {
        this.$notify({
          title: '更新处理失败',
          message: val.error.message,
          type: 'error',
          duration: this.settings.duration
        })
      }
    }
    // -----------------添加子菜单-结点 end------------------
  }
}
</script>

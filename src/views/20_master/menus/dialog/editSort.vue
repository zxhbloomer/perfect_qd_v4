<template>
  <div>
    <el-dialog
      v-if="listenVisible"
      v-el-drag-dialog
      v-loading="settings.loading"
      element-loading-text="拼命加载中，请稍后..."
      element-loading-background="rgba(255, 255, 255, 0.7)"
      :title="settings.textMap[settings.dialogStatus]"
      :visible="visible"
      :close-on-click-modal="PARAMETERS.DIALOG_CLOSE_BY_CLICK"
      :close-on-press-escape="PARAMETERS.DIALOG_CLOSE_BY_ESC"
      :show-close="PARAMETERS.DIALOG_SHOW_CLOSE"
      :append-to-body="true"
      :modal-append-to-body="true"
      width="900px"
      destroy-on-close
    >
      <div :style="{height: height + 'px'}" style="overflow-y:auto;overflow-x:auto;" class="mytree">
        <el-tree
          ref="treeObject"
          :data="dataJson.treeData"
          :props="dataJson.defaultProps"
          :filter-node-method="filterNode"
          :expand-on-click-node="false"
          :indent="0"
          highlight-current
          node-key="id"
          default-expand-all
          class="tree"
          draggable
          :allow-drop="allowDrop"
          :allow-drag="allowDrag"
          @node-drag-start="handleDragStart"
          @node-drag-enter="handleDragEnter"
          @node-drag-leave="handleDragLeave"
          @node-drag-over="handleDragOver"
          @node-drag-end="handleDragEnd"
          @node-drop="handleDrop"
          @current-change="handleCurrentChange"
        >
          <span slot-scope="{ node, data }" class="custom-tree-node">
            <span>
              <svg-icon v-if="data.type === '10'" icon-class="perfect-icon-tenant" class="el-icon--right" />
              <svg-icon v-else-if="data.type === '20'" icon-class="perfect-icon-group" class="el-icon--right" />
              <svg-icon v-else-if="data.type === '30'" icon-class="perfect-icon-company" class="el-icon--right" />
              <svg-icon v-else-if="data.type === '40'" icon-class="perfect-icon-dept" class="el-icon--right" />
              <svg-icon v-else-if="data.type === '50'" icon-class="perfect-icon-position" class="el-icon--right" />
              {{ data.simple_name }}
            </span>
            <span>[{{ data.type_text }}]</span>
          </span>
        </el-tree>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-divider />
        <el-button plain :disabled="settings.loading" @click="handleCancel()">关闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<style scoped>
  .floatRight {
    float: right;
  }
  .floatLeft {
    float: left;
  }
</style>

<style scoped>
.treeStyle {
  overflow: auto;
  border: 1px solid #ebeef5;
}
.filterInput {
  margin-bottom: 10px;
}

.leaf {
    width: 20px;
    background: #ddd
}

.folder {
    width: 20px;
    background: #888
}

.custom-tree-container {
    display: -ms-flexbox;
    display: flex;
    margin: -24px
}

.block {
    -ms-flex: 1;
    flex: 1;
    padding: 8px 24px 24px
}

.block>p {
    text-align: center;
    margin: 0;
    line-height: 4
}

.block:first-child {
    border-right: 1px solid #eff2f6
}

.custom-tree-node {
    -ms-flex: 1;
    flex: 1;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-align: center;
    align-items: center;
    -ms-flex-pack: justify;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px
}
.el-tree-node:focus > .el-tree-node__content {
  background-color: transparent !important;
}
</style>

<style lang="scss" scoped>
.mytree ::v-deep{
    .el-tree > .el-tree-node:after {
    border-top: none;
  }

  // .el-tree-node__content {
  //   margin-left: -20px;
  // }

  .el-tree-node {
    position: relative;
    padding-left: 2px;
  }
  //结点有间隙，隐藏掉展开按钮就好了,如果觉得空隙没事可以删掉
  .el-tree-node__expand-icon.is-leaf{
    // display: none;
    color: transparent;
    border-top: 1px solid #4386c6;
    height: 3px;
    top: 12px;
    width: 8px;
    margin-right: 8px;
  }
  .el-tree-node__children {
    padding-left: 20px;
  }

  .el-tree-node :last-child:before {
    height: 38px;
  }

  .el-tree > .el-tree-node:before {
    border-left: none;
  }

  .el-tree > .el-tree-node:after {
    border-top: none;
  }

  .el-tree-node:before {
    content: "";
    left: 2px;
    position: absolute;
    right: auto;
    border-width: 1px;
  }

  .el-tree-node:after {
    content: "";
    left: 2px;
    position: absolute;
    right: auto;
    border-width: 1px;
  }

  .el-tree-node:before {
    border-left: 1px solid #4386c6;
    bottom: 0px;
    height: 100%;
    top: -26px;
    width: 1px;
  }
  .el-icon--right {
    margin-left: 0px;
  }

  // .el-tree-node:after {
  //   border-top: 1px solid #4386c6;
  //   height: 20px;
  //   top: 12px;
  //   width: 8px;
  // }

  .el-tree>.el-tree-node{
    min-width:100%;
    display: inline-block ;
  }

  .el-tree-node__content>.el-tree-node__expand-icon {
    padding: 2px
  }
}

</style>

<style >
  .buttonSearch{
    color: #FFFFFF;
    background-color: #1890ff;
    border-color: #1890ff;
  }
</style>

<script>
import { dragsaveApi } from '@/api/20_master/menus/menu'
import elDragDialog from '@/directive/el-drag-dialog'
import { isNotEmpty } from '@/utils/index.js'
import { getDataByIdApi } from '@/api/20_master/position/position'
import constants_para from '@/common/constants/constants_para'

export default {
  components: { },
  directives: { elDragDialog },
  props: {
    height: {
      type: Number,
      default: 200
    },
    visible: {
      type: Boolean,
      default: false
    },
    id: {
      type: Number,
      default: null
    },
    data: {
      type: Object,
      default: null
    },
    dialogStatus: {
      type: String,
      default: constants_para.STATUS_VIEW
    }
  },
  data() {
    return {
      // 监听器
      watch: {
        unwatch_tempJson: null
      },
      dataJson: {
        // 查询使用的json
        searchForm: {
          type: this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT
        },
        // 下拉选项json
        selectOptions: [],
        filterText: '',
        treeData: [],
        // 单条数据 json
        currentJson: null,
        tempJson: {
          org_type: ''
        },
        tempJsonOriginal: null
      },
      // 页面设置json
      settings: {
        para: this.CONSTANTS.DICT_ORG_SETTING_TYPE,
        filterPara: [],
        listLoading: true,
        // 按钮状态：是否可用
        btnDisabledStatus: {
          disabledInsert: true,
          disabledUpdate: true,
          disabledDelete: true
        },
        defaultProps: {
          children: 'children',
          label: 'label'
        }
      },
      popSettingsData: {
        // 弹出窗口状态名称
        dialogStatus: '',
        textMap: {
          update: '请选择要修改结点的类型',
          insert: '请选择添加下级结点类型'
        },
        dialogFormVisible: false,
        btnDisabledStatus: {
          disabledOK: false
        },
        // 弹出的查询框参数设置
        searchDialogDataOne: {
          // 弹出框显示参数
          visible: false,
          // 点击确定以后返回的值
          selectedDataJson: {}
        },
        // 弹出的查询框参数设置
        searchDialogDataTwo: {
          // 弹出框显示参数
          visible: false,
          // 点击确定以后返回的值
          selectedDataJson: {}
        },
        // 弹出的查询框参数设置
        searchDialogDataThree: {
          // 弹出框显示参数
          visible: false,
          // 点击确定以后返回的值
          selectedDataJson: {}
        },
        // 弹出的查询框参数设置
        searchDialogDataFour: {
          // 弹出框显示参数
          visible: false,
          // 点击确定以后返回的值
          selectedDataJson: {}
        },
        // 弹出的查询框参数设置
        searchDialogDataFive: {
          id: undefined,
          data: null,
          // 弹出框显示参数
          visible: false,
          // 点击确定以后返回的值
          selectedDataJson: {}
        }
      }
    }
  },
  computed: {
  },
  watch: {
    'dataJson.filterText': {
      handler(newVal, oldVal) {
        this.$refs.treeObject.filter(newVal)
      }
    },
    'dataJson.currentJson': {
      handler(newVal, oldVal) {
        if (newVal !== null) {
          // 判断是否是第一个结点：第一个结点是租户，所以不能删除，修改，只能新增
          if (this.dataJson.currentJson.parent_id === null) {
            this.settings.btnDisabledStatus.disabledInsert = false
            this.settings.btnDisabledStatus.disabledUpdate = true
            this.settings.btnDisabledStatus.disabledDelete = true
          } else {
            // 判断是否是岗位结点
            if (this.dataJson.currentJson.type === this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION) {
              this.settings.btnDisabledStatus.disabledInsert = false
              this.settings.btnDisabledStatus.disabledUpdate = false
              this.settings.btnDisabledStatus.disabledDelete = false
            } else {
              this.settings.btnDisabledStatus.disabledInsert = false
              this.settings.btnDisabledStatus.disabledUpdate = false
              this.settings.btnDisabledStatus.disabledDelete = false
            }
          }
        } else {
          this.settings.btnDisabledStatus.disabledInsert = true
          this.settings.btnDisabledStatus.disabledUpdate = true
          this.settings.btnDisabledStatus.disabledDelete = true
        }
      }
    },
    'popSettingsData.dialogFormVisible': {
      handler(newVal, oldVal) {
        if (newVal === true) {
          const arr = []
          // arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF)
          switch (this.dataJson.currentJson.type) {
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF)
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              // arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF)
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF)
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY)
              // arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF)
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT)
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION)
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF:
              arr.push(this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT)
              break
          }
          this.dataJson.tempJson.org_type = ''
          this.settings.filterPara = arr
          // 查询数据库，获取下拉选项
          this.getCorrectTypeByInsertStatus(this.dataJson.currentJson.code, this.dataJson.currentJson.type, arr)
        }
      }
    },
    'settings.listLoading': {
      handler(newVal, oldVal) {
        switch (newVal) {
          case true:
            this.showLoading('正在查询，请稍后...')
            break
          case false:
            this.closeLoading()
            break
        }
      }
    }
  },
  created() {
    // 初始化查询
    this.getDataList()
    this.$on(this.EMITS.EMIT_ORG_LEFT, _data => {
      this.handleRefresh()
    })
  },
  mounted() {
    this.initSearchButton()
    // 和right开始绑定事件
    // 描绘完成
    this.$on(this.EMITS.EMIT_LOADING, _data => { this.settings.listLoading = true })
    this.$on(this.EMITS.EMIT_LOADING_OK, _data => { this.settings.listLoading = false })
  },
  methods: {
    // 选择or重置按钮的初始化
    initSearchButton() {
      this.$nextTick(() => {
        this.$refs.buttonSearch.$el.parentElement.className = ' buttonSearch ' + this.$refs.buttonSearch.$el.parentElement.className
      })
    },
    filterNode(value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    getDataList() {
      // 查询逻辑
      this.settings.listLoading = true
      this.dataJson.treeData = this.data
      this.settings.listLoading = false
    },
    handleCurrentChange(row) {
      this.dataJson.currentJson = Object.assign({}, row) // copy obj
      this.dataJson.tempJsonOriginal = Object.assign({}, row) // copy obj
      this.dataJson.tempJson = Object.assign({}, row) // copy obj
      this.dataJson.currentJson = this.$refs.treeObject.getCurrentNode()
      this.dataJson.currentJson.currentkey = this.$refs.treeObject.getCurrentKey()
    },
    handleButtonSearch() {
      // 查询
      this.getDataList()
    },
    // 查询后处理
    getListAfterProcess() {
      if (Object.keys(this.dataJson.filterText).length !== 0) {
        this.$nextTick(() => {
          this.$refs.treeObject.filter(this.dataJson.filterText)
        })
      }
    },
    // 点击新增子结构按钮
    handleInsert() {
      // 新增
      this.popSettingsData.dialogStatus = this.PARAMETERS.STATUS_INSERT
      this.popSettingsData.dialogFormVisible = true
    },
    // 修改当前结点按钮
    handleUpdate() {
      // 修改
      // this.popSettingsData.dialogStatus = this.PARAMETERS.STATUS_UPDATE
      // this.popSettingsData.dialogFormVisible = true
      switch (this.dataJson.currentJson.type) {
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
          this.popSettingsData.searchDialogDataOne.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
          this.popSettingsData.searchDialogDataTwo.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
          this.popSettingsData.searchDialogDataThree.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION:
          this.popSettingsData.searchDialogDataFour.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF:
          break
      }
    },
    handleRadioDictChange(val) {
      this.dataJson.tempJson.org_type = val
      this.doOk()
    },
    doOk() {
      this.popSettingsData.dialogFormVisible = false
      switch (this.dataJson.tempJson.org_type) {
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
          this.popSettingsData.searchDialogDataOne.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
          this.popSettingsData.searchDialogDataTwo.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
          this.popSettingsData.searchDialogDataThree.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION:
          this.popSettingsData.searchDialogDataFour.visible = true
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF:
          getDataByIdApi({ id: this.dataJson.currentJson.serial_id }).then(response => {
            this.popSettingsData.searchDialogDataFive.id = response.data.id
            this.popSettingsData.searchDialogDataFive.data = response.data
            this.popSettingsData.searchDialogDataFive.visible = true
          }).finally(() => {
          })
          break
      }
    },
    handleDelete() {
      this.$confirm('请注意：即将删除当前选择结点以及【子结点】的数据，而且不能恢复。', '确认信息', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确定删除',
        cancelButtonText: '取消'
      }).then(() => {
        this.doDelete()
      }).catch(action => {
        // 右上角X
        // if (action !== 'close') {
        //   //
        // }
      })
    },
    handleRefresh() {
      // 初始化查询
      this.getDataList()
    },
    handleDragStart(node, ev) {
    },
    handleDragEnter(draggingNode, dropNode, ev) {
    },
    handleDragLeave(draggingNode, dropNode, ev) {
    },
    handleDragOver(draggingNode, dropNode, ev) {
    },
    handleDragEnd(draggingNode, dropNode, dropType, ev) {
    },
    /**
     * 拖拽结束后事件
     * draggingNode:被拖拽结点对应的 Node
     * dropNode:结束拖拽时最后进入的结点
     * dropType:被拖拽结点的放置位置（before、after、inner）
     * ev:event
     */
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 进入结点，作为子结点
      if (dropType === 'inner') {
        // 获取老子的id
        const parent_id = dropNode.data.id
        // 获取儿子
        draggingNode.data.parent_id = parent_id
      }
      if (dropType === 'before') {
        // 获取老子的id
        const parent_id = dropNode.data.parent_id
        // 获取儿子
        draggingNode.data.parent_id = parent_id
      }
      if (dropType === 'after') {
        // 获取老子的id
        const parent_id = dropNode.data.parent_id
        // 获取儿子
        draggingNode.data.parent_id = parent_id
      }
      this.doDragSave()
    },
    doDragSave() {
      this.settings.listLoading = true
      dragsaveApi(this.dataJson.treeData).then((_data) => {
        this.$notify({
          title: '更新处理成功',
          message: _data.message,
          type: 'success',
          duration: this.settings.duration
        })
        // 查询
        this.getDataList()
        this.popSettingsData.dialogFormVisible = false
        this.getDataList()
      }, (_error) => {
        this.$notify({
          title: '更新处理失败',
          message: _error.message,
          type: 'error',
          duration: this.settings.duration
        })
        // this.popSettingsData.dialogFormVisible = false
      }).finally(() => {
        this.settings.listLoading = false
      })
    },
    allowDrop(draggingNode, dropNode, type) {
      // if (type !== 'inner') {
      //   return false
      // }
      // 不得放到根目录之前
      if (!isNotEmpty(dropNode.data.parent_id)) {
        return false
      }
      // 不得放到根目录之后，平级
      if (!isNotEmpty(dropNode.data.parent_id)) {
        return false
      }
      switch (draggingNode.data.type) {
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
          // 集团可嵌套，必须在租户下
          switch (dropNode.data.type) {
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
              return true
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_TENANT:
              return true
          }
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
          // 企业必须在集团下
          switch (dropNode.data.type) {
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
              if (dropNode.data.code.length <= 8 && type === 'prev') {
                return false
              }
              return true
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
              if (type === 'inner') {
                return false
              }
              return true
          }
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
          switch (dropNode.data.type) {
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
              return false
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
              if (type === 'inner') {
                return true
              }
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
              return true
          }
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION:
          switch (dropNode.data.type) {
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_GROUP:
              return false
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_COMPANY:
              return false
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_DEPT:
              if (type === 'inner') {
                return true
              }
              break
            case this.CONSTANTS.DICT_ORG_SETTING_TYPE_POSITION:
              if (type !== 'inner') {
                return true
              }
              break
          }
          break
        case this.CONSTANTS.DICT_ORG_SETTING_TYPE_STAFF:
          break
      }
      return false
    },
    // 允许拖拽的情况
    allowDrag(draggingNode) {
      if (isNotEmpty(draggingNode.data.parent_id)) {
        return true
      } else {
        return false
      }
    },
    // 取消按钮
    handleCancel() {
      this.$emit('closeMeCancel')
    }
  }
}
</script>

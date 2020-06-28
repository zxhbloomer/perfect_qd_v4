<template>
  <div @click="onFieldClick">
    <el-tooltip
      :placement="toolTipPlacement"
      :open-delay="toolTipDelay"
      :content="toolTipContent"
    >
      <div v-popover:popover @keyup.enter="onFieldClick">
        <slot name="edit-cell-content" />
      </div>
    </el-tooltip>
    <el-popover
      ref="popover"
      title="快速编辑"
      width="250"
      popper-class="perfect_popper"
    >
      <el-form
        :inline="true"
        :model="dataJson.form"
        label-position="getLabelPosition()"
      >
        <el-form-item label="">
          <component
            :is="editableComponent"
            v-if="editMode || showInput"
            ref="input"
            v-model="model"
            v-popover:edit_cell_popover
            v-bind="$attrs"
            @focus="onFieldClick"
            @keyup.enter.native="onInputExit"
            v-on="listeners"
          >
            <slot name="edit-component-slot" />
          </component>
        </el-form-item>
        <el-divider />
        <div style="text-align: right; margin: 0">
          <el-button type="text">重置</el-button>
          <el-button type="primary">提交</el-button>
        </div>
      </el-form>
    </el-popover>

  </div>
</template>
<script>
export default {
  name: 'EditableCell',
  inheritAttrs: false,
  props: {
    // value: {
    //   type: String,
    //   default: ''
    // },
    // eslint-disable-next-line vue/require-default-prop
    value: [String, Number],
    toolTipContent: {
      type: String,
      default: '点击我可编辑'
    },
    toolTipDelay: {
      type: Number,
      default: 500
    },
    toolTipPlacement: {
      type: String,
      default: 'top-start'
    },
    showInput: {
      type: Boolean,
      default: false
    },
    editableComponent: {
      type: String,
      default: 'el-input'
    },
    closeEvent: {
      type: String,
      default: 'blur'
    }
  },
  data() {
    return {
      editMode: false,
      dataJson: {
        form: {
          sort: undefined
        }
      }
    }
  },
  computed: {
    model: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    },
    listeners() {
      return {
        [this.closeEvent]: this.onInputExit,
        ...this.$listeners
      }
    }
  },
  methods: {
    onFieldClick() {
      this.editMode = true
      // this.$nextTick(() => {
      //   const inputRef = this.$refs.input
      //   if (inputRef) {
      //     inputRef.focus()
      //   }
      // })
    },
    onInputExit() {
      this.editMode = false
    },
    onInputChange(val) {
      this.$emit('input', val)
    }
  }
}
</script>

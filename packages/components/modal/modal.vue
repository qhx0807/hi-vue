<template>
  <div class="ive-modal" v-show="visible" tabindex="-1">
    <div v-show="backdrop" class="ive-modal-shade" ></div>
    <!-- @drop="drop($event)" @dragover="allowDrop($event)" @dragstart="drag($event)" -->
    <transition name="modal">
      <div key="modal"
        class="ive-modal-warp"
        v-show="visible"
        v-clickoutside="onClickBackdrop"
        :style="modalStyle"
        >
        <div class="ive-modal-head"
          :class="{'center' : center}"
          @mousedown="onMouseDown($event)"
          @mousemove="onMouseMove($event)"
          @mouseup="onMouseUp($event)"
          @mouseout="onMouseOut($event)">
          {{title}}
          <i class="iconfont icon-cross" @click="closeModal"></i>
        </div>
        <div class="ive-modal-body"><slot></slot></div>
        <div class="ive-modal-foot">
          <i-button v-if="!isFooter" @click="cancelHandler" type="text" style="margin-right:5px;">取消</i-button>
          <i-button :loading="loading" v-if="!isFooter" @click="okHandler" type="primary" style="margin-right:5px;">确定</i-button>
          <slot v-if="isFooter" name="footer"></slot>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import {addClass, removeClass} from '../../utils/domClass'
import clickoutside from '../../directives/clickoutside'
export default {
  name: 'Modal',
  props: {
    value: {
      type: Boolean,
      default: false
    },
    title: {
      type: [String, Number],
      default: ''
    },
    backdrop: {
      type: Boolean,
      default: true
    },
    center: {
      type: Boolean,
      default: false
    },
    escClosable: {
      type: Boolean,
      default: false
    },
    backdropClosable: {
      type: Boolean,
      default: false
    },
    width: {
      type: [Number, String],
      default: 520
    },
    loading: {
      type: Boolean,
      default: false
    }
  },
  directives: {
    clickoutside
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  data () {
    return {
      visible: this.value,
      preFixCls: 'ive-modal',
      modalStyle: {
        width: '520px',
        marginLeft: 'auto',
        marginTop: '18vh',
      },
      dragStart: {
        offsetX: 0,
        offsetY: 0
      },
      isMove: false,
      isFooter: true,
    }
  },
  watch: {
    visible (val) {
      this.$emit('change', this.visible)
      if(val){
        addClass(document.getElementsByTagName('body')[0], 'modal-show')
      }else{
        removeClass(document.getElementsByTagName('body')[0], 'modal-show')
      }
    },
    value (val) {
      this.visible = val
    }
  },
  created () {
    this.modalStyle.width = this.width + 'px'
    this.visible = this.value
  },
  mounted () {
    this.isFooter = this.$slots.footer !== undefined
    document.addEventListener('keyup', (event) => {
      if (this.escClosable && this.visible && event.keyCode === 27) {
        this.visible = false
      }else{
        return
      }
    })
  },
  methods: {
    closeModal () {
      this.visible = false
      this.$emit('change', this.visible)
    },
    onClickBackdrop () {
      if (this.backdropClosable) {
        this.visible = false
      }
    },
    onMouseDown (ev) {
      this.isMove = true
      this.dragStart.offsetX = ev.offsetX
      this.dragStart.offsetY = ev.offsetY
      ev.preventDefault()
    },
    onMouseUp (ev) {
      this.isMove = false
      ev.preventDefault()
    },
    onMouseMove(ev){
      if(this.isMove && this.visible){
        this.modalStyle.marginLeft = (ev.clientX - this.dragStart.offsetX) + 'px'
        this.modalStyle.marginTop = (ev.clientY - this.dragStart.offsetY) + 'px'
      }
      ev.preventDefault()
    },
    onMouseOut(ev){
      this.isMove = false
      ev.preventDefault()
    },
    cancelHandler(){
      this.$emit('on-cancel')
    },
    okHandler(){
      this.$emit('on-ok')
    }
  }
}
</script>

<style lang="less" scoped>
.modal-enter-active, .modal-leave-active {
  transition: all .3s;
  opacity: 1;
  transform: scale(1);
}
.modal-enter, .modal-leave-to {
  opacity: 0;
  transform: scale(0.7);
}
</style>


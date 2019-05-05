<template>
  <div class='vtl'>
    <div v-if="model.name !== 'root'">
      <div class="vtl-border vtl-up" :class="{'vtl-active': isDragEnterUp}"
        @drop="dropUp"
        @dragenter="dragEnterUp"
        @dragover='dragOverUp'
        @dragleave="dragLeaveUp"></div>
      <div :id='model.id' :class="treeNodeClass"
        :draggable="!model.dragDisabled"
        @dragstart='dragStart'
        @dragover='dragOver'
        @dragenter='dragEnter'
        @dragleave='dragLeave'
        @drop='drop'
        @dragend='dragEnd'
        @mouseover='mouseOver'
        @mouseout='mouseOut'
        @click.stop='click'>
        <span class="vtl-caret" v-if="model.children && model.children.length > 0">
          <feather :type="caretClass" @click.prevent.stop="toggle"/>
        </span>
        <span v-else class="vtl-offset">
        </span>

        <span v-if="model.isLeaf">
          <slot name="leafNodeIcon">
            <feather type="file" class="vtl-icon"/>
          </slot>
        </span>
        <span v-else>
          <slot name="treeNodeIcon">
            <feather type="folder" class="vtl-icon"/>
          </slot>
        </span>

        <div class="vtl-node-content" v-if="!editable">
          {{model.name}}
        </div>
        <template v-else>
          <div class="centerx default-input">
            <vs-input class="inputx" v-model="model.name"/>
          </div>
        </template>
        <div class="vtl-operation" v-show="isHover">
          <span title="add tree node" @click.stop.prevent="addChild(false)" v-if="!model.isLeaf && !model.addTreeNodeDisabled">
            <slot name="addTreeNode">
              <feather type="folder-plus"/>
            </slot>
          </span>
          <span title="add leaf node" @click.stop.prevent="addChild(true)" v-if="!model.isLeaf && !model.addLeafNodeDisabled">
            <slot name="addLeafNode">
              <feather type="file-plus"/>
            </slot>
          </span>
          <span title="edit" @click.stop.prevent="setEditable" v-if="!model.editNodeDisabled">
            <slot name="editNode">
              <feather type="edit"/>
            </slot>
          </span>
          <span title="delete" @click.stop.prevent="delNode" v-if="!model.delNodeDisabled">
            <slot name="delNode">
              <feather type="trash-2"/>
            </slot>
          </span>
        </div>
      </div>

      <div v-if="model.children && model.children.length > 0 && expanded"
        class="vtl-border vtl-bottom"
        :class="{'vtl-active': isDragEnterBottom}"
        @drop="dropBottom"
        @dragenter="dragEnterBottom"
        @dragover='dragOverBottom'
        @dragleave="dragLeaveBottom"></div>
    </div>

    <div :class="{'vtl-tree-margin': model.name !== 'root'}" v-show="model.name === 'root' || expanded" v-if="isFolder">
      <item v-for="model in model.children"
        :default-tree-node-name="defaultTreeNodeName"
        :default-leaf-node-name="defaultLeafNodeName"
        v-bind:default-expanded="defaultExpanded"
        :model="model"
        :key='model.id'>
          <slot name="addTreeNode" slot="addTreeNode" />
          <slot name="addLeafNode" slot="addLeafNode" />
          <slot name="editNode" slot="editNode" />
          <slot name="delNode" slot="delNode" />
      </item>
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import { Tree, TreeNode } from './Tree.js'
  import { addHandler, removeHandler } from './tools.js'
  import { vsInput } from 'vuesax'
  import VueFeather from 'vue-feather';

  import 'vuesax/dist/vuesax.css'
  Vue.use(vsInput)
  Vue.use(VueFeather);
  let fromComp = null

  export default {
    data: function () {
      return {
        isHover: false,
        editable: false,
        isDragEnterUp: false,
        isDragEnterBottom: false,
        isDragEnterNode: false,
        expanded: this.defaultExpanded
      }
    },
    props: {
      model: {
        type: Object
      },
      defaultLeafNodeName: {
        type: String,
        default: 'New leaf node'
      },
      defaultTreeNodeName: {
        type: String,
        default: 'New tree node'
      },
      defaultExpanded: {
        type: Boolean,
        default: true
      }
    },
    computed: {
      itemIconClass () {
        return this.model.isLeaf ? 'vtl-icon-file' : 'vtl-icon-folder'
      },

      caretClass () {
        return this.expanded ? 'chevron-down' : 'chevron-right'
      },

      isFolder () {
        return this.model.children &&
          this.model.children.length
      },

      treeNodeClass () {
        const {
          model: {
            dragDisabled,
            disabled
          },
          isDragEnterNode
        } = this

        return {
          'vtl-tree-node': true,
          'vtl-active': isDragEnterNode,
          'vtl-drag-disabled': dragDisabled,
          'vtl-disabled': disabled
        }
      }
    },
    mounted () {
      const vm = this
      addHandler(window, 'keyup', function (e) {
        // click enter
        if (e.keyCode === 13 && vm.editable) {
          vm.editable = false
        }
      })
    },
    beforeDestroy () {
      removeHandler(window, 'keyup')
    },
    methods: {
      updateName (e) {
        // console.log(e)
        // var oldName = this.model.name;
        // this.model.changeName(e.target.value)
        // var node = this.getRootNode();
        // node.$emit('change-name', {'id': this.model.id, 'oldName': oldName, 'newName': e.target.value})
      },

      delNode () {
        var node = this.getRootNode()
        node.$emit('delete-node', this.model)
      },

      setEditable () {
        this.editable = true
      },

      setUnEditable () {
        this.editable = false
      },

      toggle() {
        if (this.isFolder) {
          this.expanded = !this.expanded
        }
      },

      mouseOver(e) {
        if (this.model.disabled) return
        this.isHover = true
      },

      mouseOut(e) {
        this.isHover = false
      },

      click() {
        var node = this.getRootNode()
        node.$emit('click', this.model);
      },

      addChild(isLeaf) {
        const name = isLeaf ? this.defaultLeafNodeName : this.defaultTreeNodeName
        this.expanded = true
        var node = new TreeNode({ name, isLeaf })
        this.model.addChildren(node, true)
        var root = this.getRootNode();
        root.$emit('add-node', node)
      },

      dragStart(e) {
        if (!(this.model.dragDisabled || this.model.disabled)) {
          fromComp = this
          // for firefox
          e.dataTransfer.setData("data","data");
          e.dataTransfer.effectAllowed = 'move'
          return true
        }
        return false
      },
      dragEnd(e) {
        fromComp = null
      },
      dragOver(e) {
        e.preventDefault()
        return true
      },
      dragEnter(e) {
        if (!fromComp) return
        if (this.model.isLeaf) return
        this.isDragEnterNode = true
      },
      dragLeave(e) {
        this.isDragEnterNode = false
      },
      drop(e) {
        if (!fromComp) return
        const oldParent = fromComp.model.parent;
        fromComp.model.moveInto(this.model)
        this.isDragEnterNode = false
        var node = this.getRootNode();
        node.$emit('drop', {node: fromComp.model, oldParent: oldParent})
      },

      dragEnterUp () {
        if (!fromComp) return
        this.isDragEnterUp = true
      },
      dragOverUp (e) {
        e.preventDefault()
        return true
      },
      dragLeaveUp () {
        if (!fromComp) return
        this.isDragEnterUp = false
      },
      dropUp () {
        if (!fromComp) return
        const oldParent = fromComp.model.parent;
        fromComp.model.insertBefore(this.model)
        this.isDragEnterUp = false
        var node = this.getRootNode();
        node.$emit('drop-up', {node: fromComp.model, oldParent: oldParent})
      },

      dragEnterBottom () {
        if (!fromComp) return
        this.isDragEnterBottom = true
      },
      dragOverBottom (e) {
        e.preventDefault()
        return true
      },
      dragLeaveBottom () {
        if (!fromComp) return
        this.isDragEnterBottom = false
      },
      dropBottom () {
        if (!fromComp) return
        const oldParent = fromComp.model.parent;
        fromComp.model.insertAfter(this.model)
        this.isDragEnterBottom = false
        var node = this.getRootNode();
        node.$emit('drop-bottom', {node: fromComp.model, oldParent: oldParent})
      },
      getRootNode() {
        var node = this.$parent
        while (node._props.model.name !== 'root') {
          node = node.$parent
        }
        return node;
      }
    },
    beforeCreate () {
      this.$options.components.item = require('./VueTreeList.vue')
    }
  }
</script>

<style lang="less" rel="stylesheet/less">
  @font-face {
    font-family: 'icomoon';
    src:  url('fonts/icomoon.eot?ui1hbx');
    src:  url('fonts/icomoon.eot?ui1hbx#iefix') format('embedded-opentype'),
    url('fonts/icomoon.ttf?ui1hbx') format('truetype'),
    url('fonts/icomoon.woff?ui1hbx') format('woff'),
    url('fonts/icomoon.svg?ui1hbx#icomoon') format('svg');
    font-weight: normal;
    font-style: normal;
  }

  .vtl-icon {
    /* use !important to prevent issues with browser extensions that change fonts */
    font-family: 'icomoon' !important;
    speak: none;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
    padding-right: 6px;

    /* Better Font Rendering =========== */
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    &.vtl-menu-icon {
      margin-right: 4px;
      &:hover {
        color: inherit;
      }
    }
    &:hover {
      color: blue;
    }
  }

  .vtl-icon-file:before {
    content: "\e906";
  }
  .vtl-icon-folder:before {
    content: "\e907";
  }
  .vtl-icon-caret-down:before {
    content: "\e901";
  }
  .vtl-icon-caret-right:before {
    content: "\e900";
  }
  .vtl-icon-edit:before {
    content: "\e902";
  }
  .vtl-icon-folder-plus-e:before {
    content: "\e903";
  }
  .vtl-icon-plus:before {
    content: "\e904";
  }
  .vtl-icon-trash:before {
    content: "\e905";
  }


  .vtl-border {
    height: 5px;
    &.vtl-up {
      margin-top: -5px;
      background-color: transparent;
    }
    &.vtl-bottom {
      background-color: transparent;
    }
    &.vtl-active {
      border-bottom: 3px dashed blue;
      /*background-color: blue;*/
    }
  }

  .vtl-tree-node {
    display: flex;
    align-items: center;
    padding: 5px 0 5px 1rem;
    .vtl-input {
      border: none;
      max-width: 150px;
      border-bottom: 1px solid blue;
    }
    &:hover {
      background-color: #f0f0f0;
    }
    &.vtl-active {
      outline: 2px dashed pink;
    }
    .vtl-caret {
      margin-left: -1rem;
    }
    .vtl-offset {
      padding-left: 8px;
    }
    .vtl-operation {
      margin-left: 2rem;
      letter-spacing: 1px;
    }
  }


  .vtl-item {
    cursor: pointer;
  }
  .vtl-tree-margin {
    margin-left: 2em;
  }
</style>

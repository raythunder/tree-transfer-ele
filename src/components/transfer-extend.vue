<template>
  <div class="transfer" id="transfer" :style="{width,height}">
    <template v-if="mode == 'transfer'">
      <!-- 左侧穿梭框 原料框 -->
      <div class="transfer-left">
        <h3 class="transfer-title">
          <el-checkbox :indeterminate="from_is_indeterminate" v-model="from_check_all" @change='fromAllBoxChange'></el-checkbox>
          <span>{{fromTitle}}</span>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <!-- <slot name="from"></slot> -->
          <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterFrom" size="small" class="filter-tree">
          </el-input>
          <el-tree ref='from-tree' :data="self_from_data" show-checkbox :node-key="node_key" @check='fromTreeChecked' :default-expanded-keys="from_expanded_keys" :props="defaultProps" :filter-node-method="filterNodeFrom" :default-expand-all="openAll" :render-content='renderContent' :default-checked-keys="defaultCheckedKeys">
          </el-tree>
        </div>
      </div>
      <!-- 穿梭区 按钮框 -->
      <div class="transfer-center">
        <template v-if='button_text'>
          <p class="transfer-center-item">
            <el-button type="primary" @click="addTo" :disabled="from_disabled">
              {{fromButton || '添加'}}
              <i class="el-icon-arrow-right"></i>
            </el-button>
          </p>
          <p class="transfer-center-item">
            <el-button type="primary" @click='removeTo' :disabled="to_disabled" icon="el-icon-arrow-left">{{toButton || '移除'}}</el-button>
          </p>
        </template>
        <template v-else>
          <p class="transfer-center-item">
            <el-button type="primary" @click="addToAims" icon="el-icon-arrow-right" circle :disabled="from_disabled"></el-button>
          </p>
          <p class="transfer-center-item">
            <el-button type="primary" @click='removeToSource' :disabled="to_disabled" icon="el-icon-arrow-left" circle></el-button>
          </p>
        </template>
      </div>
      <!-- 右侧穿梭框 目标框 -->
      <div class="transfer-right">
        <h3 class="transfer-title">
          <el-checkbox :indeterminate="to_is_indeterminate" v-model="to_check_all" @change="toAllBoxChange"></el-checkbox>
          <span>{{toTitle}}</span>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <!-- <slot name='to'></slot> -->
          <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterTo" size="small" class="filter-tree">
          </el-input>
          <el-tree slot='to' ref='to-tree' :data="self_to_data" show-checkbox :node-key="node_key" @check='toTreeChecked' :default-expanded-keys="to_expanded_keys" :props="defaultProps" :filter-node-method="filterNodeTo" :default-expand-all="openAll" :render-content='renderContent'>
          </el-tree>
        </div>
      </div>
    </template>
    <template v-else-if="mode == 'addressList'">
      <!-- 左侧穿梭框 原料框 -->
      <div class="transfer-left">
        <h3 class="transfer-title">
          <el-checkbox :indeterminate="from_is_indeterminate" v-model="from_check_all" @change='fromAllBoxChange'></el-checkbox>
          <span>{{fromTitle}}</span>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <!-- <slot name="from"></slot> -->
          <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterFrom" size="small" class="filter-tree">
          </el-input>
          <el-tree ref='from-tree' :data="self_from_data" show-checkbox :node-key="node_key" @check='fromTreeChecked' :default-expanded-keys="from_expanded_keys" :props="defaultProps" :filter-node-method="filterNodeFrom" :default-expand-all="openAll" :render-content='renderContent'>
          </el-tree>
        </div>
      </div>
      <!-- 穿梭区 按钮框 -->
      <div class="transfer-center address-list-center">
        <p class="transfer-center-item" v-show='!move_up'>
          <el-button type="primary" @click="addressListTransfer(0)" icon="el-icon-arrow-right" circle :disabled="from_disabled"></el-button>
        </p>
        <p class="transfer-center-item">
          <el-button type="primary" @click='addressListTransfer(1)' :disabled="from_disabled" icon="el-icon-arrow-right" circle></el-button>
        </p>
        <p class="transfer-center-item" v-show='move_up'>
          <el-button type="primary" @click='addressListTransfer(2)' :disabled="from_disabled" icon="el-icon-arrow-right" circle></el-button>
        </p>
      </div>
      <div class="transfer-right">
        <div class="transfer-right-item" :class="{'transfer-right-small':move_up}">
          <h3 class="transfer-title">
            <span>{{toTitle}}</span>
            <span class="u-clear" @click="clearList(0,'all')" v-if="!move_up">清空</span>
            <img class="move_up_img move_down_img" v-else src="./shang.png" alt="" @click="moveUp('down')">
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main" v-if='!move_up'>
            <!-- <slot name='to'></slot> -->
            <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterListFirst" size="small" class="filter-tree">
            </el-input>
            <ul class="address-list-ul">
              <li class="address-list-li" v-for="item of addressee" :key="item[node_key]">
                <label>{{item[defaultProps.label]}}{{item.Email}}</label>
                <i class="address-list-del" @click="clearList(0,item[node_key])">x</i>
              </li>
            </ul>
          </div>
        </div>
        <div class="transfer-right-item">
          <h3 class="transfer-title">
            <span>{{toTitleSecond || '抄送人'}}</span>
            <span class="u-clear" @click="clearList(1,'all')">清空</span>
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main">
            <!-- <slot name='to'></slot> -->
            <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterListSecond" size="small" class="filter-tree">
            </el-input>
            <ul class="address-list-ul">
              <li class="address-list-li" v-for="item of Cc" :key="item[node_key]">
                <label>{{item[defaultProps.label]}}{{item.Email}}</label>
                <i class="address-list-del" @click="clearList(1,item[node_key])">x</i>
              </li>
            </ul>
          </div>
        </div>
        <div class="transfer-right-item" :class="{'transfer-right-small':!move_up}">
          <h3 class="transfer-title">
            <span>{{toTitleThird || '密送人'}}</span>
            <span class="u-clear" @click="clearList(2,'all')" v-if="move_up">清空</span>
            <img class="move_up_img" v-else src="./shang.png" alt="" @click="moveUp('up')">
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main" v-if='move_up'>
            <!-- <slot name='to'></slot> -->
            <el-input v-if="filter" placeholder="输入关键字进行过滤" v-model="filterListThird" size="small" class="filter-tree">
            </el-input>
            <ul class="address-list-ul">
              <li class="address-list-li" v-for="item of secret_receiver" :key="item[node_key]">
                <label>{{item[defaultProps.label]}}{{item.Email}}</label>
                <i class="address-list-del" @click="clearList(2,item[node_key])">x</i>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import Emitter from '@/element-ui/src/mixins/emitter'
import Locale from '@/element-ui/src/mixins/locale'
import ElInput from '@/element-ui/packages/input'
import ElButton from '@/element-ui/packages/button'
import ElTree from '@/element-ui/packages/tree'
import ElCheckbox from '@/element-ui/packages/checkbox'

export default {
  name: 'el-tree-transfer',
  components: {
    ElInput,
    ElButton,
    ElTree,
    ElCheckbox
  },
  mixins: [Emitter, Locale],
  data () {
    return {
      from_is_indeterminate: false, // 源数据是否半选
      from_check_all: false, // 源数据是否全选
      to_is_indeterminate: false, // 目标数据是否半选
      to_check_all: false, // 目标数据是否全选
      from_expanded_keys: [], // 源数据展开节点
      to_expanded_keys: [], // 目标数据展开节点
      // self_from_data: [...this.from_data], // 左侧数据
      // self_to_data: [...this.to_data], // 右侧数据
      self_from: [], // 子组件左侧数据
      self_to: [], // 子组件右侧数据
      from_disabled: true, // 添加按钮是否禁用
      to_disabled: true, // 移除按钮是否禁用
      from_check_keys: [], // 源数据选中key数组 以此属性关联穿梭按钮，总全选、半选状态
      to_check_keys: [], // 目标数据选中key数组 以此属性关联穿梭按钮，总全选、半选状态
      filterFrom: '', // 源数据筛选
      filterTo: '', // 目标数据筛选
      filterListFirst: '', // 通讯录模式 右1筛选
      filterListSecond: '', // 通讯录模式 右2筛选
      filterListThird: '', // 通讯录模式 右3筛选
      archiveFirst: [], // 存档右侧筛选前数据
      archiveSecond: [], // 存档右侧筛选前数据
      archiveThird: [], // 存档右侧筛选前数据
      addressee: [], // 收件人列表
      Cc: [], // 抄送人列表
      secret_receiver: [], // 密送人列表
      move_up: false // 通讯录模式 切换右侧
    }
  },
  props: {
    // 宽度
    width: {
      type: String,
      default: '100%'
    },
    // 高度
    height: {
      type: String,
      default: '320px'
    },
    // 标题
    title: {
      type: Array,
      default: () => ['源列表', '目标列表']
    },
    // 穿梭按钮名字
    button_text: Array,
    // 源数据
    from_data: {
      type: Array,
      default: () => []
    },
    // 选中数据
    to_data: {
      type: Array,
      default: () => []
    },
    // el-tree 配置项
    defaultProps: {
      type: Object,
      default: () => {
        return { label: 'label', children: 'children' }
      }
    },
    // el-tree node-key 必须唯一
    node_key: {
      type: String,
      default: 'id'
    },
    // 自定义 pid参数名
    pid: {
      type: String,
      default: 'pid'
    },
    // 是否只返回叶子节点
    leafOnly: {
      type: Boolean,
      default: false
    },
    // 是否启用筛选
    filter: {
      type: Boolean,
      default: false
    },
    // 是否展开所有节点
    openAll: {
      type: Boolean,
      default: false
    },
    // 自定义树节点
    renderContent: Function,
    // 穿梭框模式
    mode: {
      type: String,
      default: 'transfer'
    },
    // 穿梭后是否展开节点
    transferOpenNode: {
      type: Boolean,
      default: true
    },
    // 源数据 默认选中节点
    defaultCheckedKeys: {
      type: Array,
      default: () => []
    }
  },
  created () {
    this.from_check_keys = this.defaultCheckedKeys
  },
  methods: {
    // 添加按钮
    addToAims () {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs['from-tree'].getCheckedKeys(this.leafOnly)
      // 获取半选通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let harfKeys = this.$refs['from-tree'].getHalfCheckedKeys()
      // 选中节点数据
      let arrayCheckedNodes = this.$refs['from-tree'].getCheckedNodes(
        this.leafOnly
      )
      // 获取选中通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let nodes = JSON.parse(JSON.stringify(arrayCheckedNodes))
      // 半选中节点数据
      let arrayHalfCheckedNodes = this.$refs['from-tree'].getHalfCheckedNodes()
      // 获取半选通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let halfNodes = JSON.parse(JSON.stringify(arrayHalfCheckedNodes))
      // 自定义参数读取设置
      let children__ = this.defaultProps.children || 'children'
      let pid__ = this.pid || 'pid'
      let id__ = this['node_key'] || 'id'
      /*
       * 先整合目标树没有父节点的叶子节点选中，需要整理出来此叶子节点的父节点直到根节点路径 - 此时所有骨架节点已有
       * 再将所有末端叶子节点根据pid直接推入目标树即可
       * 声明新盒子将所有半选节点的子节点清除 - 只保留骨架 因为排序是先父后子 因此不存在子元素处理好插入时父元素还没处理的情况
       * 下面一二步是为了搭建出来目标树没有根节点躯干节点时的叶子选中，给此叶子搭建出根节点和躯干节点
       */
      // let不存在状态提升 因此在函数调用之前赋值 并递归为以为数组！
      let self_to_data = JSON.stringify(this.self_to_data)
      // 第一步
      let skeletonHalfCheckedNodes = JSON.parse(
        JSON.stringify(arrayHalfCheckedNodes)
      ) // 深拷贝数据 - 半选节点
      // 筛选目标树不存在的骨架节点 - 半选内的节点
      let newSkeletonHalfCheckedNodes = []
      skeletonHalfCheckedNodes.forEach(item => {
        if (!inquireIsExist(item)) {
          newSkeletonHalfCheckedNodes.push(item)
        }
      })
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 半选节点
      newSkeletonHalfCheckedNodes.forEach(item => {
        item[children__] = []
        if (item[pid__] == 0) {
          this.$refs['to-tree'].append(item)
        } else {
          this.$refs['to-tree'].append(item, item[pid__])
        }
      })
      // 第二步
      let cloneSkeletonCheckedNodes = JSON.parse(
        JSON.stringify(arrayCheckedNodes)
      ) // 深拷贝数据 -选中节点
      // 筛选目标树不存在的骨架节点 - 全选内的节点
      let newSkeletonCheckedNodes = []
      cloneSkeletonCheckedNodes.forEach(item => {
        if (!inquireIsExist(item)) {
          newSkeletonCheckedNodes.push(item)
        }
      })
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 全选节点
      newSkeletonCheckedNodes.forEach(item => {
        if (item[children__] && item[children__].length > 0) {
          item[children__] = []
          this.$refs['to-tree'].append(item, item[pid__])
        }
      })
      // 第三步 处理末端叶子元素 - 声明新盒子筛选出所有末端叶子节点
      let leafCheckedNodes = arrayCheckedNodes.filter(
        item => !item[children__] || item[children__].length == 0
      )
      // 末端叶子直接插入目标树
      leafCheckedNodes.forEach(item => {
        this.$refs['to-tree'].append(item, item[pid__])
      })
      // 递归查询data内是否存在item函数
      function inquireIsExist (item, strData = self_to_data) {
        // 将树形数据格式化成一维字符串 然后通过匹配来判断是否已存在
        let strItem =
          typeof item[id__] === 'number'
            ? `"${id__}":${item[id__]}`
            : `"${id__}":"${item[id__]}"`
        let reg = RegExp(strItem)
        let existed = reg.test(strData)
        /*  for (let i of data) {
          if (item.id == i.id) {
            existed = true;
          } else if (i.children.length > 0) {
            inquireIsExist(item, i.children);
          }
        } */
        return existed
      }
      /*  // 选中项 从源数据移除
      arrayCheckedNodes.forEach(item => {
        recursive(item, this.self_from_data);
      });
      // 源树递归查询
      function recursive(obj, data) {
        for (let i of data) {
          if (i.id == obj.id) {
            console.log("f", data);
            let arr = data.filter(b => b.id != obj.id);
            data = arr;
            console.log("t", data);
            continue;
          } else {
            recursive(obj, i.children);
          }
        }
      } */
      // 左侧删掉选中数据
      arrayCheckedNodes.forEach(item => {
        this.$refs['from-tree'].remove(item)
      })
      // 处理完毕按钮恢复禁用状态
      this.from_check_keys = []
      // 目标数据节点展开
      if (this.transferOpenNode) {
        this.to_expanded_keys = keys
      }
      // 传递信息给父组件
      this.$emit('addBtn', this.self_from_data, this.self_to_data, {
        keys,
        nodes,
        harfKeys,
        halfNodes
      })
    },
    // 移除按钮
    removeToSource () {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs['to-tree'].getCheckedKeys(this.leafOnly)
      // 获取半选通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let harfKeys = this.$refs['to-tree'].getHalfCheckedKeys()
      // 获取选中通过穿梭框的nodes 选中节点数据
      let arrayCheckedNodes = this.$refs['to-tree'].getCheckedNodes(
        this.leafOnly
      )
      // 获取选中通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let nodes = JSON.parse(JSON.stringify(arrayCheckedNodes))
      // 半选中节点数据
      let arrayHalfCheckedNodes = this.$refs['to-tree'].getHalfCheckedNodes()
      // 获取半选通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let halfNodes = JSON.parse(JSON.stringify(arrayHalfCheckedNodes))
      // 自定义参数读取设置
      let children__ = this.defaultProps.children || 'children'
      let pid__ = this.pid || 'pid'
      let id__ = this['node_key'] || 'id'
      /*
       * 先整合目标树没有父节点的叶子节点选中，需要整理出来此叶子节点的父节点直到根节点路径 - 此时所有骨架节点已有
       * 再将所有末端叶子节点根据pid直接推入目标树即可
       * 声明新盒子将所有半选节点的子节点清除 - 只保留骨架 因为排序是先父后子 因此不存在子元素处理好插入时父元素还没处理的情况
       * 下面一二步是为了搭建出来目标树没有根节点躯干节点时的叶子选中，给此叶子搭建出根节点和躯干节点
       */
      // let不存在状态提升 因此在函数调用之前赋值 并递归为以为数组！
      let self_from_data = JSON.stringify(this.self_from_data)
      // 第一步
      let skeletonHalfCheckedNodes = JSON.parse(
        JSON.stringify(arrayHalfCheckedNodes)
      ) // 深拷贝数据 - 半选节点
      // 筛选目标树不存在的骨架节点 - 半选内的节点
      let newSkeletonHalfCheckedNodes = []
      skeletonHalfCheckedNodes.forEach(item => {
        if (!inquireIsExist(item)) {
          newSkeletonHalfCheckedNodes.push(item)
        }
      })
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 半选节点
      newSkeletonHalfCheckedNodes.forEach(item => {
        item[children__] = []
        if (item[pid__] == 0) {
          this.$refs['from-tree'].append(item)
        } else {
          this.$refs['from-tree'].append(item, item[pid__])
        }
      })
      // 第二步
      let cloneSkeletonCheckedNodes = JSON.parse(
        JSON.stringify(arrayCheckedNodes)
      ) // 深拷贝数据 -选中节点
      // 筛选目标树不存在的骨架节点 - 全选内的节点
      let newSkeletonCheckedNodes = []
      cloneSkeletonCheckedNodes.forEach(item => {
        if (!inquireIsExist(item)) {
          newSkeletonCheckedNodes.push(item)
        }
      })
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 全选节点
      newSkeletonCheckedNodes.forEach(item => {
        if (item[children__] && item[children__].length > 0) {
          item[children__] = []
          this.$refs['from-tree'].append(item, item[pid__])
        }
      })
      // 第三步 处理末端叶子元素 - 声明新盒子筛选出所有末端叶子节点
      let leafCheckedNodes = arrayCheckedNodes.filter(
        item => !item[children__] || item[children__].length == 0
      )
      // 末端叶子直接插入目标树
      leafCheckedNodes.forEach(item => {
        this.$refs['from-tree'].append(item, item[pid__])
      })
      // 递归查询data内是否存在item函数
      function inquireIsExist (item, strData = self_from_data) {
        // 将树形数据格式化成一维字符串 然后通过匹配来判断是否已存在
        let strItem =
          typeof item[id__] === 'number'
            ? `"${id__}":${item[id__]}`
            : `"${id__}":"${item[id__]}"`
        let reg = RegExp(strItem)
        let existed = reg.test(strData)
        /*  for (let i of data) {
          if (item.id == i.id) {
            existed = true;
          } else if (i.children.length > 0) {
            inquireIsExist(item, i.children);
          }
        } */
        return existed
      }
      /*  // 选中项 从源数据移除
      arrayCheckedNodes.forEach(item => {
        recursive(item, this.self_from_data);
      });
      // 源树递归查询
      function recursive(obj, data) {
        for (let i of data) {
          if (i.id == obj.id) {
            console.log("f", data);
            let arr = data.filter(b => b.id != obj.id);
            data = arr;
            console.log("t", data);
            continue;
          } else {
            recursive(obj, i.children);
          }
        }
      } */
      // 右侧删掉选中数据
      arrayCheckedNodes.forEach(item => {
        this.$refs['to-tree'].remove(item)
      })
      // 处理完毕按钮恢复禁用状态
      this.to_check_keys = []
      // 目标数据节点展开
      if (this.transferOpenNode) {
        this.from_expanded_keys = keys
      }
      // 传递信息给父组件
      this.$emit('removeBtn', this.self_from_data, this.self_to_data, {
        keys,
        nodes,
        harfKeys,
        halfNodes
      })
    },
    // 源树选中事件 - 是否禁用穿梭按钮
    fromTreeChecked (nodeObj, treeObj) {
      this.from_check_keys = treeObj.checkedNodes
    },
    // 目标树选中事件 - 是否禁用穿梭按钮
    toTreeChecked (nodeObj, treeObj) {
      this.to_check_keys = treeObj.checkedNodes
    },
    // 源数据 总全选checkbox
    fromAllBoxChange (val) {
      if (this.self_from_data.length == 0) {
        return
      }
      if (val) {
        // let array = [...this.from_data];
        this.from_check_keys = this.self_from_data
        this.$refs['from-tree'].setCheckedNodes(this.self_from_data)
      } else {
        this.$refs['from-tree'].setCheckedNodes([])
        this.from_check_keys = []
      }
    },
    // 目标数据 总全选checkbox
    toAllBoxChange (val) {
      if (this.self_to_data.length == 0) {
        return
      }
      if (val) {
        this.to_check_keys = this.self_to_data
        this.$refs['to-tree'].setCheckedNodes(this.self_to_data)
      } else {
        this.$refs['to-tree'].setCheckedNodes([])
        this.to_check_keys = []
      }
    },
    // 源数据 筛选
    filterNodeFrom (value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    // 目标数据筛选
    filterNodeTo (value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    // 通讯录模式 穿梭操作
    addressListTransfer (type) {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs['from-tree'].getCheckedKeys(true)
      // 选中节点数据
      let arrayCheckedNodes = this.$refs['from-tree'].getCheckedNodes(true)
      // 去重筛选
      let arrayDeWeighting = []
      switch (type) {
        case 0:
          arrayDeWeighting = arrayCheckedNodes.filter(item => {
            if (
              !this.addressee.some(
                ite => ite[this.node_key] == item[this.node_key]
              )
            ) {
              return item
            }
          })
          this.addressee = [...this.addressee, ...arrayDeWeighting]
          break
        case 1:
          arrayDeWeighting = arrayCheckedNodes.filter(item => {
            if (
              !this.Cc.some(ite => ite[this.node_key] == item[this.node_key])
            ) {
              return item
            }
          })
          this.Cc = [...this.Cc, ...arrayDeWeighting]
          break
        case 2:
          arrayDeWeighting = arrayCheckedNodes.filter(item => {
            if (
              !this.secret_receiver.some(
                ite => ite[this.node_key] == item[this.node_key]
              )
            ) {
              return item
            }
          })
          this.secret_receiver = [...this.secret_receiver, ...arrayDeWeighting]
          break
      }
      // 处理完毕取消选中
      this.$refs['from-tree'].setCheckedKeys([])
      // 处理完毕按钮恢复禁用状态
      this.from_check_keys = []
      // 传递信息给父组件
      this.$emit('addBtn', this.addressee, this.Cc, this.secret_receiver)
    },
    // 清理 通讯录选中 数据
    clearList (type, id) {
      switch (type) {
        case 0:
          this.addressee =
            id == 'all'
              ? []
              : this.addressee.filter(item => item[this.node_key] != id)
          break
        case 1:
          this.Cc =
            id == 'all'
              ? []
              : this.Cc.filter(item => item[this.node_key] != id)
          break
        case 2:
          this.secret_receiver =
            id == 'all'
              ? []
              : this.secret_receiver.filter(item => item[this.node_key] != id)
          break
      }
      // 传递信息给父组件
      this.$emit('removeBtn', this.addressee, this.Cc, this.secret_receiver)
    },
    // 右侧 通讯录 上下自动
    moveUp (type) {
      if (type == 'up') {
        this.move_up = true
      } else {
        this.move_up = false
      }
    }
  },
  computed: {
    // 左侧数据
    self_from_data () {
      let from_array = [...this.from_data, ...this.self_from]
      from_array.forEach(item => {
        item[this.pid] = 0
      })
      return from_array
    },
    // 右侧数据
    self_to_data () {
      let to_array = [...this.to_data, ...this.self_to]
      to_array.forEach(item => {
        item[this.pid] = 0
      })
      return to_array
    },
    // 左侧菜单名
    fromTitle () {
      let [text] = this.title
      return text
    },
    // 右侧菜单名
    toTitle () {
      let [, text] = this.title
      return text
    },
    // 右侧菜单名2
    toTitleSecond () {
      let [, , text] = this.title
      return text
    },
    // 右侧菜单名3
    toTitleThird () {
      let [, , , text] = this.title
      return text
    },
    // 上部按钮名
    fromButton () {
      if (this.button_text == undefined) {
        return
      }
      let [text] = this.button_text
      return text
    },
    // 下部按钮名
    toButton () {
      if (this.button_text == undefined) {
        return
      }
      let [, text] = this.button_text
      return text
    }
  },
  watch: {
    // 左侧 状态监测
    from_check_keys (val) {
      if (val.length > 0) {
        // 穿梭按钮是否禁用
        this.from_disabled = false
        // 总半选是否开启
        this.from_is_indeterminate = true
        // 总全选是否开启 - 根据选中节点中为根节点的数量是否和源数据长度相等
        let allCheck = val.filter(item => item[this.pid] == 0)
        if (allCheck.length == this.self_from_data.length) {
          // 关闭半选 开启全选
          this.from_is_indeterminate = false
          this.from_check_all = true
        } else {
          this.from_is_indeterminate = true
          this.from_check_all = false
        }
      } else {
        this.from_disabled = true
        this.from_is_indeterminate = false
        this.from_check_all = false
      }
    },
    // 右侧 状态监测
    to_check_keys (val) {
      if (val.length > 0) {
        // 穿梭按钮是否禁用
        this.to_disabled = false
        // 总半选是否开启
        this.to_is_indeterminate = true
        // 总全选是否开启 - 根据选中节点中为根节点的数量是否和源数据长度相等
        let allCheck = val.filter(item => item[this.pid] == 0)
        if (allCheck.length == this.self_to_data.length) {
          // 关闭半选 开启全选
          this.to_is_indeterminate = false
          this.to_check_all = true
        } else {
          this.to_is_indeterminate = true
          this.to_check_all = false
        }
      } else {
        this.to_disabled = true
        this.to_is_indeterminate = false
        this.to_check_all = false
      }
    },
    // 左侧 数据筛选
    filterFrom (val) {
      this.$refs['from-tree'].filter(val)
    },
    // 右侧 数据筛选
    filterTo (val) {
      this.$refs['to-tree'].filter(val)
    },
    // 通讯录模式 右1筛选
    filterListFirst (newval, oldval) {
      if (oldval == '') {
        this.archiveFirst = this.addressee
      }
      if (newval == '') {
        this.addressee = this.archiveFirst
      }
      let reg = RegExp(newval)
      this.addressee = this.addressee.filter(item => reg.test(item.label))
    },
    // 通讯录模式 右2筛选
    filterListSecond (newval, oldval) {
      if (oldval == '') {
        this.archiveSecond = this.Cc
      }
      if (newval == '') {
        this.Cc = this.archiveSecond
      }
      let reg = RegExp(newval)
      this.Cc = this.Cc.filter(item => reg.test(item.label))
    },
    // 通讯录模式 右3筛选
    filterListThird (newval, oldval) {
      if (oldval == '') {
        this.archiveThird = this.secret_receiver
      }
      if (newval == '') {
        this.secret_receiver = this.archiveThird
      }
      let reg = RegExp(newval)
      this.secret_receiver = this.secret_receiver.filter(item =>
        reg.test(item.label)
      )
    }
  }
}
</script>

<style scoped>
@import "./clear.css";
.el-tree {
  min-width: 100%;
  display: inline-block !important;
}
.transfer {
  position: relative;
  overflow: hidden;
}
.transfer-left {
  position: absolute;
  top: 0;
  left: 0;
}
.transfer-right {
  position: absolute;
  top: 0;
  right: 0;
}
.transfer-right-item {
  height: calc((100% - 41px) / 2);
}
.transfer-right-small {
  height: 41px;
}
.transfer-main {
  padding: 10px;
  height: calc(100% - 41px);
  box-sizing: border-box;
  overflow: auto;
}
.transfer-left,
.transfer-right {
  border: 1px solid #ebeef5;
  width: 40%;
  height: 100%;
  box-sizing: border-box;
  border-radius: 5px;
  vertical-align: middle;
}
.transfer-center {
  position: absolute;
  top: 50%;
  left: 40%;
  width: 20%;
  transform: translateY(-50%);
  text-align: center;
}
.transfer-center-item {
  padding: 10px;
  overflow: hidden;
}
.address-list-center {
  height: 100%;
}
.address-list-center > .transfer-center-item {
  height: 50%;
  padding: 70px 10px 0;
  box-sizing: border-box;
  overflow: hidden;
}
.transfer-title {
  border-bottom: 1px solid #ebeef5;
  padding: 0 15px;
  height: 40px;
  line-height: 40px;
  color: #333;
  font-size: 16px;
  background-color: #f5f7fa;
}
.transfer-title .el-checkbox {
  margin-right: 10px;
}
.filter-tree {
  margin-bottom: 10px;
}
.address-list-ul {
  padding-bottom: 20px;
}
.address-list-li {
  position: relative;
  padding: 4px 24px 4px 4px;
  border-radius: 3px;
  overflow: hidden; /*超出部分隐藏*/
  white-space: nowrap; /*不换行*/
  text-overflow: ellipsis; /*超出部分文字以...显示*/
}
.address-list-li:hover {
  background-color: #f5f7fa;
}
.address-list-li:hover .address-list-del {
  display: block;
}
.address-list-del {
  display: none;
  position: absolute;
  top: 50%;
  right: 2px;
  margin-top: -10px;
  width: 20px;
  height: 20px;
  line-height: 20px;
  border-radius: 50%;
  text-align: center;
  background-color: #fef0f0;
  color: #f56c6c;
  cursor: pointer;
}
.u-clear {
  float: right;
  color: #67c23a;
  font-size: 14px;
  cursor: pointer;
}
.move_up_img {
  float: right;
  margin-top: 10px;
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.move_down_img {
  transform: rotate(180deg);
}
</style>

<style lang="scss">
#transfer{
.el-textarea{display:inline-block;width:100%;vertical-align:bottom;font-size:14px}.el-textarea__inner{display:block;resize:vertical;padding:5px 15px;line-height:1.5;-webkit-box-sizing:border-box;box-sizing:border-box;width:100%;font-size:inherit;color:#606266;background-color:#fff;background-image:none;border:1px solid #dcdfe6;border-radius:4px;-webkit-transition:border-color .2s cubic-bezier(.645,.045,.355,1);transition:border-color .2s cubic-bezier(.645,.045,.355,1)}.el-textarea__inner::-webkit-input-placeholder{color:#c0c4cc}.el-textarea__inner:-ms-input-placeholder{color:#c0c4cc}.el-textarea__inner::placeholder{color:#c0c4cc}.el-textarea__inner:hover{border-color:#c0c4cc}.el-textarea__inner:focus{outline:0;border-color:#409EFF}.el-textarea.is-disabled .el-textarea__inner{background-color:#f5f7fa;border-color:#e4e7ed;color:#c0c4cc;cursor:not-allowed}.el-textarea.is-disabled .el-textarea__inner::-webkit-input-placeholder{color:#c0c4cc}.el-textarea.is-disabled .el-textarea__inner:-ms-input-placeholder{color:#c0c4cc}.el-textarea.is-disabled .el-textarea__inner::placeholder{color:#c0c4cc}.el-input{position:relative;font-size:14px;display:inline-block;width:100%}.el-input::-webkit-scrollbar{z-index:11;width:6px}.el-input::-webkit-scrollbar:horizontal{height:6px}.el-input::-webkit-scrollbar-thumb{border-radius:5px;width:6px;background:#b4bccc}.el-input::-webkit-scrollbar-corner{background:#fff}.el-input::-webkit-scrollbar-track{background:#fff}.el-input::-webkit-scrollbar-track-piece{background:#fff;width:6px}.el-input .el-input__clear{color:#c0c4cc;font-size:14px;line-height:16px;cursor:pointer;-webkit-transition:color .2s cubic-bezier(.645,.045,.355,1);transition:color .2s cubic-bezier(.645,.045,.355,1)}.el-input .el-input__clear:hover{color:#909399}.el-input__inner{-webkit-appearance:none;background-color:#fff;background-image:none;border-radius:4px;border:1px solid #dcdfe6;-webkit-box-sizing:border-box;box-sizing:border-box;color:#606266;display:inline-block;font-size:inherit;height:40px;line-height:40px;outline:0;padding:0 15px;-webkit-transition:border-color .2s cubic-bezier(.645,.045,.355,1);transition:border-color .2s cubic-bezier(.645,.045,.355,1);width:100%}.el-input__prefix,.el-input__suffix{position:absolute;top:0;-webkit-transition:all .3s;text-align:center;height:100%;color:#c0c4cc}.el-input__inner::-webkit-input-placeholder{color:#c0c4cc}.el-input__inner:-ms-input-placeholder{color:#c0c4cc}.el-input__inner::placeholder{color:#c0c4cc}.el-input__inner:hover{border-color:#c0c4cc}.el-input.is-active .el-input__inner,.el-input__inner:focus{border-color:#409EFF;outline:0}.el-input__suffix{right:5px;transition:all .3s;pointer-events:none}.el-input__suffix-inner{pointer-events:all}.el-input__prefix{left:5px;transition:all .3s}.el-input__icon{height:100%;width:25px;text-align:center;-webkit-transition:all .3s;transition:all .3s;line-height:40px}.el-input__icon:after{content:'';height:100%;width:0;display:inline-block;vertical-align:middle}.el-input__validateIcon{pointer-events:none}.el-input.is-disabled .el-input__inner{background-color:#f5f7fa;border-color:#e4e7ed;color:#c0c4cc;cursor:not-allowed}.el-input.is-disabled .el-input__inner::-webkit-input-placeholder{color:#c0c4cc}.el-input.is-disabled .el-input__inner:-ms-input-placeholder{color:#c0c4cc}.el-input.is-disabled .el-input__inner::placeholder{color:#c0c4cc}.el-input.is-disabled .el-input__icon{cursor:not-allowed}.el-input--suffix .el-input__inner{padding-right:30px}.el-input--prefix .el-input__inner{padding-left:30px}.el-input--medium{font-size:14px}.el-input--medium .el-input__inner{height:36px;line-height:36px}.el-input--medium .el-input__icon{line-height:36px}.el-input--small{font-size:13px}.el-input--small .el-input__inner{height:32px;line-height:32px}.el-input--small .el-input__icon{line-height:32px}.el-input--mini{font-size:12px}.el-input--mini .el-input__inner{height:28px;line-height:28px}.el-input--mini .el-input__icon{line-height:28px}.el-input-group{line-height:normal;display:inline-table;width:100%;border-collapse:separate;border-spacing:0}.el-input-group>.el-input__inner{vertical-align:middle;display:table-cell}.el-input-group__append,.el-input-group__prepend{background-color:#f5f7fa;color:#909399;vertical-align:middle;display:table-cell;position:relative;border:1px solid #dcdfe6;border-radius:4px;padding:0 20px;width:1px;white-space:nowrap}.el-input-group--prepend .el-input__inner,.el-input-group__append{border-top-left-radius:0;border-bottom-left-radius:0}.el-input-group--append .el-input__inner,.el-input-group__prepend{border-top-right-radius:0;border-bottom-right-radius:0}.el-input-group__append:focus,.el-input-group__prepend:focus{outline:0}.el-input-group__append .el-button,.el-input-group__append .el-select,.el-input-group__prepend .el-button,.el-input-group__prepend .el-select{display:inline-block;margin:-10px -20px}.el-input-group__append button.el-button,.el-input-group__append div.el-select .el-input__inner,.el-input-group__append div.el-select:hover .el-input__inner,.el-input-group__prepend button.el-button,.el-input-group__prepend div.el-select .el-input__inner,.el-input-group__prepend div.el-select:hover .el-input__inner{border-color:transparent;background-color:transparent;color:inherit;border-top:0;border-bottom:0}.el-input-group__append .el-button,.el-input-group__append .el-input,.el-input-group__prepend .el-button,.el-input-group__prepend .el-input{font-size:inherit}.el-input-group__prepend{border-right:0}.el-input-group__append{border-left:0}.el-input-group--append .el-select .el-input.is-focus .el-input__inner,.el-input-group--prepend .el-select .el-input.is-focus .el-input__inner{border-color:transparent}.el-input__inner::-ms-clear{display:none;width:0;height:0}

.el-fade-in-enter,.el-fade-in-leave-active,.el-fade-in-linear-enter,.el-fade-in-linear-leave,.el-fade-in-linear-leave-active,.fade-in-linear-enter,.fade-in-linear-leave,.fade-in-linear-leave-active{opacity:0}.el-checkbox,.el-checkbox__input{display:inline-block;position:relative}.fade-in-linear-enter-active,.fade-in-linear-leave-active{-webkit-transition:opacity .2s linear;transition:opacity .2s linear}.el-fade-in-linear-enter-active,.el-fade-in-linear-leave-active{-webkit-transition:opacity .2s linear;transition:opacity .2s linear}.el-fade-in-enter-active,.el-fade-in-leave-active{-webkit-transition:all .3s cubic-bezier(.55,0,.1,1);transition:all .3s cubic-bezier(.55,0,.1,1)}.el-zoom-in-center-enter-active,.el-zoom-in-center-leave-active{-webkit-transition:all .3s cubic-bezier(.55,0,.1,1);transition:all .3s cubic-bezier(.55,0,.1,1)}.el-zoom-in-center-enter,.el-zoom-in-center-leave-active{opacity:0;-webkit-transform:scaleX(0);transform:scaleX(0)}.el-zoom-in-top-enter-active,.el-zoom-in-top-leave-active{opacity:1;-webkit-transform:scaleY(1);transform:scaleY(1);-webkit-transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);-webkit-transform-origin:center top;transform-origin:center top}.el-zoom-in-top-enter,.el-zoom-in-top-leave-active{opacity:0;-webkit-transform:scaleY(0);transform:scaleY(0)}.el-zoom-in-bottom-enter-active,.el-zoom-in-bottom-leave-active{opacity:1;-webkit-transform:scaleY(1);transform:scaleY(1);-webkit-transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);-webkit-transform-origin:center bottom;transform-origin:center bottom}.el-zoom-in-bottom-enter,.el-zoom-in-bottom-leave-active{opacity:0;-webkit-transform:scaleY(0);transform:scaleY(0)}.el-zoom-in-left-enter-active,.el-zoom-in-left-leave-active{opacity:1;-webkit-transform:scale(1,1);transform:scale(1,1);-webkit-transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1);transition:transform .3s cubic-bezier(.23,1,.32,1),opacity .3s cubic-bezier(.23,1,.32,1),-webkit-transform .3s cubic-bezier(.23,1,.32,1);-webkit-transform-origin:top left;transform-origin:top left}.el-zoom-in-left-enter,.el-zoom-in-left-leave-active{opacity:0;-webkit-transform:scale(.45,.45);transform:scale(.45,.45)}.collapse-transition{-webkit-transition:.3s height ease-in-out,.3s padding-top ease-in-out,.3s padding-bottom ease-in-out;transition:.3s height ease-in-out,.3s padding-top ease-in-out,.3s padding-bottom ease-in-out}.horizontal-collapse-transition{-webkit-transition:.3s width ease-in-out,.3s padding-left ease-in-out,.3s padding-right ease-in-out;transition:.3s width ease-in-out,.3s padding-left ease-in-out,.3s padding-right ease-in-out}.el-list-enter-active,.el-list-leave-active{-webkit-transition:all 1s;transition:all 1s}.el-list-enter,.el-list-leave-active{opacity:0;-webkit-transform:translateY(-30px);transform:translateY(-30px)}.el-opacity-transition{-webkit-transition:opacity .3s cubic-bezier(.55,0,.1,1);transition:opacity .3s cubic-bezier(.55,0,.1,1)}.el-checkbox{color:#606266;font-weight:500;font-size:14px;cursor:pointer;white-space:nowrap;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;user-select:none}.el-checkbox.is-bordered{padding:9px 20px 9px 10px;border-radius:4px;border:1px solid #dcdfe6;-webkit-box-sizing:border-box;box-sizing:border-box;line-height:normal;height:40px}.el-checkbox.is-bordered.is-checked{border-color:#409EFF}.el-checkbox.is-bordered.is-disabled{border-color:#ebeef5;cursor:not-allowed}.el-checkbox.is-bordered+.el-checkbox.is-bordered{margin-left:10px}.el-checkbox.is-bordered.el-checkbox--medium{padding:7px 20px 7px 10px;border-radius:4px;height:36px}.el-checkbox.is-bordered.el-checkbox--medium .el-checkbox__label{line-height:17px;font-size:14px}.el-checkbox.is-bordered.el-checkbox--medium .el-checkbox__inner{height:14px;width:14px}.el-checkbox.is-bordered.el-checkbox--small{padding:5px 15px 5px 10px;border-radius:3px;height:32px}.el-checkbox.is-bordered.el-checkbox--small .el-checkbox__label{line-height:15px;font-size:12px}.el-checkbox.is-bordered.el-checkbox--small .el-checkbox__inner{height:12px;width:12px}.el-checkbox.is-bordered.el-checkbox--small .el-checkbox__inner::after{height:6px;width:2px}.el-checkbox.is-bordered.el-checkbox--mini{padding:3px 15px 3px 10px;border-radius:3px;height:28px}.el-checkbox.is-bordered.el-checkbox--mini .el-checkbox__label{line-height:12px;font-size:12px}.el-checkbox-button__inner,.el-checkbox__input{line-height:1;vertical-align:middle;white-space:nowrap;outline:0}.el-checkbox.is-bordered.el-checkbox--mini .el-checkbox__inner{height:12px;width:12px}.el-checkbox.is-bordered.el-checkbox--mini .el-checkbox__inner::after{height:6px;width:2px}.el-checkbox__input{cursor:pointer}.el-checkbox__input.is-disabled .el-checkbox__inner{background-color:#edf2fc;border-color:#dcdfe6;cursor:not-allowed}.el-checkbox__input.is-disabled .el-checkbox__inner::after{cursor:not-allowed;border-color:#c0c4cc}.el-checkbox__input.is-disabled .el-checkbox__inner+.el-checkbox__label{cursor:not-allowed}.el-checkbox__input.is-disabled.is-checked .el-checkbox__inner{background-color:#f2f6fc;border-color:#dcdfe6}.el-checkbox__input.is-disabled.is-checked .el-checkbox__inner::after{border-color:#c0c4cc}.el-checkbox__input.is-disabled.is-indeterminate .el-checkbox__inner{background-color:#f2f6fc;border-color:#dcdfe6}.el-checkbox__input.is-disabled.is-indeterminate .el-checkbox__inner::before{background-color:#c0c4cc;border-color:#c0c4cc}.el-checkbox__input.is-checked .el-checkbox__inner,.el-checkbox__input.is-indeterminate .el-checkbox__inner{background-color:#409EFF;border-color:#409EFF}.el-checkbox__input.is-disabled+span.el-checkbox__label{color:#c0c4cc;cursor:not-allowed}.el-checkbox__input.is-checked .el-checkbox__inner::after{-webkit-transform:rotate(45deg) scaleY(1);transform:rotate(45deg) scaleY(1)}.el-checkbox__input.is-checked+.el-checkbox__label{color:#409EFF}.el-checkbox__input.is-focus .el-checkbox__inner{border-color:#409EFF}.el-checkbox__input.is-indeterminate .el-checkbox__inner::before{content:'';position:absolute;display:block;background-color:#fff;height:2px;-webkit-transform:scale(.5);transform:scale(.5);left:0;right:0;top:5px}.el-checkbox__input.is-indeterminate .el-checkbox__inner::after{display:none}.el-checkbox__inner{display:inline-block;position:relative;border:1px solid #dcdfe6;border-radius:2px;-webkit-box-sizing:border-box;box-sizing:border-box;width:14px;height:14px;background-color:#fff;z-index:1;-webkit-transition:border-color .25s cubic-bezier(.71,-.46,.29,1.46),background-color .25s cubic-bezier(.71,-.46,.29,1.46);transition:border-color .25s cubic-bezier(.71,-.46,.29,1.46),background-color .25s cubic-bezier(.71,-.46,.29,1.46)}.el-checkbox__inner:hover{border-color:#409EFF}.el-checkbox__inner::after{-webkit-box-sizing:content-box;box-sizing:content-box;content:"";border:1px solid #fff;border-left:0;border-top:0;height:7px;left:4px;position:absolute;top:1px;-webkit-transform:rotate(45deg) scaleY(0);transform:rotate(45deg) scaleY(0);width:3px;-webkit-transition:-webkit-transform .15s ease-in .05s;transition:-webkit-transform .15s ease-in .05s;transition:transform .15s ease-in .05s;transition:transform .15s ease-in .05s,-webkit-transform .15s ease-in .05s;-webkit-transform-origin:center;transform-origin:center}.el-checkbox__original{opacity:0;outline:0;position:absolute;margin:0;width:0;height:0;z-index:-1}.el-checkbox-button,.el-checkbox-button__inner{display:inline-block;position:relative}.el-checkbox__label{display:inline-block;padding-left:10px;line-height:19px;font-size:14px}.el-checkbox+.el-checkbox{margin-left:30px}.el-checkbox-button__inner{font-weight:500;cursor:pointer;background:#fff;border:1px solid #dcdfe6;border-left:0;color:#606266;-webkit-appearance:none;text-align:center;-webkit-box-sizing:border-box;box-sizing:border-box;margin:0;-webkit-transition:all .3s cubic-bezier(.645,.045,.355,1);transition:all .3s cubic-bezier(.645,.045,.355,1);-moz-user-select:none;-webkit-user-select:none;-ms-user-select:none;padding:12px 20px;font-size:14px;border-radius:0}.el-checkbox-button__inner.is-round{padding:12px 20px}.el-checkbox-button__inner:hover{color:#409EFF}.el-checkbox-button__inner [class*=el-icon-]{line-height:.9}.el-checkbox-button__inner [class*=el-icon-]+span{margin-left:5px}.el-checkbox-button__original{opacity:0;outline:0;position:absolute;margin:0;z-index:-1}.el-checkbox-button.is-checked .el-checkbox-button__inner{color:#fff;background-color:#409EFF;border-color:#409EFF;-webkit-box-shadow:-1px 0 0 0 #8cc5ff;box-shadow:-1px 0 0 0 #8cc5ff}.el-checkbox-button.is-checked:first-child .el-checkbox-button__inner{border-left-color:#409EFF}.el-checkbox-button.is-disabled .el-checkbox-button__inner{color:#c0c4cc;cursor:not-allowed;background-image:none;background-color:#fff;border-color:#ebeef5;-webkit-box-shadow:none;box-shadow:none}.el-checkbox-button.is-disabled:first-child .el-checkbox-button__inner{border-left-color:#ebeef5}.el-checkbox-button:first-child .el-checkbox-button__inner{border-left:1px solid #dcdfe6;border-radius:4px 0 0 4px;-webkit-box-shadow:none!important;box-shadow:none!important}.el-checkbox-button.is-focus .el-checkbox-button__inner{border-color:#409EFF}.el-checkbox-button:last-child .el-checkbox-button__inner{border-radius:0 4px 4px 0}.el-checkbox-button--medium .el-checkbox-button__inner{padding:10px 20px;font-size:14px;border-radius:0}.el-checkbox-button--medium .el-checkbox-button__inner.is-round{padding:10px 20px}.el-checkbox-button--small .el-checkbox-button__inner{padding:9px 15px;font-size:12px;border-radius:0}.el-checkbox-button--small .el-checkbox-button__inner.is-round{padding:9px 15px}.el-checkbox-button--mini .el-checkbox-button__inner{padding:7px 15px;font-size:12px;border-radius:0}.el-checkbox-button--mini .el-checkbox-button__inner.is-round{padding:7px 15px}.el-checkbox-group{font-size:0}.el-tree{position:relative;cursor:default;background:#fff;color:#606266}.el-tree__empty-block{position:relative;min-height:60px;text-align:center;width:100%;height:100%}.el-tree__empty-text{position:absolute;left:50%;top:50%;-webkit-transform:translate(-50%,-50%);transform:translate(-50%,-50%);color:#6f7180}.el-tree__drop-indicator{position:absolute;left:0;right:0;height:1px;background-color:#409EFF}.el-tree-node{white-space:nowrap;outline:0}.el-tree-node:focus>.el-tree-node__content{background-color:#f5f7fa}.el-tree-node.is-drop-inner>.el-tree-node__content .el-tree-node__label{background-color:#409EFF;color:#fff}.el-tree-node__content{display:-webkit-box;display:-ms-flexbox;display:flex;-webkit-box-align:center;-ms-flex-align:center;align-items:center;height:26px;cursor:pointer}.el-tree-node__content>.el-tree-node__expand-icon{padding:6px}.el-tree-node__content>.el-checkbox{margin-right:8px}.el-tree-node__content:hover{background-color:#f5f7fa}.el-tree.is-dragging .el-tree-node__content{cursor:move}.el-tree.is-dragging .el-tree-node__content *{pointer-events:none}.el-tree.is-dragging.is-drop-not-allow .el-tree-node__content{cursor:not-allowed}.el-tree-node__expand-icon{cursor:pointer;color:#c0c4cc;font-size:12px;-webkit-transform:rotate(0);transform:rotate(0);-webkit-transition:-webkit-transform .3s ease-in-out;transition:-webkit-transform .3s ease-in-out;transition:transform .3s ease-in-out;transition:transform .3s ease-in-out,-webkit-transform .3s ease-in-out}.el-tree-node__expand-icon.expanded{-webkit-transform:rotate(90deg);transform:rotate(90deg)}.el-tree-node__expand-icon.is-leaf{color:transparent;cursor:default}.el-tree-node__label{font-size:14px}.el-tree-node__loading-icon{margin-right:8px;font-size:14px;color:#c0c4cc}.el-tree-node>.el-tree-node__children{overflow:hidden;background-color:transparent}.el-tree-node.is-expanded>.el-tree-node__children{display:block}.el-tree--highlight-current .el-tree-node.is-current>.el-tree-node__content{background-color:#f0f7ff}

.el-button{display:inline-block;line-height:1;white-space:nowrap;cursor:pointer;background:#fff;border:1px solid #dcdfe6;color:#606266;-webkit-appearance:none;text-align:center;-webkit-box-sizing:border-box;box-sizing:border-box;outline:0;margin:0;-webkit-transition:.1s;transition:.1s;font-weight:500;-moz-user-select:none;-webkit-user-select:none;-ms-user-select:none;padding:12px 20px;font-size:14px;border-radius:4px}.el-button+.el-button{margin-left:10px}.el-button:focus,.el-button:hover{color:#409EFF;border-color:#c6e2ff;background-color:#ecf5ff}.el-button:active{color:#3a8ee6;border-color:#3a8ee6;outline:0}.el-button::-moz-focus-inner{border:0}.el-button [class*=el-icon-]+span{margin-left:5px}.el-button.is-plain:focus,.el-button.is-plain:hover{background:#fff;border-color:#409EFF;color:#409EFF}.el-button.is-active,.el-button.is-plain:active{color:#3a8ee6;border-color:#3a8ee6}.el-button.is-plain:active{background:#fff;outline:0}.el-button.is-disabled,.el-button.is-disabled:focus,.el-button.is-disabled:hover{color:#c0c4cc;cursor:not-allowed;background-image:none;background-color:#fff;border-color:#ebeef5}.el-button.is-disabled.el-button--text{background-color:transparent}.el-button.is-disabled.is-plain,.el-button.is-disabled.is-plain:focus,.el-button.is-disabled.is-plain:hover{background-color:#fff;border-color:#ebeef5;color:#c0c4cc}.el-button.is-loading{position:relative;pointer-events:none}.el-button.is-loading:before{pointer-events:none;content:'';position:absolute;left:-1px;top:-1px;right:-1px;bottom:-1px;border-radius:inherit;background-color:rgba(255,255,255,.35)}.el-button.is-round{border-radius:20px;padding:12px 23px}.el-button.is-circle{border-radius:50%;padding:12px}.el-button--primary{color:#fff;background-color:#409EFF;border-color:#409EFF}.el-button--primary:focus,.el-button--primary:hover{background:#66b1ff;border-color:#66b1ff;color:#fff}.el-button--primary.is-active,.el-button--primary:active{background:#3a8ee6;border-color:#3a8ee6;color:#fff}.el-button--primary:active{outline:0}.el-button--primary.is-disabled,.el-button--primary.is-disabled:active,.el-button--primary.is-disabled:focus,.el-button--primary.is-disabled:hover{color:#fff;background-color:#a0cfff;border-color:#a0cfff}.el-button--primary.is-plain{color:#409EFF;background:#ecf5ff;border-color:#b3d8ff}.el-button--primary.is-plain:focus,.el-button--primary.is-plain:hover{background:#409EFF;border-color:#409EFF;color:#fff}.el-button--primary.is-plain:active{background:#3a8ee6;border-color:#3a8ee6;color:#fff;outline:0}.el-button--primary.is-plain.is-disabled,.el-button--primary.is-plain.is-disabled:active,.el-button--primary.is-plain.is-disabled:focus,.el-button--primary.is-plain.is-disabled:hover{color:#8cc5ff;background-color:#ecf5ff;border-color:#d9ecff}.el-button--success{color:#fff;background-color:#67c23a;border-color:#67c23a}.el-button--success:focus,.el-button--success:hover{background:#85ce61;border-color:#85ce61;color:#fff}.el-button--success.is-active,.el-button--success:active{background:#5daf34;border-color:#5daf34;color:#fff}.el-button--success:active{outline:0}.el-button--success.is-disabled,.el-button--success.is-disabled:active,.el-button--success.is-disabled:focus,.el-button--success.is-disabled:hover{color:#fff;background-color:#b3e19d;border-color:#b3e19d}.el-button--success.is-plain{color:#67c23a;background:#f0f9eb;border-color:#c2e7b0}.el-button--success.is-plain:focus,.el-button--success.is-plain:hover{background:#67c23a;border-color:#67c23a;color:#fff}.el-button--success.is-plain:active{background:#5daf34;border-color:#5daf34;color:#fff;outline:0}.el-button--success.is-plain.is-disabled,.el-button--success.is-plain.is-disabled:active,.el-button--success.is-plain.is-disabled:focus,.el-button--success.is-plain.is-disabled:hover{color:#a4da89;background-color:#f0f9eb;border-color:#e1f3d8}.el-button--warning{color:#fff;background-color:#e6a23c;border-color:#e6a23c}.el-button--warning:focus,.el-button--warning:hover{background:#ebb563;border-color:#ebb563;color:#fff}.el-button--warning.is-active,.el-button--warning:active{background:#cf9236;border-color:#cf9236;color:#fff}.el-button--warning:active{outline:0}.el-button--warning.is-disabled,.el-button--warning.is-disabled:active,.el-button--warning.is-disabled:focus,.el-button--warning.is-disabled:hover{color:#fff;background-color:#f3d19e;border-color:#f3d19e}.el-button--warning.is-plain{color:#e6a23c;background:#fdf6ec;border-color:#f5dab1}.el-button--warning.is-plain:focus,.el-button--warning.is-plain:hover{background:#e6a23c;border-color:#e6a23c;color:#fff}.el-button--warning.is-plain:active{background:#cf9236;border-color:#cf9236;color:#fff;outline:0}.el-button--warning.is-plain.is-disabled,.el-button--warning.is-plain.is-disabled:active,.el-button--warning.is-plain.is-disabled:focus,.el-button--warning.is-plain.is-disabled:hover{color:#f0c78a;background-color:#fdf6ec;border-color:#faecd8}.el-button--danger{color:#fff;background-color:#f56c6c;border-color:#f56c6c}.el-button--danger:focus,.el-button--danger:hover{background:#f78989;border-color:#f78989;color:#fff}.el-button--danger.is-active,.el-button--danger:active{background:#dd6161;border-color:#dd6161;color:#fff}.el-button--danger:active{outline:0}.el-button--danger.is-disabled,.el-button--danger.is-disabled:active,.el-button--danger.is-disabled:focus,.el-button--danger.is-disabled:hover{color:#fff;background-color:#fab6b6;border-color:#fab6b6}.el-button--danger.is-plain{color:#f56c6c;background:#fef0f0;border-color:#fbc4c4}.el-button--danger.is-plain:focus,.el-button--danger.is-plain:hover{background:#f56c6c;border-color:#f56c6c;color:#fff}.el-button--danger.is-plain:active{background:#dd6161;border-color:#dd6161;color:#fff;outline:0}.el-button--danger.is-plain.is-disabled,.el-button--danger.is-plain.is-disabled:active,.el-button--danger.is-plain.is-disabled:focus,.el-button--danger.is-plain.is-disabled:hover{color:#f9a7a7;background-color:#fef0f0;border-color:#fde2e2}.el-button--info{color:#fff;background-color:#909399;border-color:#909399}.el-button--info:focus,.el-button--info:hover{background:#a6a9ad;border-color:#a6a9ad;color:#fff}.el-button--info.is-active,.el-button--info:active{background:#82848a;border-color:#82848a;color:#fff}.el-button--info:active{outline:0}.el-button--info.is-disabled,.el-button--info.is-disabled:active,.el-button--info.is-disabled:focus,.el-button--info.is-disabled:hover{color:#fff;background-color:#c8c9cc;border-color:#c8c9cc}.el-button--info.is-plain{color:#909399;background:#f4f4f5;border-color:#d3d4d6}.el-button--info.is-plain:focus,.el-button--info.is-plain:hover{background:#909399;border-color:#909399;color:#fff}.el-button--info.is-plain:active{background:#82848a;border-color:#82848a;color:#fff;outline:0}.el-button--info.is-plain.is-disabled,.el-button--info.is-plain.is-disabled:active,.el-button--info.is-plain.is-disabled:focus,.el-button--info.is-plain.is-disabled:hover{color:#bcbec2;background-color:#f4f4f5;border-color:#e9e9eb}.el-button--text,.el-button--text.is-disabled,.el-button--text.is-disabled:focus,.el-button--text.is-disabled:hover,.el-button--text:active{border-color:transparent}.el-button--medium{padding:10px 20px;font-size:14px;border-radius:4px}.el-button--mini,.el-button--small{font-size:12px;border-radius:3px}.el-button--medium.is-round{padding:10px 20px}.el-button--medium.is-circle{padding:10px}.el-button--small,.el-button--small.is-round{padding:9px 15px}.el-button--small.is-circle{padding:9px}.el-button--mini,.el-button--mini.is-round{padding:7px 15px}.el-button--mini.is-circle{padding:7px}.el-button--text{color:#409EFF;background:0 0;padding-left:0;padding-right:0}.el-button--text:focus,.el-button--text:hover{color:#66b1ff;border-color:transparent;background-color:transparent}.el-button--text:active{color:#3a8ee6;background-color:transparent}.el-button-group{display:inline-block;vertical-align:middle}.el-button-group::after,.el-button-group::before{display:table;content:""}.el-button-group::after{clear:both}.el-button-group .el-button{float:left;position:relative}.el-button-group .el-button+.el-button{margin-left:0}.el-button-group .el-button:first-child{border-top-right-radius:0;border-bottom-right-radius:0}.el-button-group .el-button:last-child{border-top-left-radius:0;border-bottom-left-radius:0}.el-button-group .el-button:first-child:last-child{border-radius:4px}.el-button-group .el-button:first-child:last-child.is-round{border-radius:20px}.el-button-group .el-button:first-child:last-child.is-circle{border-radius:50%}.el-button-group .el-button:not(:first-child):not(:last-child){border-radius:0}.el-button-group .el-button:not(:last-child){margin-right:-1px}.el-button-group .el-button.is-active,.el-button-group .el-button:active,.el-button-group .el-button:focus,.el-button-group .el-button:hover{z-index:1}.el-button-group .el-button--primary:first-child{border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--primary:last-child{border-left-color:rgba(255,255,255,.5)}.el-button-group .el-button--primary:not(:first-child):not(:last-child){border-left-color:rgba(255,255,255,.5);border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--success:first-child{border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--success:last-child{border-left-color:rgba(255,255,255,.5)}.el-button-group .el-button--success:not(:first-child):not(:last-child){border-left-color:rgba(255,255,255,.5);border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--warning:first-child{border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--warning:last-child{border-left-color:rgba(255,255,255,.5)}.el-button-group .el-button--warning:not(:first-child):not(:last-child){border-left-color:rgba(255,255,255,.5);border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--danger:first-child{border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--danger:last-child{border-left-color:rgba(255,255,255,.5)}.el-button-group .el-button--danger:not(:first-child):not(:last-child){border-left-color:rgba(255,255,255,.5);border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--info:first-child{border-right-color:rgba(255,255,255,.5)}.el-button-group .el-button--info:last-child{border-left-color:rgba(255,255,255,.5)}.el-button-group .el-button--info:not(:first-child):not(:last-child){border-left-color:rgba(255,255,255,.5);border-right-color:rgba(255,255,255,.5)}

@font-face{font-family:element-icons;src:url(./fonts/element-icons.woff) format("woff"),url(./fonts/element-icons.ttf) format("truetype");font-weight:400;font-style:normal}[class*=" el-icon-"],[class^=el-icon-]{font-family:element-icons!important;speak:none;font-style:normal;font-weight:400;font-variant:normal;text-transform:none;line-height:1;vertical-align:baseline;display:inline-block;-webkit-font-smoothing:antialiased;-moz-osx-font-smoothing:grayscale}.el-icon-info:before{content:"\e61a"}.el-icon-error:before{content:"\e62c"}.el-icon-success:before{content:"\e62d"}.el-icon-warning:before{content:"\e62e"}.el-icon-question:before{content:"\e634"}.el-icon-back:before{content:"\e606"}.el-icon-arrow-left:before{content:"\e600"}.el-icon-arrow-down:before{content:"\e603"}.el-icon-arrow-right:before{content:"\e604"}.el-icon-arrow-up:before{content:"\e605"}.el-icon-caret-left:before{content:"\e60a"}.el-icon-caret-bottom:before{content:"\e60b"}.el-icon-caret-top:before{content:"\e60c"}.el-icon-caret-right:before{content:"\e60e"}.el-icon-d-arrow-left:before{content:"\e610"}.el-icon-d-arrow-right:before{content:"\e613"}.el-icon-minus:before{content:"\e621"}.el-icon-plus:before{content:"\e62b"}.el-icon-remove:before{content:"\e635"}.el-icon-circle-plus:before{content:"\e601"}.el-icon-remove-outline:before{content:"\e63c"}.el-icon-circle-plus-outline:before{content:"\e602"}.el-icon-close:before{content:"\e60f"}.el-icon-check:before{content:"\e611"}.el-icon-circle-close:before{content:"\e607"}.el-icon-circle-check:before{content:"\e639"}.el-icon-circle-close-outline:before{content:"\e609"}.el-icon-circle-check-outline:before{content:"\e63e"}.el-icon-zoom-out:before{content:"\e645"}.el-icon-zoom-in:before{content:"\e641"}.el-icon-d-caret:before{content:"\e615"}.el-icon-sort:before{content:"\e640"}.el-icon-sort-down:before{content:"\e630"}.el-icon-sort-up:before{content:"\e631"}.el-icon-tickets:before{content:"\e63f"}.el-icon-document:before{content:"\e614"}.el-icon-goods:before{content:"\e618"}.el-icon-sold-out:before{content:"\e63b"}.el-icon-news:before{content:"\e625"}.el-icon-message:before{content:"\e61b"}.el-icon-date:before{content:"\e608"}.el-icon-printer:before{content:"\e62f"}.el-icon-time:before{content:"\e642"}.el-icon-bell:before{content:"\e622"}.el-icon-mobile-phone:before{content:"\e624"}.el-icon-service:before{content:"\e63a"}.el-icon-view:before{content:"\e643"}.el-icon-menu:before{content:"\e620"}.el-icon-more:before{content:"\e646"}.el-icon-more-outline:before{content:"\e626"}.el-icon-star-on:before{content:"\e637"}.el-icon-star-off:before{content:"\e63d"}.el-icon-location:before{content:"\e61d"}.el-icon-location-outline:before{content:"\e61f"}.el-icon-phone:before{content:"\e627"}.el-icon-phone-outline:before{content:"\e628"}.el-icon-picture:before{content:"\e629"}.el-icon-picture-outline:before{content:"\e62a"}.el-icon-delete:before{content:"\e612"}.el-icon-search:before{content:"\e619"}.el-icon-edit:before{content:"\e61c"}.el-icon-edit-outline:before{content:"\e616"}.el-icon-rank:before{content:"\e632"}.el-icon-refresh:before{content:"\e633"}.el-icon-share:before{content:"\e636"}.el-icon-setting:before{content:"\e638"}.el-icon-upload:before{content:"\e60d"}.el-icon-upload2:before{content:"\e644"}.el-icon-download:before{content:"\e617"}.el-icon-loading:before{content:"\e61e"}.el-icon-loading{-webkit-animation:rotating 2s linear infinite;animation:rotating 2s linear infinite}.el-icon--right{margin-left:5px}.el-icon--left{margin-right:5px}@-webkit-keyframes rotating{0%{-webkit-transform:rotateZ(0);transform:rotateZ(0)}100%{-webkit-transform:rotateZ(360deg);transform:rotateZ(360deg)}}@keyframes rotating{0%{-webkit-transform:rotateZ(0);transform:rotateZ(0)}100%{-webkit-transform:rotateZ(360deg);transform:rotateZ(360deg)}}
}
</style>

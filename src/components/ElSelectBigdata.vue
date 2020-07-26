<template>
  <el-select
    v-bind="$attrs"
    v-on="$listeners"
    filterable
    :filter-method="filterMethod"
    v-load-bigdata="{ callback: loadmore, dist: dist }"
  >
    <slot :list="list"></slot>
  </el-select>
</template>

<script>
export default {
  props: {
    // 要加载的数据
    bigdata: {
      type: Array,
      default() {
        return [];
      },
    },
    // 距离底部多少像素触发加载
    dist: {
      type: Number,
      default: 50,
    },
    // 每次加载多少个
    number: {
      type: Number,
      default: 30,
    },
    // 根据哪个字段搜索
    filterField: {
      type: String,
      default: 'label',
    }
  },
  data() {
    return {
      value: "",
      list: [],
    };
  },
  watch: {
    bigdata(value) {
      this.list = value.slice(0, this.number);
    },
  },
  methods: {
    filterMethod(value) {
      if (!value) {
        this.list = this.bigdata.slice(0, this.number);
        return;
      }
      const data = this.bigdata;
      const list = [];
      for (let i = 0; i < data.length; i++) {
        if (list.length >= 7) break;
        if (data[i][this.filterField].indexOf(value) > -1) {
          list.push(data[i]);
        }
      }
      this.list = list;
    },
    loadmore() {
      const { length } = this.list;
      this.list = this.bigdata.slice(0, length + this.number);
    },
  },
  directives: {
    "load-bigdata": {
      bind(el, binding) {
        // 获取element-ui定义好的scroll盒子
        const S_DOM = el.querySelector(
          ".el-select-dropdown .el-select-dropdown__wrap"
        );
        const {
          value: { dist, callback },
        } = binding;

        S_DOM.addEventListener("scroll", function() {
          /**
           * scrollHeight 获取元素内容高度(只读)
           * scrollTop 获取或者设置元素的偏移值,常用于, 计算滚动条的位置, 当一个元素的容器没有产生垂直方向的滚动条, 那它的scrollTop的值默认为0.
           * clientHeight 读取元素的可见高度(只读)
           * 如果元素滚动到底, 下面等式返回true, 没有则返回false:
           * ele.scrollHeight - ele.scrollTop === ele.clientHeight;
           * dist：距离底部还有多少像素就触发callback
           */
          const flag =
            this.scrollHeight - this.scrollTop - dist <= this.clientHeight;
          if (flag) {
            callback();
          }
        });
      },
    },
  },
};
</script>

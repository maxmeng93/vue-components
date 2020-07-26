# el-select 大量数据下拉框渲染优化

## `ElSelectBigdata` 组件

<br />解决了大数据下渲染卡顿问题，通过监听 `scroll` 事件，每次滑动到列表底部时，再追加渲染一部分数据。<br />组件默认支持了**可搜索**功能，不过由于 `select option` 是动态加载的，原 `el-select` 的逻辑已无法满足搜索要求，因此我自己实现了 filterMethod。<br />

### Attributes

| 参数         | 说明                               | 类型   | 默认值 |
| ------------ | ---------------------------------- | ------ | ------ |
| bigdata      | 要渲染的大数据列表                 | Array  | []     |
| dist         | 距离列表底部时多少像素触发追加渲染 | Number | 50     |
| number       | 每次渲染多少条数据                 | Number | 30     |
| filter-field | 根据哪个字段搜索                   | String | label  |

### Scoped Slot

| name | 说明                                 |
| ---- | ------------------------------------ |
| -    | 要渲染的 option 列表，参数为{ list } |

### 组件代码

```html
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
      },
    },
    data() {
      return {
        value: '',
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
      'load-bigdata': {
        bind(el, binding) {
          // 获取element-ui定义好的scroll盒子
          const S_DOM = el.querySelector(
            '.el-select-dropdown .el-select-dropdown__wrap',
          );
          const {
            value: { dist, callback },
          } = binding;

          S_DOM.addEventListener('scroll', function() {
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
```

### 使用

```html
<template>
  <el-form style="margin: 0 auto; width: 500px;">
    <el-form-item>下拉选择框动态加载大数据DEMO</el-form-item>
    <el-form-item>
      <el-select-bigdata
        v-model="value"
        :bigdata="bigdata"
        filter-field="label"
      >
        <template v-slot="{ list }">
          <el-option
            v-for="item in list"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </template>
      </el-select-bigdata>
    </el-form-item>
  </el-form>
</template>

<script>
  import ElSelectBigdata from '@/components/ElSelectBigdata';

  export default {
    components: {
      ElSelectBigdata,
    },
    data() {
      return {
        value: '',
        bigdata: [],
      };
    },
    mounted() {
      const bigdata = [];
      for (let i = 0; i < 1000; i++) {
        bigdata.push({
          label: `选项${i}`,
          value: `${i}`,
        });
      }
      this.bigdata = bigdata;
    },
  };
</script>
```

## element-ui 代码分析

### el-select

```javascript
// element-ui/packages/select/src/select.vue 454行
handleQueryChange(val) {
  if (this.previousQuery === val || this.isOnComposition) return;
  if (
    this.previousQuery === null &&
    (typeof this.filterMethod === 'function' || typeof this.remoteMethod === 'function')
  ) {
    this.previousQuery = val;
    return;
  }
  this.previousQuery = val;
  this.$nextTick(() => {
    if (this.visible) this.broadcast('ElSelectDropdown', 'updatePopper');
  });
  this.hoverIndex = -1;
  if (this.multiple && this.filterable) {
    this.$nextTick(() => {
      const length = this.$refs.input.value.length * 15 + 20;
      this.inputLength = this.collapseTags ? Math.min(50, length) : length;
      this.managePlaceholder();
      this.resetInputHeight();
    });
  }
  if (this.remote && typeof this.remoteMethod === 'function') {
    this.hoverIndex = -1;
    this.remoteMethod(val);
  } else if (typeof this.filterMethod === 'function') {
    this.filterMethod(val);
    this.broadcast('ElOptionGroup', 'queryChange');
  } else {
    this.filteredOptionsCount = this.optionsCount;
    // 1. select组件的 query 改变时
    // 2. 广播 queryChange 事件给 options 组件
    this.broadcast('ElOption', 'queryChange', val);
    this.broadcast('ElOptionGroup', 'queryChange');
  }
  if (this.defaultFirstOption && (this.filterable || this.remote) && this.filteredOptionsCount) {
    this.checkDefaultFirstOption();
  }
}
```

### el-option

```html
<!-- element-ui/packages/select/src/option.vue 1行 -->
<template>
  <li
    @mouseenter="hoverItem"
    @click.stop="selectOptionClick"
    class="el-select-dropdown__item"
    v-show="visible"
    :class="{
      'selected': itemSelected,
      'is-disabled': disabled || groupDisabled || limitReached,
      'hover': hover
    }"
  >
    <slot>
      <span>{{ currentLabel }}</span>
    </slot>
  </li>
</template>
```

```javascript
// element-ui/packages/select/src/option.vue 137行
queryChange(query) {
  // 3. option组件接受父组件的广播，决定自己是否显示
  this.visible = new RegExp(escapeRegexpString(query), 'i').test(this.currentLabel) || this.created;
  if (!this.visible) {
    this.select.filteredOptionsCount--;
  }
}
```

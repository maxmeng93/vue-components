# 动态设置多行文本超出隐藏，可展开/收起

## `Paragraph` 组件

### Attributes

| 参数       | 说明                      | 类型    | 默认值 |
| ---------- | ------------------------- | ------- | ------ |
| text       | 文本内容                  | String  | ""     |
| rows       | 最大显示行数              | Number  | 1      |
| expandable | 是否显示 "展开/折叠" 按钮 | Boolean | false  |

### 组件代码

```html
<template>
  <div class="paragraph-wrap">
    <div ref="paragraph" class="paragraph" :class="{ isOutRange: isCollapse }">
      {{ text }}
    </div>
    <div class="operation" v-if="isOutRange && expandable">
      <span @click="isCollapse = !isCollapse">
        {{ isCollapse ? '展开' : '折叠' }}
      </span>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Paragraph',
    props: {
      // 文本内容
      text: {
        type: String,
        default: '',
      },
      // 最大显示行数
      rows: {
        type: Number,
        default: 1,
      },
      // 是否显示 "展开/折叠" 按钮
      expandable: {
        type: Boolean,
        default: false,
      },
    },
    data() {
      return {
        isOutRange: false,
        isCollapse: false,
      };
    },
    mounted() {
      this.hasOutRange();
    },
    methods: {
      hasOutRange() {
        const rows = this.rows;

        const originEle = this.$refs['paragraph'];
        const originStyle = window.getComputedStyle(originEle);
        const lineHeight = this.pxToNumber(originStyle.lineHeight);
        const offsetHeight = originEle.offsetHeight;
        const maxHeight = Math.round(
          lineHeight * rows + this.pxToNumber(originStyle.paddingTop),
        );

        if (offsetHeight > maxHeight) {
          this.isOutRange = true;
          this.isCollapse = true;
          originEle.style.webkitLineClamp = rows;
        } else {
          this.isOutRange = false;
          this.isCollapse = false;
        }
      },
      pxToNumber(value) {
        if (!value) return 0;
        const match = value.match(/^\d*(\.\d*)?/);
        return match ? Number(match[0]) : 0;
      },
      recalculation() {
        this.isOutRange = false;
        this.isCollapse = false;
        this.$nextTick(() => {
          this.hasOutRange();
        });
      },
    },
    watch: {
      text() {
        this.recalculation();
      },
      rows() {
        this.recalculation();
      },
    },
  };
</script>

<style scoped>
  .paragraph {
    /* 文本容器不可以设置底部的内边距，否在会把隐藏掉的文本显示出来，如需设置底部边距请使用margin */
    padding-bottom: 0 !important;
    line-height: 1.3;
  }

  .paragraph.isOutRange {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 1;
    -webkit-box-orient: vertical;
  }

  .operation {
    text-align: right;
  }

  .operation > span {
    color: #1890ff;
    cursor: pointer;
  }
</style>
```

### 使用

```html
<template>
  <div>
    <h2>多行文本超出隐藏</h2>
    <el-input-number v-model="rows" :min="1"></el-input-number>
    <el-input type="textarea" v-model="text" rows="6" />
    <Paragraph :text="text" :rows="rows" expandable />
  </div>
</template>

<script>
  import Paragraph from './index';

  export default {
    name: 'demo2',
    components: {
      Paragraph,
    },
    data() {
      return {
        rows: 1,
        text:
          "To be, or not to be, that is a question: Whether it is nobler in the mind to suffer. The slings and arrows of outrageous fortune Or to take arms against a sea of troubles, And by opposing end them? To die: to sleep; No more; and by a sleep to say we end The heart-ache and the thousand natural shocks That flesh is heir to, 'tis a consummation Devoutly to be wish'd. To die, to sleep To sleep- perchance to dream: ay, there's the rub! For in that sleep of death what dreams may come When we have shuffled off this mortal coil, Must give us pause. There 's the respect That makes calamity of so long life",
      };
    },
  };
</script>
```

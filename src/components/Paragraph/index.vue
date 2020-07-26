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

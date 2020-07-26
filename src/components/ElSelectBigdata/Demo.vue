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

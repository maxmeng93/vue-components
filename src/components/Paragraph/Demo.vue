<template>
  <div>
    <h2>多行文本超出隐藏</h2>
    <el-form label-width="100px">
      <el-form-item label="行数">
        <el-input-number v-model="rows" :min="1"></el-input-number>
      </el-form-item>
      <el-form-item label="是否可展开">
        <el-switch
          v-model="expandable"
          active-color="#13ce66"
          inactive-color="#ff4949"
        ></el-switch>
      </el-form-item>
      <el-form-item label="文本">
        <el-input type="textarea" v-model="text" rows="6" />
      </el-form-item>
    </el-form>
    <Paragraph :text="text" :rows="rows" :expandable="expandable" />
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
      expandable: true,
      text:
        "To be, or not to be, that is a question: Whether it is nobler in the mind to suffer. The slings and arrows of outrageous fortune Or to take arms against a sea of troubles, And by opposing end them? To die: to sleep; No more; and by a sleep to say we end The heart-ache and the thousand natural shocks That flesh is heir to, 'tis a consummation Devoutly to be wish'd. To die, to sleep To sleep- perchance to dream: ay, there's the rub! For in that sleep of death what dreams may come When we have shuffled off this mortal coil, Must give us pause. There 's the respect That makes calamity of so long life",
    };
  },
};
</script>

<template>
  <div class="about">
    <el-button type="primary" @click="handleClick1"
      >特征和验证数据处理</el-button
    >
    <el-button type="primary" @click="handleClick2">A7Test数据处理</el-button>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import { defineComponent } from "vue";
import { ElButton } from "element-plus";
export default class AboutView extends Vue {
  handleClick1() {
    // window.location.href = "http://114.55.227.234:8081/";
    window.location.href = "http://192.168.8.2:8081/";
  }
  handleClick2() {
    console.log("1111");
  }
}
</script>

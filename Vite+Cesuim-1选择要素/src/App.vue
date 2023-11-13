<template>
  <!-- <chooseFeature></chooseFeature>//选择要素 -->
  <!-- <MOdel_Coloring></MOdel_Coloring>//模型样式 -->
  <!-- <selectedPlane></selectedPlane> -->
  <!-- <DrawModel></DrawModel> -->
  <modelCzml></modelCzml>
</template>

<script setup>
import chooseFeature from "../src/Test/1.chooseFeature.vue";
import MOdel_Coloring from "../src/Test/2.3D_Model_Coloring.vue";
import selectedPlane from "../src/Test/3.裁剪平面.vue"
import DrawModel from "../src/Test/4.绘制模型.vue"
import modelCzml from "../src/Test/5.卫星轨迹.vue"
const components  =[
  chooseFeature,
  MOdel_Coloring,
  selectedPlane,
  DrawModel,
  modelCzml,
]
</script>
<style scoped>
</style>

<script setup>
import {ref, onMounted} from "vue";
import axios from "../http/axios";

const props = defineProps({
  label: String,
  place: String,
  index: Number,
  type: String,
  projectId: Number,
  step: Object,
})
const pageData = ref({
  content: []
});
const pageSize = ref(10);
const currentPage = ref(0)
const findByProjectIdAndEleType = (event, pageNum, pSize) => {
  if (event) {
    props.step.elements[props.index] = null
    axios.get("/controller/elements/list", {
      params: {
        projectId: props.projectId,
        type: props.type,
        page: pageNum || 1,
        pageSize: pSize || pageSize.value,
      }
    }).then(resp => {
      pageData.value = resp.data
      currentPage.value = pageData.value['number'] + 1
    })
  }
}
onMounted(() => {
  if (props.step.elements[props.index]) {
    pageData.value['content'].push(props.step.elements[props.index])
  }
})
</script>
<template>
  <el-form-item
      :label="label"
      :rules="[
            { required: true, message: place, trigger: 'change' },
          ]"
      :prop="'elements['+index+']'"
  >
    <el-select
        value-key="id"
        v-model="step.elements[index]"
        :placeholder="place"
        @visible-change="findByProjectIdAndEleType"
    >
      <el-option
          v-for="item in pageData['content']"
          :key="item.id"
          :label="item['eleName']"
          :value="item"
      ></el-option>
      <div style="text-align: center;margin-top: 5px">
        <el-pagination small layout="prev, pager, next"
                       v-model:current-page="currentPage"
                       :total="pageData['totalElements']"
                       :page-size="pageSize"
                       @current-change="findByProjectIdAndEleType(true,$event)">
        </el-pagination>
      </div>
    </el-select>
  </el-form-item>
</template>
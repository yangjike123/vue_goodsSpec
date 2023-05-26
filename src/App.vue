

<script  setup>
import { ElMessage } from 'element-plus';
import { ref, reactive, nextTick, onMounted } from 'vue';

const specName = ref(''),//规格名
  inputTags = ref(''), // 规格值
  inputVisible = ref(false),
  spec = reactive({ specList: [] });
const InputRef = ref(null);

onMounted(() => {
  console.log(InputRef);
})

function addSpecName() {
  if (specName.value) {
    spec.specList.push({ label: specName.value, tags: [] });
    specName.value = '';
  } else {
    ElMessage.warning('请输入规格名称')
  }
}
function handleInputConfirm() {

  inputTags.value = '';
  inputVisible.value = !inputVisible.value;
}
function onClilkInput() {
  inputVisible.value = !inputVisible.value;
  nextTick(() => {
    // 

    // console.log('InputRef.value?.input: ', InputRef.value?.input);
  })
}
// console.log('specValue: ', specValue);

</script>
<template>
  <div style="width: 1200px;margin: 30px auto;">
    <div style="margin-left: 100px;">
      <el-popover width="370" placement="bottom" trigger="click">
        <div>
          <el-input v-model="specName" @keyup.enter="addSpecName()" placeholder="请输入规格名称">
            <template #append>
              <el-button type="primary" size="small" @click="addSpecName()">添加规格名</el-button>
            </template>
          </el-input>
        </div>
        <template #reference>
          <el-button>添加规格</el-button>
        </template>
      </el-popover>
    </div>
    <div style="width: 700px;">
      <div v-if="spec.specList.length > 0">
        <div v-for="(item, index) in spec.specList" :key="index">
          <div>
            <h3 style="display: flex;align-items: center;">
              <span>{{ item.label }}</span>
              <span style="margin-left: 15px;">
                <el-icon>
                  <Close />
                </el-icon>
              </span>
            </h3>
          </div>
          <div style="display: flex;">
            <div v-for="(tagNmae, tagIndex) in item.tags" :key="tagIndex">
              <el-tag>{{ tagNmae }}</el-tag>
            </div>
            <div>
              <el-input v-if="inputVisible" ref="InputRef" v-model="inputTags" @keyup.enter="handleInputConfirm(index)"
                @blur="handleInputConfirm(index)" />
              <el-button v-else @click="onClilkInput">添加规格值</el-button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <el-table style="width: 100%">
      <el-table-column prop="date" label="Date" width="180" />
      <el-table-column prop="name" label="Name" width="180" />
      <el-table-column prop="address" label="Address" />
    </el-table>
  </div>
</template>
<style></style>

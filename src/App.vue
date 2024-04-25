<template>
  <div style="width: 1200px; margin: 30px auto">
    <div style="margin-left: 100px">
      <el-popover width="370" placement="bottom" trigger="click">
        <div>
          <el-input
            v-model="specName"
            ref="InpuSpecRef"
            @keyup.enter="addSpecName()"
            placeholder="请输入规格名称"
          >
            <template #append>
              <el-button type="primary" size="small" @click="addSpecName()"
                >添加规格名</el-button
              >
            </template>
          </el-input>
        </div>
        <template #reference>
          <el-button>添加规格</el-button>
        </template>
      </el-popover>
    </div>
    <div style="width: 700px">
      <div v-if="specData.specList.length > 0">
        <div v-for="(item, index) in specData.specList" :key="index">
          <div>
            <h3 style="display: flex; align-items: center">
              <span>{{ item.label }}</span>
              <span style="margin-left: 15px" @click="onDeletSpec(index)">
                <el-icon>
                  <Close />
                </el-icon>
              </span>
            </h3>
          </div>
          <div style="display: flex">
            <div v-for="(tagNmae, tagIndex) in item.tags" :key="tagIndex">
              <el-tag
                @close="onDeletSpecValue(index, tagIndex)"
                closable
                style="margin: 0 10px"
                >{{ tagNmae }}</el-tag
              >
            </div>
            <div>
              <el-input
                v-if="inputIndex === index && inputVisible"
                size="small"
                ref="InputRef"
                v-model="inputTags"
                @keyup.enter="handleInputConfirm(index, item.label)"
                @blur="handleInputConfirm(index, item.label)"
              />
              <el-button v-else size="small" @click="onClilkInput(index)"
                >添加规格值</el-button
              >
            </div>
          </div>
        </div>
      </div>
    </div>
    <el-table :data="submitList" border>
      <el-table-column
        v-for="(item, index) in specData.specList"
        :label="item.label"
        :key="index"
      >
        <template #default="{ row, column }">
          <span>{{ row[column.label] }}</span>
        </template>
      </el-table-column>
      <el-table-column
        v-for="(item, index) in tableHader"
        :label="item.label"
        :key="index"
      >
        <template #default="{ $index }">
          <el-input-number
            v-if="item.label === '价格'"
            v-model="submitList[$index][item.key]"
            :precision="2"
            size="small"
          />
          <el-input-number
            v-else
            v-model="submitList[$index][item.key]"
            size="small"
          />
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script setup>
import { ElMessage } from "element-plus";
import { ref, reactive, nextTick, watch, onMounted } from "vue";
// ------------变量----------------
const specName = ref(""), //规格名
  inputTags = ref(""), // 规格值
  inputVisible = ref(false),
  inputIndex = ref(0),
  // tableSkuList = ref([]), // table显示数量和显示值
  submitList = ref([]),
  specData = reactive({
    specList: [],
  });

const InputRef = ref(null);
const InpuSpecRef = ref(null);
const tableHader = [
  { label: "价格", key: "price" },
  { label: "库存", key: "stock" },
];
// ------------变量----------------

// ------------生命周期----------------
// watch(submitList, (newValue) => {}, { deep: true });
onMounted(() => {
  tableSKU();
});
// ------------生命周期----------------

// -----------function-----------------
function addSpecName() {
  if (!specName.value) return ElMessage.warning("请输入规格名称");
  const isExist = specData.specList.find(
    (item) => item.label === specName.value
  );
  if (isExist) ElMessage.warning(`"${specName.value}"规格名已存在`);
  else {
    specData.specList.push({ label: specName.value, tags: [] });
    specName.value = "";
  }
}
function handleInputConfirm(index) {
  if (inputTags.value) {
    //找到并吧输入值push到tags里面
    specData.specList[index].tags.push(inputTags.value);
    tableSKU();
  }
  inputTags.value = "";
  inputVisible.value = !inputVisible.value;
}
function onClilkInput(index) {
  inputIndex.value = index; // 获取输入框下标
  inputVisible.value = !inputVisible.value; // 显示输入框
  nextTick(() => {
    InputRef.value[0]?.input?.focus();
  });
}
function onDeletSpec(index) {
  //删除规格
  specData.specList.splice(index, 1); // 同时删除展示的标签
  tableSKU(); //触发tableSKU方法 重绘数据和页面
}
function onDeletSpecValue(index, tagIndex) {
  //删除规格值
  specData.specList[index].tags.splice(tagIndex, 1); // 规格值
  tableSKU(); //重绘数据和页面
}
function tableSKU() {
  let temp = [];
  specData.specList.forEach((item, index) => {
    if (!temp.length) {
      temp.push(
        ...item.tags.map((t) => {
          const oldValue = submitList.value.find((v) => v.sku === t);
          if (oldValue) {
            return { ...oldValue };
          } else {
            return {
              [`skuName${index + 1}`]: item.label,
              [`skuValue${index + 1}`]: t,
              [item.label]: t,
              price: 0,
              stock: 0,
              sku: t,
            };
          }
        })
      );
    } else {
      const array = [];
      temp.forEach((obj) => {
        if (item.tags.length === 0) array.push(obj);
        array.push(
          ...item.tags.map((t) => {
            if (obj.sku) obj.sku = obj.sku + t; //唯一值
            const oldValue = submitList.value.find(
              (item) => item.sku === obj.sku
            );
            if (oldValue)
              return { ...oldValue }; // 必须展开否则会变成 proxy对象
            else {
              return {
                ...obj,
                [`skuName${index + 1}`]: item.label,
                [`skuValue${index + 1}`]: t,
                [item.label]: t,
                price: 0,
                stock: 0,
              };
            }
          })
        );
      });
      temp = array;
    }
  });
  submitList.value = temp;
}

// -----------function-----------------
</script>

<style></style>

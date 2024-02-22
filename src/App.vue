<template>
	<div style="width: 1200px;margin: 30px auto;">
		<div style="margin-left: 100px;">
			<el-popover width="370" placement="bottom" trigger="click">
				<div>
					<el-input v-model="specName" ref="InpuSpecRef" @keyup.enter="addSpecName()" placeholder="请输入规格名称">
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
			<div v-if="specData.specList.length > 0">
				<div v-for="(item, index) in specData.specList" :key="index">
					<div>
						<h3 style="display: flex;align-items: center;">
							<span>{{ item.label }}</span>
							<span style="margin-left: 15px;" @click="onDeletSpec(index)">
								<el-icon>
									<Close />
								</el-icon>
							</span>
						</h3>
					</div>
					<div style="display: flex;">
						<div v-for="(tagNmae, tagIndex) in item.tags" :key="tagIndex">
							<el-tag @close="onDeletSpecValue(index, tagIndex)" closable style="margin: 0 10px;">{{ tagNmae
							}}</el-tag>
						</div>
						<div>
							<el-input v-if="inputIndex === index && inputVisible" size="small" ref="InputRef"
								v-model="inputTags" @keyup.enter="handleInputConfirm(index, item.label)"
								@blur="handleInputConfirm(index, item.label)" />
							<el-button v-else size="small" @click="onClilkInput(index)">添加规格值</el-button>
						</div>
					</div>
				</div>
			</div>
		</div>
		<el-table :data="submitList" border>
			<el-table-column v-for="(item, index) in specData.specList" :label="item.label" :key="index">
				<template #default="{ row, column }">
					<span>{{ row[column.label] }}</span>
				</template>
			</el-table-column>
			<el-table-column v-for="(item, index) in tableHader" :label="item.label" :key="index">
				<template #default="{ $index }">
					<el-input-number v-if="item.label === '价格'" v-model="submitList[$index][item.key]" :precision="2"
						size="small" />
					<el-input-number v-else v-model="submitList[$index][item.key]" size="small" />
				</template>
			</el-table-column>

		</el-table>
	</div>
</template>

<script setup>
import { ElMessage } from 'element-plus';
import { ref, reactive, nextTick, watch } from 'vue';
// ------------变量----------------
const specName = ref(''),//规格名
	inputTags = ref(''), // 规格值
	inputVisible = ref(false),
	inputIndex = ref(0),
	// tableSkuList = ref([]), // table显示数量和显示值
	submitList = ref([]),
	specData = reactive({ specList: [], skuValue: {} });

const InputRef = ref(null);
const InpuSpecRef = ref(null);
const tableHader = [
	{ label: '价格', key: 'price' },
	{ label: '库存', key: 'stock' }
]
// ------------变量----------------

// ------------生命周期----------------
watch(submitList, (newValue) => {
}, { deep: true });
// ------------生命周期----------------

// -----------function-----------------
function addSpecName() {
	if (specName.value) {
		specData.specList.push({ label: specName.value, tags: [] });
		specName.value = '';
	} else {
		ElMessage.warning('请输入规格名称')
	}
}
function handleInputConfirm(index, label) {
	if (inputTags.value) {
		//找到并吧输入值push到tags里面
		specData.specList[index].tags.push(inputTags.value);
		if (!specData.skuValue[label]) specData.skuValue[label] = [inputTags.value];
		else specData.skuValue[label].push(inputTags.value);
		tableSKU();
	}
	inputTags.value = '';
	inputVisible.value = !inputVisible.value;
}
function onClilkInput(index) {
	inputIndex.value = index; // 获取输入框下标
	inputVisible.value = !inputVisible.value; // 显示输入框
	nextTick(() => {
		InputRef.value[0]?.input?.focus();
	});
}
function onDeletSpec(index) { //删除规格
	delete specData.skuValue[specData.specList[index].label]; //找到相对于的名称，然后删除skuValue相对于的值
	specData.specList.splice(index, 1); // 同时删除展示的标签
	tableSKU(); //触发tableSKU方法 重绘数据和页面
}
function onDeletSpecValue(index, tagIndex) {//删除规格值
	specData.skuValue[specData.specList[index].label].splice(tagIndex, 1);//删除tags里面的其中一个值
	specData.specList[index].tags.splice(tagIndex, 1); // 规格值
	tableSKU();//重绘数据和页面
}
function tableSKU() {
	const list = specData.specList.reduce((def, item, index) => {
		const data = [];
		def.forEach(t => {
			item.tags.forEach(tag => {
				if (t.sku) t.sku += `,${tag}`;
				else t.sku = tag;
				const oldValue = submitList.value.find(o => o.sku === t.sku);
				if (oldValue) data.push({ ...oldValue });
				else {
					data.push({
						...t,
						[`skuName${index + 1}`]: item.label,
						[`skuValue${index + 1}`]: tag,
						[item.label]: tag,
						price: 0,
						stock: 0,
					})
				}

			})
		});
		return data;
	}, [[]]);
	const tableList = disposeSku(list);
	// console.log('tableList: ', tableList);
	submitList.value = tableList;
}
function disposeSku(list) {
	const result = [];
	for (let index = 0; index < list.length; index++) {
		const item = list[index];
		const sku = [];
		for (const key in item) {
			if (key.indexOf('skuValue') >= 0) sku.push(item[key])
		}
		item.sku = sku.join(',');
		result.push(item);
	}
	return result;
}
// -----------function-----------------

</script>

<style></style>
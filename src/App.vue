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
		<el-table :data="tableSkuList" border>
			<el-table-column v-for="(item, index) in specData.specList" :label="item.label" :key="index">
				<template #default="props">
					<span>{{ props.row[props.column.label] }}</span>
				</template>
			</el-table-column>
			<el-table-column v-for="(item, index) in tableHader" :label="item.label" :key="index">
				<template #default="props">
					<el-input-number v-if="item.label === '价格'" v-model="submitList[props.$index][item.key]" :precision="2"
						size="small" />
					<el-input-number v-else v-model="submitList[props.$index][item.key]" size="small" />
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
	tableSkuList = ref([]), // table显示数量和显示值
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
// watch(submitList, (newValue) => {
// 	console.log('newValue: ', newValue);
// }, { deep: true });
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
		tableSKU(specData.skuValue);
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
	tableSKU(specData.skuValue); //触发tableSKU方法 重绘数据和页面
}
function onDeletSpecValue(index, tagIndex) {//删除规格值
	specData.skuValue[specData.specList[index].label].splice(tagIndex, 1);//删除tags里面的其中一个值
	specData.specList[index].tags.splice(tagIndex, 1); // 规格值
	tableSKU(specData.skuValue);//重绘数据和页面
}
function tableSKU(skuObj) {
	let temp = [];
	for (const key in skuObj) {
		const items = skuObj[key];
		if (!temp.length) temp.push(...items.map(t => ({ [key]: t })));
		else {
			const i2 = [];
			temp.forEach(obj => {
				if (items.length === 0) i2.push(obj);
				else i2.push(...items.map(t => ({ ...obj, [key]: t })))
			});
			temp = i2;
		}
	}
	tableSkuList.value = temp;
	const tableHader = Object.keys(skuObj),
		specItems = [];
	for (let index = 0; index < temp.length; index++) {
		const el = temp[index];
		let count = 0, skuObj = {};
		for (let i = 0; i < tableHader.length; i++) {
			const hader = tableHader[i];
			if (hader) {
				count += 1;
				//防止输入的价格和库存数量丢失
				const oldSpec = submitList.value[index];
				if (oldSpec) {
					skuObj = {
						...skuObj,
						[`skuNmae${count}`]: hader,
						[`skuValue${count}`]: el[hader],
						price: oldSpec.price || 0,
						stock: oldSpec.stock || 0
					}
				} else {
					//如果没有就是正常的给普通的值
					skuObj = {
						...skuObj,
						[`skuName${count}`]: hader,
						[`skuValue${count}`]: el[hader],
						price: 0,
						stock: 0
					}
				}
			}
		}
		specItems.push(skuObj);
	}
	submitList.value = specItems; // 提交数据
	return temp;
}
// -----------function-----------------

</script>

<style></style>
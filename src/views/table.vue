<template>
	<div>
		<!-- 新的带表头的高级表格 -->
		<div class="container">

			<form class="filter-form" @submit.prevent="filterData">
			<div class="filter-group">
				<label for="roomSelect">选择房间：</label>
				<select v-model="selectedRoom" id="roomSelect">
					<option value="">所有房间</option>
					<option v-for="(room, index) in rooms" :key="index" :value="room">{{ room }}</option>
				</select>
			</div>

			<div class="filter-group">
				<label for="channelSelect">选择通道：</label>
				<select v-model="selectedChannel" id="channelSelect">
					<option value="">所有通道</option>
					<option v-for="(channel, index) in channels" :key="index" :value="channel">{{ channel }}</option>
				</select>
			</div>

			<button type="submit" class="filter-btn">筛选</button>
		</form>
			<table class="advanced-table">
				<thead>
					<tr>
						<th></th> <!-- 空的表头占位 -->
						<th v-for="(column, columnIndex) in alphabetHeaders" :key="columnIndex">{{ column }}</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="(row, rowIndex) in advancedTableData" :key="rowIndex">
						<td>{{ row[0].header }}</td> <!-- 左侧列头 -->
						<td v-for="(cell, cellIndex) in row" :key="cellIndex"
							:style="{ background: cell.background, boxShadow: '0 0 10px rgba(0, 0, 0, 0.1)' }">
							{{ cell.value }}
						</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
</template>

<script setup lang="ts" name="basetable">
import { ref, reactive } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';
import { Delete, Edit, Search, CirclePlusFilled, View } from '@element-plus/icons-vue';
import { fetchData } from '../api/index';
import TableEdit from '../components/table-edit.vue';
import TableDetail from '../components/table-detail.vue';



interface TableItem {
	id: number;
	name: string;
	thumb: string;
	money: number;
	state: string;
	date: string;
	address: string;
}

const query = reactive({
	address: '',
	name: '',
	pageIndex: 1,
	pageSize: 10
});
const tableData = ref<TableItem[]>([]);
const pageTotal = ref(0);
// 获取表格数据
const getData = async () => {
	const res = await fetchData();
	tableData.value = res.data.list;
	pageTotal.value = res.data.pageTotal || 50;
};
getData();

// 查询操作
const handleSearch = () => {
	query.pageIndex = 1;
	getData();
};
// 分页导航
const handlePageChange = (val: number) => {
	query.pageIndex = val;
	getData();
};

// 删除操作
const handleDelete = (index: number) => {
	// 二次确认删除
	ElMessageBox.confirm('确定要删除吗？', '提示', {
		type: 'warning'
	})
		.then(() => {
			ElMessage.success('删除成功');
			tableData.value.splice(index, 1);
		})
		.catch(() => { });
};

const visible = ref(false);
let idx: number = -1;
const idEdit = ref(false);
const rowData = ref({});
const handleEdit = (index: number, row: TableItem) => {
	idx = index;
	rowData.value = row;
	idEdit.value = true;
	visible.value = true;
};
const updateData = (row: TableItem) => {
	idEdit.value ? (tableData.value[idx] = row) : tableData.value.unshift(row);
	console.log(tableData.value);
	closeDialog();
};

const closeDialog = () => {
	visible.value = false;
	idEdit.value = false;
};

const visible1 = ref(false);
const handleView = (row: TableItem) => {
	rowData.value = row;
	visible1.value = true;
};


const generateAlphabetHeaders = () => {
	const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
	const headers = [];
	for (let i = 0; i < 10; i++) {
		headers.push(alphabet[i]);
	}
	return headers;
};
const alphabetHeaders = ref<string[]>(generateAlphabetHeaders());

interface Cell {
	value: string;
	background: string;
	header: string; // 列头标题
}

// 生成10x10的带表头的高级填充表格数据
const generateAdvancedTableData = (): Cell[][] => {
	const data: Cell[][] = [];
	for (let i = 0; i < 8; i++) {
		const row: Cell[] = [];
		for (let j = 0; j < 10; j++) {
			// 生成渐变背景色
			const color = Math.random() < 0.8 ? 'lightblue' : '#f1f1f1';
			row.push({ value: `${i + 1}-${j + 1}`, background: color, header: `${8 - j - i }` });
			//   row.push({ value: ``, background: color, header: `${j + 1}` });
		}
		data.push(row);
	}
	return data;
};

const advancedTableData = ref<Cell[][]>(generateAdvancedTableData());


// 模拟数据
const rooms = ref(['Room 101', 'Room 102', 'Room 103']);
const channels = ref(['Channel A', 'Channel B', 'Channel C']);

// 用户选择的房间和通道
const selectedRoom = ref('');
const selectedChannel = ref('');

// 过滤数据
const filterData = () => {
	// 根据用户选择的房间和通道，过滤数据
	// 这里使用模拟的方式，实际应用中可以根据具体的需求来实现
	const filteredData = [];
	if (selectedRoom.value && selectedChannel.value) {
		filteredData.push(`房间：${selectedRoom.value}，通道：${selectedChannel.value}`);
	} else if (selectedRoom.value) {
		filteredData.push(`房间：${selectedRoom.value}`);
	} else if (selectedChannel.value) {
		filteredData.push(`通道：${selectedChannel.value}`);
	} else {
		filteredData.push('未选择任何筛选条件');
	}
	return filteredData;
};

// 过滤后的数据
const filteredData = ref<string[]>([]);

// 监听房间和通道的变化，重新过滤数据
// watchEffect(() => {
//   filteredData.value = filterData();
// });

</script>


<style scoped>
.advanced-table {
	border-collapse: collapse;
	border-spacing: 0;
	width: 100%;
}

.advanced-table th,
.advanced-table td {
	width: 30px;
	height: 30px;
	text-align: center;
	font-size: small;
	padding: 2;
	border: 10px solid #fff;
	box-shadow: 5 5 5px rgba(0, 0, 0, 1);
}
</style>


<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.filter-form {
  margin-bottom: 20px;
  display: flex;
  align-items: center; /* 垂直居中 */
}

.filter-group {
  margin-right: 10px;
}

.filter-group:last-child {
  margin-right: 0; /* 最后一个筛选组件不添加右边距 */
}

.filter-group label {
  margin-right: 5px;
}

.filter-group select {
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.filter-btn {
  padding: 8px 16px;
  border-radius: 5px;
  background-color: #007bff;
  color: #fff;
  border: none;
  cursor: pointer;
}

.filter-btn:hover {
  background-color: #0056b3;
}



.search-input {
	width: 200px;
}

.mr10 {
	margin-right: 10px;
}

.table-td-thumb {
	display: block;
	margin: auto;
	width: 40px;
	height: 40px;
}
</style>

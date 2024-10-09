<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" type="number" />
        <q-btn v-if="editMode" color="primary" class="q-mt-md" @click="editData"
          >變更</q-btn
        >
        <q-btn v-else color="primary" class="q-mt-md" @click="addData"
          >新增</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width>
              <q-btn
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
                @click="handleClickOption(btn, props.row)"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span>無相關資料</span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { QTableProps } from 'quasar';
import { ref } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface DataItem {
  id: number;
  name: string;
  age: number | string;
}

const blockData = ref<DataItem[]>([]);
const tableConfig = ref([
  { label: '姓名', name: 'name', field: 'name', align: 'left' },
  { label: '年齡', name: 'age', field: 'age', align: 'left' },
]);
const tableButtons = ref<btnType[]>([
  { label: '編輯', icon: 'edit', status: 'edit' },
  { label: '刪除', icon: 'delete', status: 'delete' },
]);

const tempData = ref<Partial<DataItem>>({ name: '', age: '' });
let editMode = false;
let editIndex = -1;

function generateId() {
  return Date.now();
}

function resetForm() {
  tempData.value = { name: '', age: '' };
  editMode = false;
  editIndex = -1;
}

function handleClickOption(btn: btnType, data: DataItem) {
  if (btn.status === 'edit') {
    editMode = true;
    editIndex = blockData.value.findIndex((item) => item.id === data.id);
    tempData.value = { ...data };
  } else if (btn.status === 'delete') {
    deleteData(data.id);
  }
}

function editData() {
  if (editIndex !== -1 && tempData.value.name && tempData.value.age !== null) {
    blockData.value[editIndex] = {
      ...tempData.value,
      id: blockData.value[editIndex].id,
    } as DataItem;
    resetForm();
  } else {
    console.warn('無效的資料或未找到要編輯的項目');
  }
}

function deleteData(id: number) {
  const deleteIndex = blockData.value.findIndex((item) => item.id === id);
  if (deleteIndex !== -1) {
    blockData.value.splice(deleteIndex, 1);
  } else {
    console.warn('未找到要刪除的項目');
  }
}

function addData() {
  if (tempData.value.name && tempData.value.age !== null) {
    const newItem: DataItem = {
      ...tempData.value,
      id: generateId(),
    } as DataItem;
    blockData.value.push(newItem);
    resetForm();
  } else {
    alert('請填入正確資料');
  }
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>

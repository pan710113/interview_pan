<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" 
          lazy-rules
          :rules="[ val => val && val.length > 0 || 'Please type something']" />
        <q-input v-model="tempData.age" label="年齡" 
          lazy-rules
          :rules="[
            val => val !== null && val !== '' || 'Please type your age',
            val => val > 0 && val < 100 || 'Please type a real age']" />
        <q-btn color="primary" class="q-mt-md" @click="create_method">新增</q-btn>
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
              <!-- <div>{{ props.row[col.name] }}</div> -->
              <q-input v-model="props.row[col.name]" />
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
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
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>

    </div>
  </q-page>
</template>

<script setup lang="ts">
  
  import axios from 'axios';
  import { QTableProps } from 'quasar';
  import { ref } from 'vue';

  // Remove the btnType.
  // interface btnType {
  //   label: string;
  //   icon: string;
  //   status: string;
  // }

  const blockData = ref([
    // {
    //   name: 'test',
    //   age: 25,
    //   id: '1112'
    // }
  ]);
  const tableConfig = ref([
    {
      label: '姓名',
      name: 'name',
      field: 'name',
      align: 'left',
    },
    {
      label: '年齡',
      name: 'age',
      field: 'age',
      align: 'left',
    }
  ]);
  const tableButtons = ref([
    {
      label: '編輯',
      icon: 'edit',
      status: 'edit',
    },
    {
      label: '刪除',
      icon: 'delete',
      status: 'delete',
    },
  ]);

  const tempData = ref({
    name: '',
    age: '',
  });
  function handleClickOption(btn, data) {
    // ...
    // console.log(btn)
    // console.log(data)

    if(btn.status == 'delete') {
      delete_method(data.id)
    } else {
      update_method(data)
    }
  }

  async function create_method() {

    let url = 'https://dahua.metcfire.com.tw/api/CRUDTest'

    let input_dict = {}
    input_dict = {...input_dict, ...tempData.value}
    input_dict['age'] = Number(input_dict['age'])

    let result = await axios.post(url, input_dict)
    if (result.status == 200) {
      // console.log(result.data)
      // blockData.value.push(input_dict)

      let buffer_dict = await read_method()
      blockData.value.push(buffer_dict)
    }
  }

  async function read_method() {

    let url = 'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    let result = await axios(url)
    if(result.status == 200) {
      if(result.data.length > 0) {
        // console.log(result.data[result.data.length-1])
        return result.data[result.data.length-1]
      }
    }

    return {}
  }

  async function update_method(data_dict) {

    let url = 'https://dahua.metcfire.com.tw/api/CRUDTest'
    data_dict['age'] = Number(data_dict['age'])

    let result = await axios.patch(url, data_dict)
    if (result.status == 200) {
      console.log(result.data)
      // blockData.value.push(input_dict)

      // let buffer_dict = await read_method()
      // blockData.value.push(buffer_dict)
    }
  }

  async function delete_method(id) {

    // let id = "6be09cf7-ee99-4f54-861b-c53e8b7e12a0"
    let url = `https://dahua.metcfire.com.tw/api/CRUDTest/${id}`
    let result = await axios.delete(url)
    if(result.status == 200) {
      console.log(result.data)
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

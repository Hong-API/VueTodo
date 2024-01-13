<template>
  <div class="app w-full justify-center grid ">
    <div class="bg-gray-100 p-2 w-full rounded-sm grid justify-center relative">
      <div class="flex justify-end mb-1">
        <div class="background001 px-2 py-1 rounded-b-md text-white font-medium absolute top-0 left-2"><span>Todo
            List</span></div>
        <el-button type="primary"
          @click="dialoginFormVisible = true, isEdit = false, clearFields(), resetForm(ruleFormRef)"> Add New</el-button>
      </div>
      <!-- Data-table list -->
      <div class="w-full overflow-auto">
        <div class="w-[1200px]">
          <TodoListForm :todo-list="todoList" @handl-delete="handleDelete" @handle-edit="openEditForm">
          </TodoListForm>
        </div>
      </div>
    </div>
    <!-- Add/Edit form -->

    <el-dialog v-model="dialoginFormVisible">
      <div class="flex justify-center p-2 background001  text-xl font-medium relative">
        <span class="text-white">
          {{ isEdit ? "Update your task" : "Create new task" }}
        </span>
        <span v-show="isEdit" class=" absolute  right-5 bg-blac px-2 rounded-sm bg-opacity-50 capitalize"
          :class="{ 'text-green-500': form.Status === 'done', 'text-yellow-500': form.Status === 'pending', ' text-blue-300': form.Status === 'doing' }">{{
            form.Status }}
        </span>
      </div>

      <div class="p-5">
        <el-form :model="form" ref="ruleFormRef" status-icon class="demo-ruleForm">

          <el-form-item label="Title" :label-width="formLabelWidth" required prop="Title">
            <el-input v-model="form.Title" placeholder="Enter value" autocomplete="off" />
          </el-form-item>
          <el-form-item label="Description" :label-width="formLabelWidth" required prop="Description">
            <el-input v-model="form.Description" placeholder="Enter value" autocomplete="off" type="textarea" />
          </el-form-item>
          <el-form-item label="Create By" :label-width="formLabelWidth" required prop="CreatedBy">
            <el-input :disabled="isEdit" v-model="form.CreatedBy" placeholder="Enter value" autocomplete="off" />
          </el-form-item>
          <el-form-item label="Status" :label-width="formLabelWidth" required prop="Status">

            <el-select v-model="form.Status" placeholder="Status">
              <el-option v-for="status in Object.values(Status)" :key="status" :label="status" :value="status" />
            </el-select>
          </el-form-item>
          <el-form-item v-show="isEdit" label="Modified By" :label-width="formLabelWidth" :required="isEdit"
            prop="ModifiedBy">
            <el-input v-model="form.ModifiedBy" placeholder="Enter value" autocomplete="off" />
          </el-form-item>
        </el-form>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialoginFormVisible = false">Cancel</el-button>
          <el-button :loading="loading" type="primary" @click="isEdit ? updateData(ruleFormRef) : AddData(ruleFormRef)">
            {{ isEdit ? 'Update' : 'Submit' }}
          </el-button>
        </span>
      </template>
    </el-dialog>

    <!-- Delete Confirm -->
    <el-dialog v-model="confirmDeleteDialog" title="Warning" width="300px" center>
      <div class="flex justify-center w-full items-center py-4">
        <span class="text-lg">Do you want to delete? </span>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="confirmDeleteDialog = false">Cancel</el-button>
          <el-button type="primary" @click="onConfirmDelete">
            Confirm
          </el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>
<script lang="ts" setup>
import TodoListForm from '../components/TodoTableList.vue'
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios';
const dialoginFormVisible = ref(false)
import type { FormInstance, FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
const ruleFormRef = ref<FormInstance>()
const confirmDeleteDialog = ref(false)
const isEdit = ref(false)
const loading = ref(false)
const BaseULR = "http://localhost:3000"
const formLabelWidth = '100px'


enum Status {
  pending = 'pending',
  doing = 'doing',
  Done = 'done'
}
interface Todo {
  Title: string,
  Description: string,
  Status: string,
  CreatedOn?: string,
  CreatedBy: string,
  ModifiedOn: string,
  ModifiedBy?: string
}
const form = reactive({
  Title: '',
  Description: '',
  Status: '',
  CreatedBy: '',
  ModifiedBy: ''
})

const todoList = ref<Todo[]>([]);

onMounted(() => {
  getTodo();
});

const getTodo = async () => {
  await axios.get(`${BaseULR}/todo`).then((response) => {
    todoList.value = response.data
  }).catch((error) => {
    ElMessage({
      message: error.message,
      type: 'error',
    })
  })
}


const clearFields = () => { form.Title = "", form.Status = "", form.Description = "", form.CreatedBy = "", form.ModifiedBy = "" }
const AddData = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  try {
    const valid = await formEl.validate();
    if (valid) {
      loading.value = true;
      await axios.post(`${BaseULR}/todo`, form);
      ElMessage({
        message: 'Created new task successfully',
        type: 'success'
      });
      getTodo();
      dialoginFormVisible.value = false;
    }
  } catch (error) {
    console.error(error);
  } finally {
    loading.value = false;
  }
}
const requestID = ref<number>(0)
const openEditForm = async (id: number) => {
  requestID.value = id
  isEdit.value = true;
  dialoginFormVisible.value = true;
  const response = await axios.get(`${BaseULR}/todo/${id}`)
  const data = await response.data[0]
  form.Title = data.title
  form.Description = data.description
  form.Status = data.status
  form.CreatedBy = data.createdBy
  form.ModifiedBy = data.modifiedBy

}
const updateData = async (formEl: FormInstance | undefined) => {
  loading.value = true;
  try {
    if (!formEl) return;
    const valid = await formEl.validate();
    if (valid) {
      await axios.put(`${BaseULR}/todo/${requestID.value}`, form);
      ElMessage({
        message: 'Updated successfully',
        type: 'success'
      });
      getTodo()
      dialoginFormVisible.value = false;
    }
  } catch (error) {
    console.error(error);
  } finally {
    loading.value = false;
  }

}

const handleDelete = async (id: number) => {
  requestID.value = id
  confirmDeleteDialog.value = true;
}

const onConfirmDelete = async () => {
  await axios.delete(`${BaseULR}/todo/${requestID.value}`).then((response) => {
    getTodo()
    ElMessage({
      message: response.data,
      type: 'success'
    })
    confirmDeleteDialog.value = false;
  }).catch((error) => {
    console.error(error.message)
  })
}
const resetForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  formEl.resetFields()
}
</script>
<style>
@media screen and (max-width: 500px) {
  .el-dialog {
    width: 90% !important;
  }
}
</style>
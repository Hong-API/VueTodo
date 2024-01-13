<template>
    <el-table :data="props.todoList" header-row-class-name="headerColumn" >
        <el-table-column prop="title" label="Title" />
        <el-table-column prop="description" label="Description" />

        <el-table-column prop="createdBy" label="Created By" />
        <el-table-column prop="createdOn" label="Created On" >
            <template #default="scope">
                {{ dateFormat(scope.row.createdOn) }}
            </template>
        </el-table-column>
        <el-table-column prop="modifiedBy" label="Modified By" />
        <el-table-column prop="modifiedOn" label="Modified On" >
            <template #default="scope">
                {{ dateFormat(scope.row.modifiedOn) }}
            </template>
        </el-table-column>
        <el-table-column prop="status" label="Status" width="100" align="center">
            <template #default="scope">
                <div class=" uppercase rounded-sm w-[80px] bg-opacity-80 text-center text-white font-medium" :class="{
                    'bg-yellow-500': scope.row.status === 'pending',
                    'bg-green-500': scope.row.status === 'done',
                    'bg-blue-500': scope.row.status === 'doing'
                }">
                    {{ scope.row.status }}
                </div>
            </template>
        </el-table-column>
        <el-table-column fixed="right" label="Action" width="150">
            <template #default="scope">
                <el-button type="danger" size="small" @click="handlDelete(scope.row.id)">Delete</el-button>
                <el-button type="primary" size="small" @click="handleEdit(scope.row.id)">Edit</el-button>
            </template>
        </el-table-column>
    </el-table>
</template>
<script lang="ts" setup>

import { format } from 'date-fns';
const props = defineProps({
    todoList: Object,
})
const emit = defineEmits(['handleEdit', 'handlDelete'])
const handleEdit = (id: number) => emit('handleEdit', id)
const handlDelete = (id: number) => emit('handlDelete', id)

const dateFormat = (date: string) => {
    return format(date, "yyyy-MM-dd hh:mm:ss")
}
</script>

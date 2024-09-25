<script setup>
import {pageListProject, addProject, deleteProject, getProjectDetail, editProject} from "@/api/apiTest/project";
import {parseTime} from "@/utils/common.js";

const {proxy} = getCurrentInstance();

const loading = ref(false)
const total = ref(0);
const queryParams = ref({projectName: undefined, pageNum: 1, pageSize: 10})

const model_type = ref(0) // 0为新增 1为编辑
const dialogStatus = ref(false)
const form = ref({
  projectName: undefined,
  projectOwner: undefined,
  projectStatus: undefined,
  projectTester: undefined,
  projectDescribe: undefined,
  remark: undefined,
})
const formRules = ref({
  projectName: [{required: true, message: "项目名称不能为空", trigger: "blur"}],
  projectOwner: [{required: true, message: "项目负责人不能为空", trigger: "blur"}],
  projectStatus: [{required: true, message: "项目状态不能为空", trigger: "blur"}],
  projectTester: [{required: true, message: "项目测试人员不能为空", trigger: "blur"}],
  projectDescribe: [{required: false, message: "项目描述不能为空", trigger: "blur"}],
  remark: [{required: false, message: "项目备注不能为空", trigger: "blur"}],
})

function reset() {
  form.value = {
    projectName: undefined,
    projectOwner: undefined,
    projectStatus: undefined,
    projectTester: undefined,
    projectDescribe: undefined,
    remark: undefined,
  }
}

function handleClose() {
  proxy.resetForm("projectRef");
  dialogStatus.value = false
}


function handleAdd() {
  model_type.value = 0
  reset()
  dialogStatus.value = true

}

function handleEdit(row) {
  model_type.value = 1
  reset()
  dialogStatus.value = true
  getProjectDetail(row.projectId).then(
      (res) => {
        form.value = res.data
      }
  )
}

function submitForm() {
  proxy.$refs["projectRef"].validate(valid => {
    if (valid) {
      if (!form.value.projectId) {
        addProject(form.value).then(
            response => {
              if (response.code === 200) {
                handleClose()
                proxy.$modal.msgSuccess(response.msg);
                getList()
              } else {
                proxy.$modal.msgError(response.msg)
              }
            }
        )
      } else {
        editProject(form.value).then(
            res => {
              handleClose()
              proxy.$modal.msgSuccess(res.msg)
              getList()
            }
        )
      }

    }
  })

}

function cancel() {
  handleClose()
  console.log('取消');
}


function handDelete(row) {

  proxy.$modal.confirm('是否确认删除名称为 ' + row.projectName + ' 的数据项?').then(function () {
    return deleteProject(row.projectId)
  }).then(() => {
    getList()
    proxy.$modal.msgSuccess("删除成功")
  }).catch(() => {
  })
}


// 查询
function handleQuery() {
  queryParams.value.pageNum = 1
  getList()
}


function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

const projectList = ref([])

function getList() {
  loading.value = true;
  pageListProject(queryParams.value).then(res => {
    projectList.value = res.rows;
    total.value = res.total;
    loading.value = false;
  })
}


getList()
</script>

<template>
  <div class="app-container">
    <!--搜索栏-->
    <el-form :model="queryParams" ref="queryRef" :inline="true">
      <el-form-item label="项目名称" prop="projectName">
        <el-input
            v-model="queryParams.projectName"
            placeholder="请输入项目名称"
            clearable
            style="width: 200px"
        />

      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!--新增/编辑项目弹窗-->
    <el-dialog :title="model_type === 0 ? '添加项目' : '编辑项目' " v-model="dialogStatus" width="680px" append-to-body>
      <el-form ref="projectRef" :model="form" :rules="formRules" label-width="100px">
        <el-form-item label="项目名称" prop="projectName">
          <el-input placeholder="请输入项目名称" v-model="form.projectName"/>
        </el-form-item>
        <el-form-item label="负责人" prop="projectOwner">
          <el-input placeholder="请输入负责人" v-model="form.projectOwner"/>
        </el-form-item>
        <el-form-item label="测试人员" prop="projectTester">
          <el-input placeholder="请输入测试人员" v-model="form.projectTester"/>
        </el-form-item>
        <el-form-item label="状态" prop="projectStatus">
          <el-select v-model="form.projectStatus" placeholder="请选择项目状态" clearable style="width: 200px">

            <el-option value="未开始">未开始</el-option>
            <el-option value="进行中">进行中</el-option>
            <el-option value="已结束">已结束</el-option>

          </el-select>
        </el-form-item>

        <el-form-item label="描述" prop="projectDescribe">
          <el-input placeholder="请输入描述" v-model="form.projectDescribe"/>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input placeholder="请输入备注" v-model="form.remark"/>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 新增按钮-->
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
            type="primary"
            plain
            icon="Plus"
            @click="handleAdd"
        >新增
        </el-button>
      </el-col>
    </el-row>

    <!--项目管理Table-->
    <el-table v-loading="loading" :data="projectList">
      <el-table-column label="序号" type="index" align="center"></el-table-column>
      <el-table-column label="项目名称" align="center" prop="projectName"></el-table-column>
      <el-table-column label="负责人" align="center" prop="projectOwner"></el-table-column>
      <el-table-column label="测试人员" align="center" prop="projectTester"></el-table-column>
      <el-table-column label="状态" align="center" prop="projectStatus"></el-table-column>
      <el-table-column label="描述" align="center" prop="projectDescribe"></el-table-column>
      <el-table-column label="备注" align="center" prop="remark"></el-table-column>
      <el-table-column label="更新时间" align="center" prop="updateTime">
        <template #default="scope">
          <span>{{ parseTime(scope.row.updateTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新人" align="center" prop="updateBy"></el-table-column>
      <el-table-column label="创建时间" align="center" prop="createTime">
        <template #default="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>


      </el-table-column>
      <el-table-column label="创建人" align="center" prop="createBy"></el-table-column>
      <el-table-column label="操作" width="160" align="center">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleEdit(scope.row)">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total > 0" :total="total" v-model:page="queryParams.pageNum"
                v-model:limit="queryParams.pageSize"
                @pagination="getList"/>

  </div>
</template>

<style scoped lang="scss">

</style>
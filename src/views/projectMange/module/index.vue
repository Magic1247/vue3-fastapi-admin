<script setup>
import {
  listProject
} from "@/api/apiTest/project";
import {parseTime} from "@/utils/common.js";
import {addModule, deleteModule, editModule, getModuleDetail, pageListModule} from "@/api/apiTest/module.js";

const {proxy} = getCurrentInstance();

const loading = ref(false)
const total = ref(0);
const queryParams = ref({moduleName: undefined, subProjectId: undefined, pageNum: 1, pageSize: 10})

const model_type = ref(0) // 0为新增 1为编辑
const dialogStatus = ref(false)
const form = ref({
  moduleName: undefined,
  moduleTester: undefined,
  moduleDescribe: undefined,
  subProjectId: undefined,
  remark: undefined,
})
const formRules = ref({
  moduleName: [{required: true, message: "模块名称不能为空", trigger: "blur"}],
  moduleTester: [{required: true, message: "模块测试人员不能为空", trigger: "blur"}],
  moduleDescribe: [{required: false, message: "模块描述不能为空", trigger: "blur"}],
  subProjectId: [{required: true, message: '所属项目不能为空', trigger: 'blur'}],
  remark: [{required: false, message: "模块备注不能为空", trigger: "blur"}],
})

const subProjectList = ref([])

function getSubProjectList() {
  listProject().then(res => {
    subProjectList.value = res.data
  })
}


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
  getModuleDetail(row.moduleId).then(
      (res) => {
        form.value = res.data
      }
  )
}

function submitForm() {
  proxy.$refs["moduleFormRef"].validate(valid => {
    if (valid) {
      if (!form.value.moduleId) {
        addModule(form.value).then(
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
        console.log(form.value)
        editModule(form.value).then(
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
    return deleteModule(row.moduleId)
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
  queryParams.value = {moduleName: undefined, subProjectId: undefined, pageNum: 1, pageSize: 10}
  proxy.resetForm("ModuleQueryRef");
  handleQuery();
}

const moduleList = ref([])

function getList() {
  loading.value = true;
  pageListModule(queryParams.value).then(res => {
    moduleList.value = res.rows;
    total.value = res.total;
    loading.value = false;
  })
}


getList()
getSubProjectList()

</script>

<template>
  <div class="app-container">
    <!--搜索栏-->
    <el-form :model="queryParams" ref="ModuleQueryRef" :inline="true">
      <el-form-item label="模块名称" prop="projectName">
        <el-input
            v-model="queryParams.moduleName"
            placeholder="请输入模块名称"
            clearable
            style="width: 200px"
        />
      </el-form-item>

      <el-form-item label="所属项目" prop="projectName">

        <el-select v-model="queryParams.subProjectId"
                   placeholder="所属项目"
                   clearable
                   style="width: 240px"
        >
          <el-option v-for="project in subProjectList"
                     :key="project.projectId"
                     :label="project.projectName"
                     :value="project.projectId"
          />
        </el-select>

      </el-form-item>

      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!--新增/编辑模块弹窗-->
    <el-dialog :title="model_type === 0 ? '添加模块' : '编辑模块' " v-model="dialogStatus" width="680px" append-to-body>
      <el-form ref="moduleFormRef" :model="form" :rules="formRules" label-width="100px">
        <el-form-item label="模块名称" prop="moduleName">
          <el-input placeholder="请输入模块名称" v-model="form.moduleName"/>
        </el-form-item>
        <el-form-item label="测试人员" prop="moduleTester">
          <el-input placeholder="请输入测试人员" v-model="form.moduleTester"/>
        </el-form-item>

        <el-form-item label="所属项目" prop="subProjectId">
          <el-select v-model="form.subProjectId"
                     placeholder="所属项目"
                     clearable
                     style="width: 240px"
          >
            <el-option v-for="project in subProjectList"
                       :key="project.projectId"
                       :label="project.projectName"
                       :value="project.projectId"
            />
          </el-select>

        </el-form-item>


        <el-form-item label="描述" prop="moduleDescribe">
          <el-input placeholder="请输入描述" v-model="form.moduleDescribe"/>
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

    <!--模块管理Table-->
    <el-table v-loading="loading" :data="moduleList">
      <el-table-column label="序号" type="index" align="center"></el-table-column>
      <el-table-column label="模块名称" align="center" prop="moduleName"></el-table-column>
      <el-table-column label="所属项目" align="center" prop="sub_project"></el-table-column>
      <el-table-column label="测试人员" align="center" prop="moduleTester"></el-table-column>
      <el-table-column label="用例数" align="center" prop="caseNum">
        <template #default="scope">
          {{ scope.row.caseNum || 0 }}
        </template>
      </el-table-column>
      <el-table-column label="描述" align="center" prop="moduleDescribe"></el-table-column>
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
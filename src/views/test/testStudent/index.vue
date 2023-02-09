<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="学生名称" prop="studentName">
        <el-input
          v-model="queryParams.studentName"
          placeholder="请输入学生名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="年龄" prop="studentAge">
        <el-input
          v-model="queryParams.studentAge"
          placeholder="请输入年龄"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="爱好" prop="studentHobby">
        <el-input
          v-model="queryParams.studentHobby"
          placeholder="请输入爱好"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="性别" prop="studentSex">
        <el-select v-model="queryParams.studentSex" placeholder="请选择性别" clearable>
          <el-option
            v-for="dict in sys_user_sex"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="状态" prop="studentStatus">
        <el-select v-model="queryParams.studentStatus" placeholder="请选择状态" clearable>
          <el-option
            v-for="dict in sys_normal_disable"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="生日" style="width: 308px">
        <el-date-picker
          v-model="daterangeStudentBirthday"
          value-format="YYYY-MM-DD"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['test:testStudent:save']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['test:testStudent:update']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['test:testStudent:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['test:testStudent:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="testStudentList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="编号" align="center" prop="id" />
      <el-table-column label="学生名称" align="center" prop="studentName" />
      <el-table-column label="年龄" align="center" prop="studentAge" />
      <el-table-column label="爱好" align="center" prop="studentHobby" />
      <el-table-column label="性别" align="center" prop="studentSex">
        <template #default="scope">
          <dict-tag :options="sys_user_sex" :value="scope.row.studentSex"/>
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center" prop="studentStatus">
        <template #default="scope">
          <dict-tag :options="sys_normal_disable" :value="scope.row.studentStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="生日" align="center" prop="studentBirthday" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.studentBirthday, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
          <template #default="scope">
              <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{m}:{s}') }}</span>
          </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['test:testStudent:update']">修改</el-button>
          <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['test:testStudent:remove']">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改学生信息对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="testStudentRef" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="学生名称" prop="studentName">
            <el-input v-model="form.studentName" placeholder="请输入学生名称" />
        </el-form-item>
        <el-form-item label="年龄" prop="studentAge">
            <el-input-number v-model="form.studentAge" controls-position="right"></el-input-number>
        </el-form-item>
        <el-form-item label="爱好" prop="studentHobby">
            <el-input-number v-model="form.studentHobby" controls-position="right"></el-input-number>
        </el-form-item>
        <el-form-item label="性别" prop="studentSex">
          <el-select v-model="form.studentSex" placeholder="请选择性别">
            <el-option
              v-for="dict in sys_user_sex"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="状态" prop="studentStatus">
          <el-select v-model="form.studentStatus" placeholder="请选择状态">
            <el-option
              v-for="dict in sys_normal_disable"
              :key="dict.value"
              :label="dict.label"
              :value="parseInt(dict.value)"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="生日" prop="studentBirthday">
          <el-date-picker clearable
            v-model="form.studentBirthday"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="请选择生日">
          </el-date-picker>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup name="TestStudent">
import { listTestStudent, getTestStudent, delTestStudent, addTestStudent, updateTestStudent } from "@/api/test/testStudent";

const { proxy } = getCurrentInstance();
const { sys_user_sex, sys_normal_disable } = proxy.useDict('sys_user_sex', 'sys_normal_disable');

const testStudentList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");
const daterangeStudentBirthday = ref([]);
const daterangeCreateTime = ref([]);

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    studentName: null,
    studentAge: null,
    studentHobby: null,
    studentSex: null,
    studentStatus: null,
    studentBirthday: null,
  },
  rules: {
    studentName: [
      { required: true, message: "学生名称不能为空", trigger: "blur" }
    ],
    studentSex: [
      { required: true, message: "性别不能为空", trigger: "change" }
    ],
    studentStatus: [
      { required: true, message: "状态不能为空", trigger: "change" }
    ],
    createTime: [
      { required: true, message: "创建时间不能为空", trigger: "blur" }
    ],
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询学生信息列表 */
function getList() {
  loading.value = true;
  if (null != daterangeStudentBirthday.value && '' != daterangeStudentBirthday.value) {
    queryParams.value["beginStudentBirthday"] = daterangeStudentBirthday.value[0];
    queryParams.value["endStudentBirthday"] = daterangeStudentBirthday.value[1];
  }
  else {
    queryParams.value["beginStudentBirthday"] = null;
    queryParams.value["endStudentBirthday"] = null;
  }
  if (null != daterangeCreateTime.value && '' != daterangeCreateTime.value) {
    queryParams.value["beginCreateTime"] = daterangeCreateTime.value[0];
    queryParams.value["endCreateTime"] = daterangeCreateTime.value[1];
  }
  else {
    queryParams.value["beginCreateTime"] = null;
    queryParams.value["endCreateTime"] = null;
  }
  listTestStudent(queryParams.value).then(response => {
    testStudentList.value = response.data.records;
    total.value = response.data.total;
    loading.value = false;
  });
}

// 取消按钮
function cancel() {
  open.value = false;
  reset();
}

// 表单重置
function reset() {
  form.value = {
    id: null,
    studentName: null,
    studentAge: null,
    studentHobby: null,
    studentSex: null,
    studentStatus: null,
    studentBirthday: null,
    createTime: null,
    updateTime: null,
    deleteTime: null,
    createBy: null,
    updateBy: null
  };
  proxy.resetForm("testStudentRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  daterangeStudentBirthday.value = [];
  daterangeCreateTime.value = [];
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加学生信息";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getTestStudent(_id).then(response => {
    form.value = response.data;
    open.value = true;
    title.value = "修改学生信息";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["testStudentRef"].validate(valid => {
    if (valid) {
      if (form.value.id != null) {
        updateTestStudent(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addTestStudent(form.value).then(response => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal.confirm('是否确认删除学生信息编号为"' + _ids + '"的数据项？').then(function() {
    return delTestStudent(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('test/testStudent/export', {
    ...queryParams.value
  }, `testStudent_${new Date().getTime()}.xlsx`)
}

getList();
</script>

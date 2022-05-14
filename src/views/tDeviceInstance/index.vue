<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">产品id</label>
        <el-input v-model="query.productId" clearable placeholder="产品id" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">设备类型,1：直连设备；2：网关；3：网关子设备</label>
        <el-input v-model="query.deviceType" clearable placeholder="设备类型,1：直连设备；2：网关；3：网关子设备" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">状态：0：离线，1：在线</label>
        <el-input v-model="query.state" clearable placeholder="状态：0：离线，1：在线" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="产品id" prop="productId">
            <el-input v-model="form.productId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="设备实例名称" prop="name">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="其他配置">
            <el-input v-model="form.configuration" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="父级设备ID">
            <el-input v-model="form.parentId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="设备类型,1：直连设备；2：网关；3：网关子设备" prop="deviceType">
            <el-input v-model="form.deviceType" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="状态：0：离线，1：在线" prop="state">
            <el-input v-model="form.state" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="说明">
            <el-input v-model="form.description" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="产品Key" prop="productKey">
            <el-input v-model="form.productKey" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="设备名称" prop="deviceName">
            <el-input v-model="form.deviceName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="设备名称" prop="deviceSecret">
            <el-input v-model="form.deviceSecret" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否激活,0:未激活；1：已激活" prop="active">
            <el-input v-model="form.active" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="id" />
        <el-table-column prop="productId" label="产品id" />
        <el-table-column prop="name" label="设备实例名称" />
        <el-table-column prop="configuration" label="其他配置" />
        <el-table-column prop="registryTime" label="激活时间" />
        <el-table-column prop="parentId" label="父级设备ID" />
        <el-table-column prop="deviceType" label="设备类型,1：直连设备；2：网关；3：网关子设备">
          <template slot-scope="scope">
            {{ dict.label.device_type[scope.row.deviceType] }}
          </template>
        </el-table-column>
        <el-table-column prop="state" label="状态：0：离线，1：在线">
          <template slot-scope="scope">
            {{ dict.label.device_status[scope.row.state] }}
          </template>
        </el-table-column>
        <el-table-column prop="description" label="说明" />
        <el-table-column prop="createTime" label="createTime">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="productKey" label="产品Key" />
        <el-table-column prop="deviceName" label="设备名称" />
        <el-table-column prop="deviceSecret" label="设备名称" />
        <el-table-column prop="active" label="是否激活,0:未激活；1：已激活">
          <template slot-scope="scope">
            {{ dict.label.active_status[scope.row.active] }}
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','tDeviceInstance:edit','tDeviceInstance:del']" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudTDeviceInstance from '@/api/tDeviceInstance'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, productId: null, name: null, configuration: null, creatorId: null, registryTime: null, parentId: null, deviceType: null, state: null, description: null, createTime: null, productKey: null, deviceName: null, deviceSecret: null, active: null }
export default {
  name: 'TDeviceInstance',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['device_type', 'device_status', 'active_status'],
  cruds() {
    return CRUD({ title: 'asd', url: 'api/tDeviceInstance', idField: 'id', sort: 'id,desc', crudMethod: { ...crudTDeviceInstance }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'tDeviceInstance:add'],
        edit: ['admin', 'tDeviceInstance:edit'],
        del: ['admin', 'tDeviceInstance:del']
      },
      rules: {
        productId: [
          { required: true, message: '产品id不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '设备实例名称不能为空', trigger: 'blur' }
        ],
        deviceType: [
          { required: true, message: '设备类型,1：直连设备；2：网关；3：网关子设备不能为空', trigger: 'blur' }
        ],
        state: [
          { required: true, message: '状态：0：离线，1：在线不能为空', trigger: 'blur' }
        ],
        productKey: [
          { required: true, message: '产品Key不能为空', trigger: 'blur' }
        ],
        deviceName: [
          { required: true, message: '设备名称不能为空', trigger: 'blur' }
        ],
        deviceSecret: [
          { required: true, message: '设备名称不能为空', trigger: 'blur' }
        ],
        active: [
          { required: true, message: '是否激活,0:未激活；1：已激活不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'productId', display_name: '产品id' },
        { key: 'deviceType', display_name: '设备类型,1：直连设备；2：网关；3：网关子设备' },
        { key: 'state', display_name: '状态：0：离线，1：在线' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>

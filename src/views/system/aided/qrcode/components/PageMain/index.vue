<template>
  <div class="cs-p">
    <el-form
      :inline="true"
      size="small">
      <el-form-item v-if="auth.add">
        <el-button
          icon="el-icon-plus"
          :disabled="loading"
          @click="handleCreate">新增二维码</el-button>
      </el-form-item>

      <el-form-item v-if="auth.del">
        <el-button
          icon="el-icon-delete"
          :disabled="loading"
          @click="handleDelete(null)">删除</el-button>
      </el-form-item>

      <cs-help
        :router="$route.path"
        style="padding-bottom: 19px;">
      </cs-help>
    </el-form>

    <el-table
      v-loading="loading"
      :data="currentTableData"
      :highlight-current-row="true"
      @selection-change="handleSelectionChange"
      @sort-change="sortChange">
      <el-table-column type="selection" width="30"/>

      <el-table-column
        label="名称"
        prop="name"
        min-width="180">
      </el-table-column>

      <el-table-column
        label="内容"
        prop="text"
        min-width="220"
        :show-overflow-tooltip="true">
      </el-table-column>

      <el-table-column
        label="Logo"
        prop="logo"
        align="center">
        <template slot-scope="scope">
          <el-popover
            v-if="scope.row.logo"
            width="150"
            placement="top"
            trigger="hover">
            <div class="popover-image">
              <el-image
                :src="scope.row.logo"
                @click.native="$preview(scope.row.logo)"
                fit="fill"/>
            </div>
            <i slot="reference" class="el-icon-picture"/>
          </el-popover>
        </template>
      </el-table-column>

      <el-table-column
        label="尺寸"
        prop="size"
        sortable="custom"
        width="75">
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        min-width="100">
        <template slot-scope="scope">
          <el-button
            v-if="auth.view"
            @click="handleView(scope.$index)"
            size="small"
            type="text">详细</el-button>

          <el-button
            v-if="auth.set"
            @click="handleUpdate(scope.$index)"
            size="small"
            type="text">编辑</el-button>

          <el-button
            v-if="auth.del"
            @click="handleDelete(scope.$index)"
            size="small"
            type="text">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
      :append-to-body="true"
      :close-on-click-modal="false"
      width="600px">
      <el-form
        :model="form"
        :rules="rules"
        ref="form"
        label-width="80px">
        <el-form-item
          label="名称"
          prop="name">
          <el-input
            v-model="form.name"
            placeholder="请输入二维码名称"
            :clearable="true"/>
        </el-form-item>

        <el-form-item
          label="内容"
          prop="text">
          <el-input
            v-model="form.text"
            placeholder="请输入二维码内容"
            type="textarea"
            :rows="3"/>
        </el-form-item>

        <el-form-item
          label="Logo"
          prop="logo">
          <el-input
            v-model="form.logo"
            placeholder="可输入二维码Logo链接"
            :clearable="true">
            <el-dropdown
              slot="append"
              :show-timeout="50"
              @command="handleCommand">
              <el-button icon="el-icon-upload"/>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item command="storage" icon="el-icon-finished">资源选择</el-dropdown-item>
                <el-dropdown-item command="upload" icon="el-icon-upload2">上传资源</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </el-input>
        </el-form-item>

        <el-form-item
          label="尺寸"
          prop="size">
          <el-input-number
            v-model="form.size"
            controls-position="right"
            :min="1"
            :max="10"
            style="width: 120px;"/>
        </el-form-item>
      </el-form>

      <el-divider>效果预览</el-divider>
      <div v-if="form.text" class="cs-tc">
        <el-image v-if="qrcodeImage" fit="fill" :src="qrcodeImage"/>
      </div>

      <div slot="footer" class="dialog-footer">
        <el-button
          @click="dialogFormVisible = false"
          size="small">取消</el-button>

        <el-button
          v-if="dialogStatus === 'create'"
          type="primary"
          :loading="dialogLoading"
          @click="create"
          size="small">确定</el-button>

        <el-button
          v-else type="primary"
          :loading="dialogLoading"
          @click="update"
          size="small">修改</el-button>
      </div>

      <cs-storage
        ref="storage"
        style="display: none"
        :limit="1"
        @confirm="_getStorageFileList">
      </cs-storage>

      <cs-upload
        style="display: none"
        ref="upload"
        type="slot"
        accept="image/*"
        :limit="1"
        :multiple="false"
        @confirm="_getUploadFileList">
      </cs-upload>
    </el-dialog>

    <el-dialog
      title="二维码预览"
      :visible.sync="dialogQrcodeVisible"
      :append-to-body="true"
      :close-on-click-modal="false"
      width="600px">
      <el-row :gutter="20">
        <el-col :span="4">
          <p>效果预览</p>
        </el-col>
        <el-col :span="20">
          <el-image v-if="qrcodeData.url" fit="fill" :src="qrcodeData.url"/>
        </el-col>
      </el-row>

      <el-row :gutter="20">
        <el-col :span="4">
          <p>名称</p>
        </el-col>
        <el-col :span="20">
          <p>{{qrcodeData.name}}</p>
        </el-col>
      </el-row>

      <el-row :gutter="20">
        <el-col :span="4">
          <p>实际内容</p>
        </el-col>
        <el-col :span="20">
          <p>{{qrcodeData.text}}</p>
        </el-col>
      </el-row>

      <el-row :gutter="20">
        <el-col :span="4">
          <p>调用地址</p>
        </el-col>
        <el-col :span="20">
          <p>{{qrcodeData.url}}</p>
        </el-col>
      </el-row>

      <div slot="footer" class="dialog-footer">
        <el-button
          @click="handleCopy"
          size="small">复制调用地址</el-button>

        <el-button
          type="primary"
          @click="dialogQrcodeVisible = false"
          size="small">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  getQrcodeCallurl,
  delQrcodeList,
  addQrcodeItem,
  setQrcodeItem
} from '@/api/aided/qrcode'
import { debounce } from 'lodash'
import * as clipboard from 'clipboard-polyfill'
import util from '@/utils/util'

export default {
  components: {
    'csUpload': () => import('@/components/cs-upload'),
    'csStorage': () => import('@/components/cs-storage')
  },
  props: {
    loading: {
      default: false
    },
    tableData: {
      default: () => []
    }
  },
  data() {
    return {
      qrcodeUrl: '',
      qrcodeImage: '',
      currentTableData: [],
      multipleSelection: [],
      auth: {
        add: false,
        set: false,
        del: false,
        view: false
      },
      dialogLoading: false,
      dialogFormVisible: false,
      dialogStatus: '',
      dialogQrcodeVisible: false,
      qrcodeData: {
        name: '',
        text: '',
        url: ''
      },
      textMap: {
        update: '编辑二维码',
        create: '新增二维码'
      },
      form: {
        name: undefined,
        text: undefined,
        size: undefined,
        logo: undefined
      },
      rules: {
        name: [
          {
            required: true,
            message: '名称不能为空',
            trigger: 'blur'
          },
          {
            max: 50,
            message: '长度不能大于 50 个字符',
            trigger: 'blur'
          }
        ],
        text: [
          {
            required: true,
            message: '内容不能为空',
            trigger: 'blur'
          },
          {
            max: 255,
            message: '内容不能大于 255 个字符',
            trigger: 'blur'
          }
        ],
        logo: [
          {
            max: 512,
            message: '长度不能大于 512 个字符',
            trigger: 'blur'
          }
        ],
        size: [
          {
            type: 'number',
            message: '必须为数字值',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  computed: {
    changeQrcode() {
      const { text, size, logo } = this.form
      return {
        text,
        size,
        logo
      }
    }
  },
  watch: {
    tableData: {
      handler(val) {
        this.currentTableData = val
      },
      immediate: true
    },
    changeQrcode: {
      handler() {
        this.$nextTick(() => {
          this._getQrcodeImage()
        })
      }
    },
    dialogQrcodeVisible: {
      handler(val) {
        if (val === false) {
          this.qrcodeData.url = ''
        }
      }
    }
  },
  mounted() {
    this._validationAuth()
    getQrcodeCallurl()
      .then(res => {
        this.qrcodeUrl = res.data['call_url']
      })
  },
  methods: {
    // 验证权限
    _validationAuth() {
      this.auth.add = this.$has('/system/aided/qrcode/add')
      this.auth.set = this.$has('/system/aided/qrcode/set')
      this.auth.del = this.$has('/system/aided/qrcode/del')
      this.auth.view = this.$has('/system/aided/qrcode/view')
    },
    // 获取列表中的编号
    _getIdList(val) {
      if (val === null) {
        val = this.multipleSelection
      }

      let idList = []
      if (Array.isArray(val)) {
        val.forEach(value => {
          idList.push(value.qrcode_id)
        })
      } else {
        idList.push(this.currentTableData[val].qrcode_id)
      }

      return idList
    },
    // 资源下拉框事件
    handleCommand(command) {
      switch (command) {
        case 'storage':
          this.$refs.storage.handleStorageDlg([0, 2])
          break

        case 'upload':
          this.$refs.upload.handleUploadDlg()
          break
      }
    },
    // 获取二维码预览
    _getQrcodeImage: debounce(function() {
      if (!this.form.text) {
        return
      }

      let parm = `?text=${this.form.text}&size=${this.form.size}&logo=${this.form.logo}`
      this.qrcodeImage = this.qrcodeUrl + encodeURI(parm)
    }, 500),
    // 获取上传资源
    _getUploadFileList(files) {
      if (!files.length) {
        return
      }

      const response = files[0].response
      if (!response || response.status !== 200) {
        return
      }

      if (response.data[0].type === 0) {
        this.form.logo = util.checkUrl(response.data[0].url)
      }
    },
    // 获取选择资源
    _getStorageFileList(files) {
      if (!files.length) {
        return
      }

      for (const value of files) {
        if (value.type === 0) {
          this.form.logo = util.checkUrl(value.url)
          break
        }
      }
    },
    // 选中数据项
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    // 获取排序字段
    sortChange({ column, prop, order }) {
      let sort = {
        order_type: undefined,
        order_field: undefined
      }

      if (column && order) {
        sort.order_type = order === 'ascending' ? 'asc' : 'desc'
        sort.order_field = prop
      }

      this.$emit('sort', sort)
    },
    // 弹出新建对话框
    handleCreate() {
      this.qrcodeImage = ''
      this.form = {
        name: '',
        text: '',
        size: 3,
        logo: ''
      }

      this.$nextTick(() => {
        this.$refs.form.clearValidate()
      })

      this.dialogStatus = 'create'
      this.dialogLoading = false
      this.dialogFormVisible = true
    },
    // 请求创建
    create() {
      this.$refs.form.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          addQrcodeItem({ ...this.form })
            .then(res => {
              this.currentTableData.unshift(res.data)
              this.dialogFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    },
    // 批量删除
    handleDelete(val) {
      let qrcode_id = this._getIdList(val)
      if (qrcode_id.length === 0) {
        this.$message.error('请选择要操作的数据')
        return
      }

      this.$confirm('确定要执行该操作吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        closeOnClickModal: false
      })
        .then(() => {
          delQrcodeList(qrcode_id)
            .then(() => {
              for (let i = this.currentTableData.length - 1; i >= 0; i--) {
                if (qrcode_id.indexOf(this.currentTableData[i].qrcode_id) !== -1) {
                  this.currentTableData.splice(i, 1)
                }
              }

              if (this.currentTableData.length <= 0) {
                this.$emit('refresh', true)
              }

              this.$message.success('操作成功')
            })
        })
        .catch(() => {
        })
    },
    // 编辑二维码
    handleUpdate(index) {
      this.qrcodeImage = ''
      this.currentIndex = index
      this.form = { ...this.currentTableData[index] }

      this.$nextTick(() => {
        this.$refs.form.clearValidate()
      })

      this.dialogStatus = 'update'
      this.dialogLoading = false
      this.dialogFormVisible = true
    },
    // 请求编辑
    update() {
      this.$refs.form.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          setQrcodeItem({ ...this.form })
            .then(res => {
              this.$set(
                this.currentTableData,
                this.currentIndex,
                {
                  ...this.currentTableData[this.currentIndex],
                  ...res.data
                })

              this.dialogFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    },
    // 查看预览
    handleView(index) {
      const data = this.currentTableData[index]
      this.qrcodeData = {
        name: data.name,
        text: data.text,
        url: this.qrcodeUrl + `?qrcode_id=${data.qrcode_id}`
      }

      this.dialogQrcodeVisible = true
    },
    // 复制调用地址
    handleCopy() {
      clipboard.writeText(this.qrcodeData.url)
        .then(() => {
          this.$message.success('已复制调用地址到剪贴板')
        })
        .catch(err => {
          this.$message.error(err)
        })
    }
  }
}
</script>

<style scoped>
  .popover-image {
    text-align: center;
    line-height: 0;
  }
  .popover-image >>> img {
    vertical-align: middle;
    cursor: pointer;
  }
  .el-image >>> .el-image__error {
    line-height: 1.4;
  }
</style>

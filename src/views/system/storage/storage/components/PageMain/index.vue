<template>
  <div class="cs-p">
    <el-form
      :inline="true"
      size="small">

      <el-form-item>
        <el-button-group>
          <el-button
            v-if="auth.add"
            icon="el-icon-plus"
            :disabled="loading"
            @click="handleCreate">新增目录</el-button>

          <el-button
            v-if="auth.upload"
            icon="el-icon-upload2"
            :disabled="loading"
            @click="handleUpload">上传资源</el-button>
        </el-button-group>
      </el-form-item>

      <el-form-item>
        <el-button-group>
          <el-tooltip content="勾选当前页全部资源" placement="top">
            <el-button
              icon="el-icon-plus"
              :disabled="loading"
              @click="allCheckBox">全选</el-button>
          </el-tooltip>

          <el-tooltip content="反向勾选当前页资源" placement="top">
            <el-button
              icon="el-icon-minus"
              :disabled="loading"
              @click="reverseCheckBox">反选</el-button>
          </el-tooltip>

          <el-tooltip content="取消当前页勾选" placement="top">
            <el-button
              icon="el-icon-close"
              :disabled="loading"
              @click="cancelCheckBox">取消</el-button>
          </el-tooltip>

          <el-tooltip content="清除所有已选中勾选" placement="top">
            <el-button
              icon="el-icon-refresh"
              :disabled="loading"
              @click="checkList = []">清除</el-button>
          </el-tooltip>
        </el-button-group>
      </el-form-item>

      <el-form-item>
        <el-button-group>
          <el-button
            v-if="auth.move"
            icon="el-icon-rank"
            :disabled="loading"
            @click="handleMove(null)">移动</el-button>

          <el-button
            v-if="auth.del"
            icon="el-icon-delete"
            :disabled="loading"
            @click="handleDelete(null)">删除</el-button>
        </el-button-group>
      </el-form-item>

      <cs-help
        :router="$route.path"
        style="padding-bottom: 19px;">
      </cs-help>
    </el-form>

    <el-breadcrumb class="breadcrumb cs-mb" separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>
        <a class="cs-cp" @click="switchFolder(0)">资源管理</a>
      </el-breadcrumb-item>

      <el-breadcrumb-item
        v-for="item in naviData"
        :key="item.storage_id">
        <a class="cs-cp" @click="switchFolder(item.storage_id)">{{item.name}}</a>
      </el-breadcrumb-item>
    </el-breadcrumb>

    <el-checkbox-group v-model="checkList">
      <ul class="storage-list" v-loading="loading">
        <li v-for="(item, index) in currentTableData" :key="index">
          <dl>
            <dt>
              <div class="picture cs-m-10">
                <el-checkbox :label="item.storage_id" class="storage-check"/>
                <el-image fit="fill" :src="item | getImageThumb" @click.native="openStorage(index)" lazy/>
              </div>
              <span class="storage-name cs-ml-10" :title="item.name">{{item.name}}</span>
              <el-dropdown
                placement="bottom"
                :show-timeout="50"
                size="small"
                @command="command => handleControlItemClick(command)">
                <i class="el-icon-more more"/>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item
                    v-if="auth.rename"
                    icon="el-icon-edit"
                    :command="{type: 'rename', index}">重命名</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type !== 2 && auth.replace"
                    icon="el-icon-upload"
                    :command="{type: 'replace', index}"
                    divided>替换上传</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type === 0 && item.parent_id && auth.cover"
                    icon="el-icon-picture"
                    :command="{type: 'cover', index}">设为封面</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type === 3 && !item.cover && auth.cover"
                    icon="el-icon-picture"
                    :command="{type: 'video_cover', index}">选择海报</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type === 2 && auth.default"
                    icon="el-icon-folder-checked"
                    :command="{type: 'default', index}">{{item.is_default ? '取消默认' : '设为默认'}}</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.cover && auth.clear_cover"
                    icon="el-icon-picture"
                    :command="{type: 'clear_cover', index}"
                    divided>{{item.type === 2 ? '取消封面' : '取消海报'}}</el-dropdown-item>

                  <el-dropdown-item
                    v-if="auth.move"
                    icon="el-icon-rank"
                    :command="{type: 'move', storage_id: item.storage_id}"
                    divided>转移目录</el-dropdown-item>

                  <el-dropdown-item
                    v-if="auth.del"
                    icon="el-icon-delete"
                    :command="{type: 'delete', storage_id: item.storage_id}">删除资源</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type === 0 && auth.refresh"
                    icon="el-icon-refresh"
                    :command="{type: 'refresh', index}">清除缓存</el-dropdown-item>

                  <el-dropdown-item
                    v-if="item.type !== 2 && auth.link"
                    icon="el-icon-link"
                    :command="{type: 'link', index}"
                    divided>复制外链</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </dt>
            <dd class="cs-ml-10">
              <p>
                <span>日期：{{item['create_time']}}</span>
              </p>
              <p>
                <span v-if="item.type === 0">尺寸：{{`${item.pixel['width']},${item.pixel['height']}`}}</span>
                <span v-else>类型：<i :class="item.type | getFileTypeIocn"/></span>
              </p>
            </dd>
          </dl>
        </li>
      </ul>
    </el-checkbox-group>

    <cs-upload
      style="display: none"
      ref="upload"
      type="slot"
      :upload-tip="uploadConfig.uploadTip"
      :multiple="uploadConfig.multiple"
      :accept="uploadConfig.accept"
      :limit="uploadConfig.limit"
      :storage-id="storageId"
      @confirm="_getUploadFileList">
    </cs-upload>

    <cs-storage
      style="display: none"
      ref="storage"
      :limit="1"
      @confirm="handleVideoCover">
    </cs-storage>

    <el-dialog
      :title="nameMap[nameStatus]"
      :visible.sync="nameFormVisible"
      :append-to-body="true"
      :close-on-click-modal="false"
      width="600px">
      <el-form
        :model="nameForm"
        :rules="rules"
        ref="name"
        label-width="50px"
        label-position="left"
        @submit.native.prevent>
        <el-form-item
          label="名称"
          prop="name">
          <el-input
            v-model="nameForm.name"
            placeholder="请输入目录名称"
            @keyup.enter.native="nameStatus === 'create' ? create() : rename()"
            ref="input"/>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button
          @click="nameFormVisible = false"
          size="small">取消</el-button>

        <el-button
          v-if="nameStatus === 'create'"
          type="primary"
          :loading="dialogLoading"
          @click="create"
          size="small">确定</el-button>

        <el-button
          v-else type="primary"
          :loading="dialogLoading"
          @click="rename"
          size="small">修改</el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="移动资源"
      :visible.sync="moveFormVisible"
      :append-to-body="true"
      :close-on-click-modal="false"
      width="600px">

      <el-tree
        style="margin-top: -25px;"
        node-key="storage_id"
        :data="directoryList"
        :props="directoryProps"
        :highlight-current="true"
        :default-expand-all="true"
        ref="directory">
        <span slot-scope="{node, data}">
          <span class="brother-showing">
            <i v-if="data.children" :class="`el-icon-${node.expanded ? 'folder-opened' : 'folder'}`"/>
            <i v-else class="el-icon-folder"/>
            {{node.label}}
          </span>
        </span>
      </el-tree>

      <div slot="footer" class="dialog-footer">
        <el-button
          @click="moveFormVisible = false"
          size="small">取消</el-button>

        <el-button
          type="primary"
          :loading="dialogLoading"
          @click="moveStorage"
          size="small">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import util from '@/utils/util'
import storage from '../mixins'
import * as clipboard from 'clipboard-polyfill'
import {
  delStorageList,
  addStorageDirectoryItem,
  getStorageDirectorySelect,
  moveStorageList,
  renameStorageItem,
  setStorageDirectoryDefault,
  clearStorageThumb,
  setStorageCover,
  clearStorageCover
} from '@/api/upload/storage'

export default {
  mixins: [storage],
  components: {
    'csUpload': () => import('@/components/cs-upload'),
    'csStorage': () => import('@/components/cs-storage')
  },
  props: {
    tableData: {
      default: () => []
    },
    naviData: {
      default: () => []
    },
    loading: {
      default: false
    },
    storageId: {
      default: 0
    }
  },
  data() {
    return {
      uploadConfig: {},
      currentTableData: [],
      checkList: [],
      dialogLoading: false,
      nameForm: {},
      nameFormVisible: false,
      nameStatus: '',
      nameMap: {
        update: '重命名',
        create: '新增目录'
      },
      moveFormVisible: false,
      moveIdList: [],
      directoryList: [],
      directoryProps: {
        label: 'name',
        children: 'children'
      },
      videoCover: 0,
      auth: {
        add: false,
        upload: false,
        rename: false,
        replace: false,
        cover: false,
        clear_cover: false,
        default: false,
        move: false,
        del: false,
        refresh: false,
        link: false
      },
      rules: {
        name: [
          {
            required: true,
            message: '目录名称不能为空',
            trigger: 'blur'
          },
          {
            max: 255,
            message: '长度不能大于 255 个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  watch: {
    tableData: {
      handler(val) {
        this.currentTableData = val
      },
      immediate: true
    }
  },
  mounted() {
    this._validationAuth()
  },
  methods: {
    // 验证权限
    _validationAuth() {
      this.auth.add = this.$has('/system/storage/storage/add')
      this.auth.upload = this.$has('/system/storage/storage/upload')
      this.auth.rename = this.$has('/system/storage/storage/rename')
      this.auth.replace = this.$has('/system/storage/storage/replace')
      this.auth.cover = this.$has('/system/storage/storage/cover')
      this.auth.clear_cover = this.$has('/system/storage/storage/clear_cover')
      this.auth.default = this.$has('/system/storage/storage/default')
      this.auth.move = this.$has('/system/storage/storage/move')
      this.auth.del = this.$has('/system/storage/storage/del')
      this.auth.refresh = this.$has('/system/storage/storage/refresh')
      this.auth.link = this.$has('/system/storage/storage/link')
    },
    // 资源上传成功后处理
    _getUploadFileList(files) {
      let pos = -1
      if (!this.uploadConfig.replace) {
        this.currentTableData.forEach((value, index) => {
          // 查找文件夹出现的位置
          if (value.type === 2) {
            pos = index
          }
        })
      }

      // eslint-disable-next-line no-unused-vars
      for (const value of files) {
        if (value.status !== 'success') {
          continue
        }

        const response = value.response
        if (!response || response.status !== 200) {
          continue
        }

        if (!this.uploadConfig.replace) {
          this.currentTableData.splice(pos + 1, 0, response.data[0])
        } else {
          this.$set(this.currentTableData, this.uploadConfig.replace, response.data[0])
        }
      }
    },
    // 切换目录
    switchFolder(storageId) {
      this.$emit('clearName')
      this.$emit('refresh', storageId, true)
    },
    // 打开资源
    openStorage(index) {
      // 当前资源对象
      const storage = this.currentTableData[index]
      switch (storage['type']) {
        case 0:
          this.$preview(storage['url'])
          break

        case 1:
          util.open(util.getDownloadUrl(storage, ''))
          break

        case 2:
          this.switchFolder(storage['storage_id'])
          break

        case 3:
          this.$player(storage['url'], storage['mime'], storage['cover'])
          break

        default:
          this.$message.warning('打开资源出现异常操作')
      }
    },
    // 批量删除
    handleDelete(val) {
      const storageId = val ? [val] : this.checkList
      if (storageId.length === 0) {
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
          delStorageList(storageId)
            .then(() => {
              for (let i = this.currentTableData.length - 1; i >= 0; i--) {
                if (storageId.indexOf(this.currentTableData[i].storage_id) !== -1) {
                  this.currentTableData.splice(i, 1)
                }
              }

              if (this.currentTableData.length <= 0) {
                this.$emit('refresh', this.storageId)
              }

              this.directoryList = []
              this.$message.success('操作成功')
            })
        })
        .catch(() => {
        })
    },
    // 新建目录
    handleCreate() {
      this.nameForm = {
        name: undefined,
        parent_id: this.storageId
      }

      this.$nextTick(() => {
        this.$refs.name.clearValidate()
        this.$refs.input.focus()
      })

      this.dialogLoading = false
      this.nameStatus = 'create'
      this.nameFormVisible = true
    },
    // 请求创建目录
    create() {
      this.$refs.name.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          addStorageDirectoryItem({ ...this.nameForm })
            .then(res => {
              this.currentTableData.unshift({
                ...res.data,
                is_default: 0
              })

              this.directoryList = []
              this.nameFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    },
    // 获取可选择目录
    getStorageDirectory() {
      if (!this.directoryList.length) {
        getStorageDirectorySelect()
          .then(res => {
            this.directoryList = util.formatDataToTree(res.data.list, 'storage_id')
            this.directoryList.unshift({
              storage_id: 0,
              parent_id: 0,
              name: '根目录'
            })
          })
      }
    },
    // 移动目录
    handleMove(val) {
      const storageId = val ? [val] : this.checkList
      if (storageId.length === 0) {
        this.$message.error('请选择要操作的数据')
        return
      }

      this.getStorageDirectory()
      this.moveIdList = storageId
      this.dialogLoading = false
      this.moveFormVisible = true
    },
    // 请求移动目录
    moveStorage() {
      const node = this.$refs.directory.getCurrentNode()
      if (!node) {
        this.$message.error('请选择需要移动到的目录')
        return
      }

      this.dialogLoading = true
      moveStorageList(this.moveIdList, node.storage_id)
        .then(res => {
          if (res.data.length) {
            this.$emit('refresh', this.storageId)
          }

          this.checkList = []
          this.directoryList = []
          this.moveFormVisible = false
          this.$message.success('操作成功')
        })
        .catch(() => {
          this.dialogLoading = false
        })
    },
    handleRename(index) {
      this.nameForm = {
        name: this.currentTableData[index]['name'],
        storage_id: this.currentTableData[index]['storage_id'],
        index
      }

      this.$nextTick(() => {
        this.$refs.name.clearValidate()
        this.$refs.input.select()
      })

      this.dialogLoading = false
      this.nameStatus = 'update'
      this.nameFormVisible = true
    },
    rename() {
      this.$refs.name.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          renameStorageItem(this.nameForm.storage_id, this.nameForm.name)
            .then(res => {
              this.currentTableData[this.nameForm.index].name = res.data.name
              this.directoryList = []
              this.nameFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    },
    setDefault(index) {
      const is_default = this.currentTableData[index].is_default ? 0 : 1
      setStorageDirectoryDefault(this.currentTableData[index].storage_id, is_default)
        .then(() => {
          this.currentTableData.forEach((value, index) => {
            if (value.type === 2) {
              this.currentTableData[index].is_default = 0
            }
          })

          this.currentTableData[index].is_default = is_default
          this.$message.success('操作成功')
        })
    },
    getLink(index) {
      let link = ''
      const storage = this.currentTableData[index]

      switch (storage.type) {
        case 0:
        case 3:
          link = storage.url
          break
        case 1:
          link = util.getDownloadUrl(storage, '')
          break
      }

      clipboard.writeText(link)
        .then(() => {
          this.$message.success('已复制链接到剪贴板')
        })
        .catch(err => {
          this.$message.error(err)
        })
    },
    handleRefresh(index) {
      clearStorageThumb(this.currentTableData[index].storage_id)
        .then(() => {
          this.$message.success('操作成功')
        })
    },
    handleCover(index) {
      const storage = this.currentTableData[index]
      setStorageCover(storage.storage_id, storage.parent_id)
        .then(() => {
          this.$message.success('操作成功')
        })
    },
    handleClearCover(index) {
      const storage = this.currentTableData[index]
      clearStorageCover(storage.storage_id)
        .then(() => {
          storage['cover'] = ''
          this.$message.success('操作成功')
        })
    },
    // 资源操作
    handleControlItemClick(command) {
      switch (command.type) {
        // 重命名
        case 'rename':
          this.handleRename(command.index)
          break
        // 设为默认
        case 'default':
          this.setDefault(command.index)
          break
        // 转移目录
        case 'move':
          this.handleMove(command.storage_id)
          break
        // 删除资源
        case 'delete':
          this.handleDelete(command.storage_id)
          break
        // 复制外链
        case 'link':
          this.getLink(command.index)
          break
        // 清除缓存
        case 'refresh':
          this.handleRefresh(command.index)
          break
        // 设为封面
        case 'cover':
          this.handleCover(command.index)
          break
        // 选择海报
        case 'video_cover':
          this.videoCover = command.index
          this.$refs.storage.handleStorageDlg([0, 2])
          break
        // 取消封面
        case 'clear_cover':
          this.handleClearCover(command.index)
          break
        // 替换上传
        case 'replace':
          this.handleReplace(command.index)
          break
        default:
          this.$message.error('无效的操作')
          break
      }
    },
    // 上传资源
    handleUpload() {
      this.uploadConfig = {
        uploadTip: '请选择资源进行(支持拖拽)上传，',
        multiple: true,
        accept: '*/*',
        limit: 0,
        replace: false
      }

      this.$refs.upload.handleUploadDlg()
    },
    // 替换资源
    handleReplace(index) {
      const storage = this.currentTableData[index]
      this.uploadConfig = {
        uploadTip: '替换上传，资源类型需要相同(支持拖拽)，',
        multiple: false,
        accept: storage.mime,
        limit: 1,
        replace: index
      }

      this.$refs.upload.setReplaceId(storage.storage_id)
      this.$refs.upload.handleUploadDlg()
    },
    // 选择视频海报
    handleVideoCover(files) {
      let cover
      const storage = this.currentTableData[this.videoCover]

      // eslint-disable-next-line no-unused-vars
      for (const value of files) {
        if (value.type === 0) {
          cover = value
          break
        }
      }

      if (cover) {
        setStorageCover(cover.storage_id, storage.storage_id)
          .then(() => {
            storage['cover'] = cover.url
            this.$message.success('操作成功')
          })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .breadcrumb {
    background-color: #fff;
    border: 1px solid $color-border-1;
    padding: 10px !important;
  }
  .storage-list {
    overflow: hidden;
    list-style: none;
    padding: 0;
    margin: 0;
    border-style: solid;
    border-color: $color-border-1;
    border-width: 1px 0 0 1px;
    background-color: #fff;
  }
  .storage-list li {
    float: left;
    height: 275px;
    font-size: 13px;
    opacity: 1;
    border-style: solid;
    border-color: $color-border-1;
    border-width: 0 1px 1px 0;
  }
  .storage-list>li:hover {
     background-color: $color-bg;
  }
  .storage-list>li:hover .more {
    display: inline-block;
  }
  .storage-list li dl dt .picture {
    border: none;
  }
  .storage-list li dl dt .covers .el-image,
  .storage-list li dl dt .picture .el-image {
    background-color: #F5F7FA;
    text-align: center;
    vertical-align: middle;
    display: table-cell;
    width: 158px;
    height: 158px;
    overflow: hidden;
    cursor: pointer;
  }
  .storage-list li:after {
    content: "";
    height: 100%;
  }
  .storage-list li:after,
  .storage-list li span {
    display: inline-block;
    vertical-align: middle;
  }
  .storage-list li span {
    line-height: normal;
    color: $color-text-sub;
    transition: color .15s linear;
  }
  .storage-list .storage-name {
    color: $color-text-main;
    text-overflow:ellipsis;
    white-space:nowrap;
    overflow:hidden;
    width:148px;
    height: 21px;
  }
  .storage-list .more {
    width: 20px;
    height: 15px;
    color: $color-text-sub;
    cursor: pointer;
    display: none;
  }
  .storage-check {
    position: absolute;
    margin-top: 3px;
    margin-left: 3px;
    width: 30px;
    height: 30px;
    /deep/ .el-checkbox__label{
      display: none;
    }
  }
  .el-image /deep/ .el-image__inner {
    width: auto;
    height: auto;
    max-width: 158px;
    max-height: 158px;
  }
  .brother-showing i {
    width: 16px;
  }
</style>

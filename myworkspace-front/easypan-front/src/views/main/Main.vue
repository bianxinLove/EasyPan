<template>
  <div class="main-container">
      <div class="top">
          <div class="top-op">
              <div class="left-buttons">
                <el-upload
                  :show-file-list="false"
                  :with-credentials="true"
                  :multiple="true"
                  :http-request="addFile"
                  :accept="fileAccept"
                >
                    <el-button type="primary" class="upload-btn">
                      <span class="iconfont icon-upload"></span>
                      上传文件
                    </el-button>
                </el-upload>
                
                <el-button type="success" @click="newFolder" class="folder-btn">
                  <span class="iconfont icon-folder-add"></span>
                  新建文件夹
                </el-button>
              </div>

              <div class="right-buttons">
                <div class="search-panel">
                  <el-input
                    clearable
                    placeholder="搜索文件..."
                    v-model="fileNameFuzzy"
                    @keyup.enter="search"
                    class="search-input"
                  >
                      <template #prefix>
                          <i class="iconfont icon-search"></i>
                      </template>
                  </el-input>
                </div>

                <div class="action-buttons">
                  <el-button
                    type="danger"
                    :class="['action-btn', {'disabled': selectFileIdList.length == 0}]"
                    :disabled="selectFileIdList.length == 0"
                    @click="delFileBatch"
                  >
                    <span class="iconfont icon-del"></span>
                    批量删除
                  </el-button>
                  
                  <el-button
                    type="warning"
                    :class="['action-btn', {'disabled': selectFileIdList.length == 0}]"
                    :disabled="selectFileIdList.length == 0"
                    @click="moveFolderBatch"
                  >
                    <span class="iconfont icon-move"></span>
                    批量移动
                  </el-button>

                  <div class="refresh-btn" @click="loadDataList">
                    <i class="iconfont icon-refresh"></i>
                  </div>
                </div>
              </div>
          </div>
          
          <Navigation ref="navigationRef" @navChange="navChange"></Navigation>
      </div>

      <div class="content-area" :class="{'empty': !tableData.list || tableData.list.length === 0}">
        <div class="file-list" v-if="tableData.list && tableData.list.length > 0">
          <Table
            ref="dataTableRef"
            :columns="columns"
            :dataSource="tableData"
            :fetch="loadDataList"
            :initFetch="false"
            :options="tableOptions"
            @rowSelected="rowSelected"
          >
          <template #fileName="{ index, row }">
              <div
                class="file-item"
                @mouseenter="showOp(row)"
                @mouseleave="cancelShowOp(row)"
              >
                <div class="file-icon">
                  <template
                    v-if="(row.fileType == 3 || row.fileType == 1) && row.status == 2"
                  >
                    <Icon :cover="row.fileCover" :width="40"></Icon>
                  </template>
                  <template v-else>
                    <Icon v-if="row.folderType == 0" :fileType="row.fileType" :width="40"></Icon>
                    <Icon v-if="row.folderType == 1" :fileType="0" :width="40"></Icon>
                  </template>
                </div>

                <div class="file-info">
                  <span class="file-name" v-if="!row.showEdit" :title="row.fileName">
                    <span @click="preview(row)">{{ row.fileName }}</span>
                    <span v-if="row.status == 0" class="transfer-status">转码中</span>
                    <span v-if="row.status == 1" class="transfer-status transfer-fail">转码失败</span>
                  </span>
                  
                  <div class="edit-panel" v-if="row.showEdit">
                    <el-input
                      v-model.trim="row.fileNameReal"
                      ref="editNameRef"
                      :maxLength="190"
                      @keyup.enter="saveNameEdit(index)"
                    >
                      <template #suffix>{{ row.fileSuffix }}</template>
                    </el-input>
                    <div class="edit-buttons">
                      <span
                        :class="['iconfont icon-right1', {'not-allow': !row.fileNameReal}]"
                        @click="saveNameEdit(index)"
                      ></span>
                      <span
                        class="iconfont icon-error"
                        @click="cancelNameEdit(index)"
                      ></span>
                    </div>
                  </div>
                </div>

                <div class="file-actions" v-if="row.showOp && row.fileId && row.status == 2">
                  <el-tooltip content="分享" placement="top">
                    <span class="iconfont action-icon icon-share1" @click="share(row)"></span>
                  </el-tooltip>
                  
                  <el-tooltip content="下载" placement="top" v-if="row.folderType == 0">
                    <span class="iconfont action-icon icon-download" @click="download(row)"></span>
                  </el-tooltip>
                  
                  <el-tooltip content="删除" placement="top">
                    <span class="iconfont action-icon icon-del" @click="delFile(row)"></span>
                  </el-tooltip>
                  
                  <el-tooltip content="重命名" placement="top">
                    <span class="iconfont action-icon icon-edit" @click="editFileName(index)"></span>
                  </el-tooltip>
                  
                  <el-tooltip content="移动" placement="top">
                    <span class="iconfont action-icon icon-move" @click="moveFolder(row)"></span>
                  </el-tooltip>
                </div>
              </div>
          </template>
          <template #fileSize="{index, row}">
              <span v-if="row.fileSize" class="file-size">{{
                proxy.Utils.size2Str(row.fileSize)
              }}</span>
          </template>
          </Table>
        </div>

        <div class="no-data" v-else>
          <div class="no-data-inner">
            <Icon iconName="no_data" :width="150" fit="fill"></Icon>
            <div class="tips">当前目录为空，开始创建或上传内容吧</div>
            <div class="op-list">
              <el-upload
                :show-file-list="false"
                :with-credentials="true"
                :multiple="true"
                :http-request="addFile"
                :accept="fileAccept"
              >
                <div class="op-item upload">
                  <Icon iconName="file" :width="70"></Icon>
                  <div>上传文件</div>
                </div>
              </el-upload>
              <div class="op-item folder" v-if="category == 'all'" @click="newFolder">
                <Icon iconName="folder" :width="70"></Icon>
                <div>新建目录</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <FolderSelect
        ref="folderSelectRef"
        @folderSelect="moveFolderDone"
      ></FolderSelect>
      <Preview ref="previewRef"></Preview>
      <ShareFile ref="shareRef"></ShareFile>
  </div>
</template>

<script setup>
import CategoryInfo from "@/js/CategoryInfo.js";
import ShareFile from "./ShareFile.vue";
import { ref, reactive, getCurrentInstance, nextTick, computed } from "vue";
const { proxy } = getCurrentInstance();

const emit = defineEmits(["addFile"]);
const addFile = (fileData) => {
  emit("addFile", { file: fileData.file, filePid: currentFolder.value.fileId });
};

// 添加文件回调
const reload = () => {
  showLoading.value = false;
  loadDataList();
};
defineExpose({
  reload,
});
// 当前目录
const currentFolder = ref({ fileId: "0" });

const api = {
    loadDataList: "/file/loadDataList",
    rename: "/file/rename",
    newFoloder: "/file/newFoloder",
    getFolderInfo: "/file/getFolderInfo",
    delFile: "/file/delFile",
    changeFileFolder: "/file/changeFileFolder",
    createDownloadUrl: "/file/createDownloadUrl",
    download: "/api/file/download",
};

const fileAccept = computed( () => {
  const categoryItem = CategoryInfo[category.value];
  return categoryItem ? categoryItem.accept : "*";
});

const columns = [
    {
        label: "文件名",
        prop: "fileName",
        scopedSlots: "fileName",
    },
    {
        label: "修改时间",
        prop: "lastUpdateTime",
        width: 200,
    },
    {
        label: "大小",
        prop: "fileSize",
        scopedSlots: "fileSize",
        width: 200,
    },
];

// 搜索
const search = () => {
  showLoading.value = true;
  loadDataList();
};
const tableData = ref({});
const tableOptions = ref({
    extHeight: 50,
    selectType: "checkbox",
});

const fileNameFuzzy = ref();
const showLoading = ref(true);
const category = ref();
const loadDataList = async () => {
    let params = {
        pageNo: tableData.value.pageNo,
        pageSize: tableData.value.pageSize,
        fileNameFuzzy: fileNameFuzzy.value,
        filePid: currentFolder.value.fileId,
        category: category.value,
    };
    if (params.category !== "all") {
        delete params.filePid;
    }
    let result = await proxy.Request({
        url: api.loadDataList,
        showLoading: showLoading.value,
        params: params,
    });
    if (!result) {
        return;
    }
    tableData.value = result.data;
};

// 展示操作按钮
const showOp = (row) => {
  tableData.value.list.forEach((element) => {
    element.showOp = false;
  });
  row.showOp = true;
};

const cancelShowOp = (row) => {
    row.showOp = false;
};

// 编辑行
const editing = ref(false);
const editNameRef = ref();
// 新建文件夹
const newFolder = () => {
    if (editing.value) {
        return;
    }
    tableData.value.list.forEach(element => {
        element.showEdit = false;
    });
    editing.value = true;
    tableData.value.list.unshift({
        showEdit: true,
        fileType: 0,
        fileId: "",
        filePid: 0,
    });
    nextTick(() => {
        editNameRef.value.focus();
    });
};
const cancelNameEdit = (index) => {
    const fileData = tableData.value.list[index];
    if (fileData.fileId) {
        fileData.showEdit = false;
    } else {
        tableData.value.list.splice(index, 1);
    }
    editing.value = false;
};

const saveNameEdit = async (index) => {
    const { fileId, filePid, fileNameReal } = tableData.value.list[index];
    if (fileNameReal == "" || fileNameReal.indexOf("/") != -1) {
        proxy.Message.warning("文件名不能为空且不能含有斜杠");
        return;
    }
    let url = api.rename;
    if (fileId == "") {
        url = api.newFoloder;
    }
    let result = await proxy.Request({
        url: url,
        params: {
            fileId: fileId,
            filePid: filePid,
            fileName: fileNameReal,
        },
    });
    if (!result) {
        return;
    }
    tableData.value.list[index] = result.data;
    editing.value = false;
};

const editFileName = (index) => {
    if (tableData.value.list[0].fileId == "") {
        tableData.value.list.splice(0, 1);
        index = index - 1;
    }
    tableData.value.list.forEach((element) => {
        element.showEdit = false;
    });
    let currentData = tableData.value.list[index];
    currentData.showEdit = true;
    // 编辑文件
    if (currentData.folderType == 0) {
        currentData.fileNameReal = currentData.fileName.substring(
          0,
          currentData.fileName.indexOf(".")
        );
        currentData.fileSuffix = currentData.fileName.substring(
          currentData.fileName.indexOf(".")
        );
    } else {
        currentData.fileNameReal = currentData.fileName;
        currentData.fileSuffix = "";
    }
    editing.value = true;
    nextTick(() => {
        editNameRef.value.focus();
    });
};

// 多选
const selectFileIdList = ref([]);
const rowSelected = (rows) => {
  selectFileIdList.value = [];
  rows.forEach((item) => {
    selectFileIdList.value.push(item.fileId);
  });
};
// 删除
const delFile = (row) => {
  proxy.Confirm(
    `你确定要删除【${row.fileName}】吗? 删除的文件可在10天内通过回收站还原`,
    async () => {
      let result = await proxy.Request({
        url: api.delFile,
        params: {
          fileIds: row.fileId,
        },
      });
      if (!result) {
        return;
      }
      loadDataList();
    }
  );
};

const delFileBatch = () => {
  if (selectFileIdList.value.length == 0) {
    return;
  }
  proxy.Confirm(
    `你确定要删除这些文件吗? 删除的文件可在10天内通过回收站还原`,
    async () => {
      let result = await proxy.Request({
        url: api.delFile,
        params: {
          fileIds: selectFileIdList.value.join(","),
        },
      });
      if (!result) {
        return;
      }
      loadDataList();
    }
  );
};

const folderSelectRef = ref();
const currentMoveFile = ref({});

const moveFolder = (data) => {
  currentMoveFile.value = data;
  folderSelectRef.value.showFolderDialog(currentFolder.value.fileId);
};

const moveFolderBatch = () => {
  currentMoveFile.value = {};
  folderSelectRef.value.showFolderDialog(currentFolder.value.fileId);
};

const moveFolderDone  = async (folderId) => {
  if (currentFolder.value.fileId == folderId) {
    proxy.Message.warning("文件正在当前目录, 无需移动");
    return;
  }
  let fileIdsArray = [];
  if (currentMoveFile.value.fileId) {
    fileIdsArray.push(currentMoveFile.value.fileId);
  } else {
    fileIdsArray = fileIdsArray.concat(selectFileIdList.value);
  }
  let result = await proxy.Request({
    url: api.changeFileFolder,
    params: {
      fileIds: fileIdsArray.join(","),
      filePid: folderId,
    },
  });
  if (!result) {
    return;
  }
  folderSelectRef.value.close();
  loadDataList();
};

// 预览
const navigationRef = ref();
const previewRef = ref();
const preview = (data) => {
  // 目录
  if (data.folderType == 1) {
    navigationRef.value.openFolder(data);
    return;
  }
  // 文件
  if (data.status != 2) {
    proxy.Message.warning("文件未完成转码, 无法预览");
    return;
  }
  previewRef.value.showPreview(data, 0);
};

const navChange = (data) => {
  const { categoryId, curFolder } = data;
  currentFolder.value = curFolder;
  category.value = categoryId;
  loadDataList();
};

// 下载文件
const download = async (row) => {
  let result = await proxy.Request({
      url: api.createDownloadUrl + "/" + row.fileId,
  });
  if (!result) {
      return;
  }
  window.location.href = api.download + "/" + result.data;
};

// 分享
const shareRef = ref();
const share = (row) => {
  shareRef.value.show(row);
};
</script>

<style lang="scss" scoped>
.main-container {
  height: 100vh;
  padding: 16px;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e7eb 100%);

  .top {
    background: #fff;
    border-radius: 16px;
    padding: 16px;
    margin-bottom: 16px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    flex-shrink: 0;

    .top-op {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;

      .left-buttons {
        display: flex;
        gap: 12px;

        .upload-btn, .folder-btn {
          height: 36px;
          padding: 0 20px;
          border-radius: 8px;
          font-weight: 500;
          transition: all 0.3s ease;
          
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
          }

          .iconfont {
            margin-right: 8px;
            font-size: 16px;
          }
        }
      }

      .right-buttons {
        display: flex;
        align-items: center;
        gap: 16px;

        .search-panel {
          .search-input {
            width: 280px;

            :deep(.el-input__wrapper) {
              border-radius: 20px;
              padding: 0 16px;
              height: 36px;
              box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
              transition: all 0.3s ease;

              &:hover, &:focus {
                box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
              }
            }
          }
        }

        .action-buttons {
          display: flex;
          align-items: center;
          gap: 12px;

          .action-btn {
            height: 36px;
            border-radius: 8px;
            padding: 0 16px;
          }

          .refresh-btn {
            width: 36px;
            height: 36px;
          }
        }
      }
    }
  }

  .content-area {
    background: #fff;
    border-radius: 16px;
    padding: 16px;
    flex: 1;
    overflow: auto;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
    display: flex;
    flex-direction: column;

    &.empty {
      justify-content: center;
      align-items: center;
    }

    .file-list {
      height: 100%;
      overflow: auto;

      :deep(.el-table) {
        height: 100%;
      }

      .file-item {
        display: flex;
        align-items: center;
        padding: 8px;
        border-radius: 8px;
        transition: all 0.3s ease;
        
        &:hover {
          background: #f8fafd;
          transform: translateX(4px);
        }

        .file-icon {
          margin-right: 12px;
        }

        .file-info {
          flex: 1;
          
          .file-name {
            font-size: 14px;
            color: #303133;
            cursor: pointer;

            &:hover {
              color: #409eff;
            }
            
            .transfer-status {
              margin-left: 10px;
              padding: 2px 6px;
              border-radius: 10px;
              font-size: 12px;
              background: #e6a23c;
              color: #fff;

              &.transfer-fail {
                background: #f56c6c;
              }
            }
          }

          .edit-panel {
            display: flex;
            align-items: center;
            gap: 10px;

            .el-input {
              :deep(.el-input__wrapper) {
                border-radius: 8px;
              }
            }

            .edit-buttons {
              display: flex;
              gap: 6px;

              .iconfont {
                font-size: 18px;
                cursor: pointer;

                &.icon-right1 {
                  color: #67c23a;
                  
                  &.not-allow {
                    color: #c0c4cc;
                    cursor: not-allowed;
                  }
                }

                &.icon-error {
                  color: #f56c6c;
                }
              }
            }
          }
        }

        .file-actions {
          display: none;
          gap: 12px;
          padding: 0 10px;

          .iconfont {
            font-size: 16px;
            color: #606266;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;

            &::after {
              margin-left: 4px;
              font-size: 12px;
            }

            &.icon-share1::after {
              content: "分享";
            }

            &.icon-download::after {
              content: "下载";
            }

            &.icon-del::after {
              content: "删除";
            }

            &.icon-edit::after {
              content: "重命名";
            }

            &.icon-move::after {
              content: "移动";
            }

            &:hover {
              color: #409eff;
              transform: scale(1.1);
            }
          }
        }

        &:hover .file-actions {
          display: flex;
        }
      }
    }

    .no-data {
      padding: 30px;

      .no-data-inner {
        .tips {
          margin: 20px 0;
        }

        .op-list {
          gap: 40px;
          margin-top: 30px;

          .op-item {
            padding: 24px;
            border-radius: 12px;
            
            &:hover {
              transform: translateY(-3px);
            }

            div {
              margin-top: 10px;
            }
          }
        }
      }
    }
  }
}
</style>

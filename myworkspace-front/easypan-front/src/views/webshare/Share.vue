<template>
  <div class="share">
    <div class="header">
      <div class="header-content">
        <div class="logo" @click="jump">
          <span class="iconfont icon-pan"></span>
          <div class="name">个人云盘</div>
        </div>
      </div>
    </div>
    <div class="share-body">
      <template v-if="Object.keys(shareInfo).length == 0">
        <div class="loading" v-loading="Object.keys(shareInfo).length == 0"></div>
      </template>
      <template v-else>
        <div class="share-panel">
          <div class="share-user-info">
            <!-- <div class="avatar"> -->
              <Avatar
                :userId="shareInfo.userId"
                :avatar="shareInfo.avatar"
                :width="80"
                class="avatar-img"
              ></Avatar>
            <!-- </div> -->
            <div class="share-info">
              <div class="user-info">
                <span class="nick-name">{{ shareInfo.nickName }}</span>
                <span class="share-time">分享于：{{ shareInfo.shareTime }}</span>
              </div>
              <div class="file-name">分享文件：{{ shareInfo.fileName }}</div>
            </div>
          </div>
          <div class="share-op-btn">
            <el-button
              type="primary"
              @click="cancelShare"
              v-if="shareInfo.currentUser"
              class="action-btn"
            >
              <span class="iconfont icon-cancel"></span>
              取消分享
            </el-button>
            <el-button
              v-else
              type="primary"
              @click="save2MyPan"
              :disabled="selectIdList.length == 0"
              class="action-btn"
            >
              <span class="iconfont icon-import"></span>
              保存到我的网盘
            </el-button>
          </div>
        </div>
        <Navigation
          ref="navigationRef"
          @navChange="navChange"
          :shareId="shareId"
        ></Navigation>
        <div class="file-list">
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
                <template
                  v-if="
                    (row.fileType == 3 || row.fileType == 1) && row.status == 2
                  "
                >
                  <Icon :cover="row.fileCover" :width="32"></Icon>
                </template>
                <template v-else>
                  <Icon
                    v-if="row.folderType == 0"
                    :fileType="row.fileType"
                  ></Icon>
                  <Icon v-if="row.folderType == 1" :fileType="0"></Icon>
                </template>
                <span class="file-name" :title="row.fileName">
                  <span @click="preview(row)">{{ row.fileName }}</span>
                </span>
                <span class="op">
                  <span
                    class="iconfont icon-download"
                    v-if="row.folderType == 0"
                    @click="download(row)"
                    >下载</span
                  >
                  <span
                    class="iconfont icon-import"
                    @click="save2MyPanSingle(row)"
                    v-if="row.showOp && !shareInfo.currentUser"
                    >保存到我的网盘</span
                  >
                </span>
              </div>
            </template>
            <template #fileSize="{ index, row }">
              <span v-if="row.fileSize">{{
                proxy.Utils.size2Str(row.fileSize)
              }}</span>
            </template>
          </Table>
        </div>
      </template>
      <FolderSelect
        ref="folderSelectRef"
        @folderSelect="save2MyPanDone"
      ></FolderSelect>
      <Preview ref="previewRef"></Preview>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance, nextTick } from "vue";
const { proxy } = getCurrentInstance();
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
const route = useRoute();

const api = {
    getShareLoginInfo: "/showShare/getShareLoginInfo",
    loadFileList: "/showShare/loadFileList",
    createDownloadUrl: "/showShare/createDownloadUrl",
    download: "/api/showShare/download",
    cancelShare: "/share/cancelShare",
    saveShare: "/showShare/saveShare",
};

const shareId = route.params.shareId;
const shareInfo = ref({});
const getShareInfo = async () => {
    let result = await proxy.Request({
        url: api.getShareLoginInfo,
        showLoading: false,
        params: {
            shareId
        },
    });
    if (!result) {
        return;
    }
    if (result.data == null) {
        router.push(`/shareCheck/${shareId}`);
        return;
    }
    shareInfo.value = result.data;
};
getShareInfo();

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
const tableData = ref({});
const tableOptions = {
    extHeight: 80,
    selectType: "checkbox",
};

const loadDataList = async () => {
    let params = {
        pageNo: tableData.value.pageNo,
        pageSize: tableData.value.pageSize,
        shareId: shareId,
        filePid: currentFolder.value.fileId,
    };
    let result = await proxy.Request({
        url: api.loadFileList,
        params,
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

const selectIdList = ref([]);
const rowSelected = (rows) => {
    selectIdList.value = [];
    rows.forEach((item) => {
        selectIdList.value.push(item.fileId);
    });
};

const currentFolder = ref({ fileId: "0" });
const navChange = (data) => {
    const { curFolder } = data;
    currentFolder.value = curFolder;
    loadDataList();
};

// 预览, 查看
const previewRef = ref();
const navigationRef = ref();

const preview = (data) => {
    if (data.folderType == 1) {
        navigationRef.value.openFolder(data);
        return;
    }
    data.shareId = shareId;
    previewRef.value.showPreview(data, 2);
};

// 下载文件
const download = async (row) => {
  let result = await proxy.Request({
      url: api.createDownloadUrl + "/" + shareId + "/" + row.fileId,
  });
  if (!result) {
      return;
  }
  window.location.href = api.download + "/" + result.data;
};

// 保存到我的网盘
const folderSelectRef = ref();
const save2MyPanFileIdArray = [];
const save2MyPan = () => {
    if (selectIdList.value.length == 0) {
        return;
    }
    if (!proxy.VueCookies.get("userInfo")) {
        router.push("/login?redirectUrl=" + route.path);
        return;
    }
    save2MyPanFileIdArray.value = selectIdList.value;
    folderSelectRef.value.showFolderDialog();
};

const save2MyPanSingle = (row) => {
    if (!proxy.VueCookies.get("userInfo")) {
        router.push("/login?redirectUrl=" + route.path);
        return;
    }
    save2MyPanFileIdArray.value = [row.fileId];
    folderSelectRef.value.showFolderDialog();
};

const save2MyPanDone = async (folderId) => {
    let result = await proxy.Request({
        url: api.saveShare,
        params: {
            shareId: shareId,
            shareFileIds: save2MyPanFileIdArray.value.join(","),
            myFolderId: folderId,
        },
    });
    if (!result) {
        return;
    }
    loadDataList();
    proxy.Message.success("保存成功");
    folderSelectRef.value.close();
};

// 取消分享
const cancelShare = () => {
    proxy.Confirm(`你确定要取消分享吗?`, async () => {
        let result = await proxy.Request({
            url: api.cancelShare,
            params: {
                shareIds: shareId,
            },
        });
        if (!result) {
            return;
        }
        proxy.Message.success("取消分享成功");
        router.push("/");
    });
};

const jump = () => {
    router.push("/");
};
</script>

<style lang="scss" scoped>
.share {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e7ed 100%);
  position: relative;

  .header {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 100;
    height: 60px;

    .header-content {
      max-width: 1200px;
      margin: 0 auto;
      height: 100%;
      display: flex;
      align-items: center;
      padding: 0 20px;

      .logo {
        display: flex;
        align-items: center;
        cursor: pointer;
        padding: 8px 16px;
        border-radius: 12px;
        background: linear-gradient(135deg, rgba(64, 158, 255, 0.1) 0%, rgba(64, 158, 255, 0.05) 100%);
        backdrop-filter: blur(8px);
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        position: relative;
        overflow: hidden;

        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: linear-gradient(45deg, 
            rgba(64, 158, 255, 0) 0%,
            rgba(64, 158, 255, 0.1) 50%,
            rgba(64, 158, 255, 0) 100%);
          transform: translateX(-100%);
          transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
        }

        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 4px 12px rgba(64, 158, 255, 0.15);

          &::before {
            transform: translateX(100%);
          }

          .icon-pan {
            transform: rotate(15deg) scale(1.1);
            color: #409EFF;
          }

          .name {
            background: linear-gradient(to right, #409EFF, #40E0D0);
            -webkit-background-clip: text;
            color: transparent;
            letter-spacing: 1px;
          }
        }

        .icon-pan {
          font-size: 28px;
          color: #2c3e50;
          margin-right: 12px;
          transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          position: relative;

          &::after {
            content: '';
            position: absolute;
            width: 8px;
            height: 8px;
            background: #409EFF;
            border-radius: 50%;
            top: 0;
            right: -2px;
            transform: scale(0);
            transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          }
        }

        .name {
          font-size: 20px;
          font-weight: 600;
          color: #2c3e50;
          transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          position: relative;
          
          &::after {
            content: '';
            position: absolute;
            bottom: -2px;
            left: 0;
            width: 100%;
            height: 2px;
            background: linear-gradient(to right, #409EFF, transparent);
            transform: scaleX(0);
            transform-origin: left;
            transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          }
        }

        &:hover {
          .icon-pan::after {
            transform: scale(1);
          }

          .name::after {
            transform: scaleX(1);
          }
        }
      }
    }
  }

  .share-body {
    max-width: 1200px;
    margin: 80px auto 0;
    padding: 0 20px;

    .loading {
      height: calc(100vh - 80px);
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .share-panel {
      background: rgba(255, 255, 255, 0.95);
      backdrop-filter: blur(10px);
      border-radius: 16px;
      padding: 30px;
      margin-bottom: 25px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
      display: flex;
      justify-content: space-between;
      align-items: center;
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
      border: 1px solid rgba(255, 255, 255, 0.2);

      &:hover {
        transform: translateY(-2px);
        box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
      }

      .share-user-info {
        display: flex;
        align-items: center;
        flex: 1;

        .avatar {
          margin-right: 20px;
          border-radius: 50%;
          overflow: hidden;
          border: 2px solid rgba(102, 126, 234, 0.2);
          transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          display: flex;
          align-items: center;
          justify-content: center;
          background: linear-gradient(135deg, rgba(102, 126, 234, 0.1) 0%, rgba(102, 126, 234, 0.05) 100%);
          width: 50px;
          height: 50px;

          &:hover {
            transform: scale(1.05) rotate(5deg);
            border-color: rgba(102, 126, 234, 0.4);
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
          }

          :deep(.avatar-img) {
            width: 100%;
            height: 100%;
            object-fit: cover;
            border-radius: 50%;
            display: block;
          }
        }

        .share-info {
          .user-info {
            display: flex;
            align-items: center;
            margin-bottom: 10px;

            .nick-name {
              font-size: 18px;
              font-weight: 600;
              color: #2c3e50;
              position: relative;
              padding-right: 15px;

              &::after {
                content: '';
                position: absolute;
                right: 0;
                top: 50%;
                transform: translateY(-50%);
                width: 4px;
                height: 4px;
                background: #409EFF;
                border-radius: 50%;
              }
            }

            .share-time {
              margin-left: 15px;
              font-size: 14px;
              color: #666;
              background: rgba(102, 126, 234, 0.1);
              padding: 4px 8px;
              border-radius: 4px;
            }
          }

          .file-name {
            font-size: 15px;
            color: #666;
            background: rgba(102, 126, 234, 0.05);
            padding: 8px 12px;
            border-radius: 8px;
            display: inline-block;
          }
        }
      }

      .share-op-btn {
        .action-btn {
          height: 44px;
          padding: 0 24px;
          border-radius: 10px;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          border: none;
          transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
          font-weight: 500;
          letter-spacing: 0.5px;
          position: relative;
          overflow: hidden;

          &::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, 
              rgba(255, 255, 255, 0) 0%,
              rgba(255, 255, 255, 0.1) 50%,
              rgba(255, 255, 255, 0) 100%);
            transform: translateX(-100%);
            transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
          }

          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);

            &::before {
              transform: translateX(100%);
            }
          }

          &:active {
            transform: translateY(0);
          }

          .iconfont {
            margin-right: 8px;
            font-size: 16px;
          }
        }
      }
    }

    .file-list {
      background: rgba(255, 255, 255, 0.95);
      backdrop-filter: blur(10px);
      border-radius: 16px;
      padding: 25px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
      border: 1px solid rgba(255, 255, 255, 0.2);

      :deep(.el-table) {
        background: transparent;
        
        th {
          background: rgba(102, 126, 234, 0.05);
          color: #2c3e50;
          font-weight: 600;
          border-bottom: 1px solid rgba(102, 126, 234, 0.1);
        }

        td {
          border-bottom: 1px solid rgba(102, 126, 234, 0.05);
        }

        tr:hover > td {
          background: rgba(102, 126, 234, 0.03);
        }
      }

      .file-item {
        display: flex;
        align-items: center;
        padding: 8px 0;
        transition: all 0.3s ease;

        &:hover {
          transform: translateX(5px);
        }

        .file-name {
          flex: 1;
          margin-left: 10px;
          color: #2c3e50;
          font-size: 14px;
          cursor: pointer;
          transition: all 0.3s ease;

          &:hover {
            color: #409EFF;
          }
        }

        .op {
          width: 170px;
          display: flex;
          gap: 10px;
          
          span {
            padding: 6px 12px;
            border-radius: 6px;
            transition: all 0.3s ease;
            cursor: pointer;
            font-size: 13px;
            background: rgba(102, 126, 234, 0.05);
            color: #667eea;
            
            &:hover {
              background: rgba(102, 126, 234, 0.1);
              color: #764ba2;
              transform: translateY(-2px);
            }
          }
        }
      }
    }
  }
}

// 动画效果
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.share-panel {
  animation: fadeIn 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.file-list {
  animation: fadeIn 0.6s cubic-bezier(0.4, 0, 0.2, 1) 0.2s both;
}
</style>

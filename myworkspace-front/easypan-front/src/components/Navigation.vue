<template>
    <div class="top-navigation">
        <div class="nav-container">
            <template v-if="folderList.length > 0">
                <span class="back link" @click="backParent">
                    <span class="back-icon"></span>
                    返回上一级
                </span>
                <el-divider direction="vertical"></el-divider>
            </template>
            <span v-if="folderList.length == 0" class="all-file">全部文件</span>
            <span
              v-if="folderList.length > 0"
              class="link home-link"
              @click="setCurrentFolder(-1)"
            >
              <span class="home-icon"></span>
              全部文件
            </span>
            <template v-for="(item, index) in folderList">
                <span class="nav-arrow">
                    <svg width="16" height="16" viewBox="0 0 24 24">
                        <path fill="currentColor" d="M9.29 6.71a.996.996 0 0 0 0 1.41L13.17 12l-3.88 3.88a.996.996 0 1 0 1.41 1.41l4.59-4.59a.996.996 0 0 0 0-1.41L10.7 6.7c-.38-.38-1.02-.38-1.41.01z"/>
                    </svg>
                </span>
                <span
                  class="link folder-link"
                  v-if="index < folderList.length - 1"
                  @click="setCurrentFolder(index)"
                >
                  <span class="folder-icon"></span>
                  {{ item.fileName }}
                </span>
                <span v-if="index == folderList.length - 1" class="text current-folder">
                    <span class="folder-icon"></span>
                    {{ item.fileName }}
                </span>
            </template>
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance, nextTick, watch } from "vue";
const { proxy } = getCurrentInstance();
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
const route = useRoute();

const props = defineProps({
    watchPath: {
        type: Boolean,
        default: true,
    },
    shareId: {
        type: String,
    },
    adminShow: {
        type: Boolean,
        default: false,
    },
});

const api = {
    getFolderInfo: "/file/getFolderInfo",
    getFolderInfo4Share: "/showShare/getFolderInfo",
    getFolderInfo4Admin: "/admin/getFolderInfo",
};

// 分类
const category = ref();
// 目录集合
const folderList = ref([]);
// 当前目录
const currentFolder = ref({fileId: "0"});

const init = () => {
    folderList.value = [];
    currentFolder.value = {fileId: "0"};
    doCallback();
};

const openFolder = (data) => {
    const { fileId, fileName } = data;
    const folder = {
        fileName: fileName,
        fileId: fileId,
    };
    folderList.value.push(folder);
    currentFolder.value = folder;
    setPath();
};
defineExpose({ openFolder });

// 返回上一级
const backParent = () => {
    let currentIndex = null;
    for (let i = 0; i < folderList.value.length; i++) {
        if (folderList.value[i].fileId == currentFolder.value.fileId) {
            currentIndex = i;
            break;
        }
    }
    setCurrentFolder(currentIndex - 1);
};
// 点击导航  设置当前目录
const setCurrentFolder = (index) => {
    if (index == -1) {
      // 返回全部
      currentFolder.value = { fileId: "0" };
      folderList.value = [];
    } else {
        currentFolder.value = folderList.value[index];
        folderList.value.splice(index + 1, folderList.value.length);
    }
    setPath();
};

const setPath = () => {
    if (!props.watchPath) {
        // TODO 设置不监听路由回调方法
        doCallback();
        return;
    }
    let pathArray = [];
    folderList.value.forEach(item => {
        pathArray.push(item.fileId);
    })
    router.push({
        path: route.path,
        query: pathArray.length == 0 ? "" : { path: pathArray.join("/") },
    });
};

// 获取当前路径的目录
const getNavigationFolder = async (path) => {
    let url = api.getFolderInfo;
    if (proxy.shareId) {
        url = api.getFolderInfo4Share;
    }
    if (props.adminShow) {
        url = api.getFolderInfo4Admin;
    }
    let result = await proxy.Request({
        url: url,
        showLoading: false,
        params: {
            path: path,
            shareId: props.shareId,
        },
    });
    if (!result) {
        return;
    }
    folderList.value = result.data;
};

const emit = defineEmits(["navChange"]);
const doCallback = () => {
    emit("navChange", {
        categoryId: category.value,
        curFolder: currentFolder.value,
    });
};

watch(
    () => route,
    (newVal, oldVal) => {
        if (!props.watchPath) {
            return;
        }
        if (newVal.path.indexOf("/main") === -1 &&
            newVal.path.indexOf("/settings/fileList") === -1 &&
            newVal.path.indexOf("/share") === -1
        ) {
            return;
        }
        const path = newVal.query.path;
        category.value = newVal.params.category;
        if (path == undefined) {
            init();
        } else {
            getNavigationFolder(path);
            let pathArray = path.split("/");
            currentFolder.value = {
                fileId: pathArray[pathArray.length - 1],
            };
            doCallback();
        }
    },
    { immediate: true, deep: true }
);
</script>

<style lang="scss" scoped>
.top-navigation {
    padding: 8px 0;
    
    .nav-container {
        font-size: 14px;
        display: flex;
        align-items: center;
        line-height: 40px;
        background: linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.8) 100%);
        backdrop-filter: blur(10px);
        border-radius: 12px;
        padding: 4px 16px;
        box-shadow: 
            0 4px 6px -1px rgba(0, 0, 0, 0.1),
            0 2px 4px -1px rgba(0, 0, 0, 0.06),
            inset 0 0 0 1px rgba(255, 255, 255, 0.5);
        position: relative;
        overflow: hidden;

        &::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 1px;
            background: linear-gradient(90deg, 
                rgba(255, 255, 255, 0) 0%,
                rgba(255, 255, 255, 0.8) 50%,
                rgba(255, 255, 255, 0) 100%);
        }
    }
    
    .all-file {
        font-weight: 600;
        color: #2c3e50;
        position: relative;
        padding: 8px 16px 8px 40px;
        border-radius: 8px;
        background: rgba(64, 158, 255, 0.1);
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        
        &::before {
            content: '';
            position: absolute;
            left: 12px;
            top: 50%;
            transform: translateY(-50%);
            width: 20px;
            height: 20px;
            background: #409EFF;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M3 3h18v18H3V3zm16 16V5H5v14h14zM7 7h10v2H7V7zm0 4h10v2H7v-2zm0 4h7v2H7v-2z'/%3E%3C/svg%3E") no-repeat center;
            mask-size: contain;
            transition: all 0.3s ease;
        }

        &:hover::before {
            transform: translateY(-50%) scale(1.1);
        }
    }

    .back {
        display: inline-flex;
        align-items: center;
        padding: 8px 16px;
        border-radius: 8px;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        color: #606266;
        background: transparent;
        
        .back-icon {
            width: 20px;
            height: 20px;
            margin-right: 6px;
            background: currentColor;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z'/%3E%3C/svg%3E") no-repeat center;
            mask-size: contain;
            transition: transform 0.3s ease;
        }

        &:hover {
            background: rgba(64, 158, 255, 0.1);
            color: #409EFF;
            transform: translateX(-2px);

            .back-icon {
                transform: translateX(-2px);
            }
        }
    }

    .home-link, .folder-link {
        color: #606266;
        cursor: pointer;
        padding: 8px 16px 8px 40px;
        border-radius: 8px;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        position: relative;
        
        .home-icon, .folder-icon {
            position: absolute;
            left: 12px;
            top: 50%;
            transform: translateY(-50%);
            width: 20px;
            height: 20px;
            background: currentColor;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z'/%3E%3C/svg%3E") no-repeat center;
            mask-size: contain;
            transition: all 0.3s ease;
        }

        &:hover {
            color: #409EFF;
            background: rgba(64, 158, 255, 0.1);
            transform: translateX(2px);

            .home-icon, .folder-icon {
                transform: translateY(-50%) scale(1.1);
            }
        }
    }

    .current-folder {
        color: #2c3e50;
        font-weight: 500;
        padding: 8px 16px 8px 40px;
        position: relative;
        background: rgba(64, 158, 255, 0.1);
        border-radius: 8px;

        .folder-icon {
            position: absolute;
            left: 12px;
            top: 50%;
            transform: translateY(-50%);
            width: 20px;
            height: 20px;
            background: #409EFF;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M10 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V8c0-1.1-.9-2-2-2h-8l-2-2z'/%3E%3C/svg%3E") no-repeat center;
            mask-size: contain;
        }
    }

    .nav-arrow {
        color: #909399;
        padding: 0 4px;
        display: inline-flex;
        align-items: center;
        animation: fadeIn 0.3s ease;
    }

    :deep(.el-divider--vertical) {
        margin: 0 12px;
        height: 20px;
        border-color: rgba(144, 147, 153, 0.3);
    }
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateX(-10px);
    }
    to {
        opacity: 1;
        transform: translateX(0);
    }
}

// 添加响应式设计
@media screen and (max-width: 768px) {
    .top-navigation {
        .nav-container {
            padding: 4px 8px;
            font-size: 13px;
        }

        .back, .home-link, .folder-link, .current-folder {
            padding: 6px 12px 6px 32px;

            .back-icon, .home-icon, .folder-icon {
                left: 8px;
                width: 16px;
                height: 16px;
            }
        }

        .nav-arrow {
            padding: 0 2px;
        }

        :deep(.el-divider--vertical) {
            margin: 0 8px;
        }
    }
}
</style>

<template>
    <div class="share-container">
        <div class="top">
            <div class="top-op">
                <div class="left-buttons">
                    <el-button
                        type="danger"
                        :disabled="selectIdList.length == 0"
                        @click="cancelShareBatch"
                        class="cancel-btn"
                    >
                        <span class="iconfont icon-cancel"></span>
                        取消分享
                    </el-button>
                </div>
                <div class="right-buttons">
                    <div class="refresh-btn" @click="loadDataList">
                        <i class="iconfont icon-refresh"></i>
                    </div>
                </div>
            </div>
        </div>
        <div class="content-area" :class="{'empty': !tableData.list || tableData.list.length === 0}">
            <div class="file-list" v-if="tableData.list && tableData.list.length > 0">
                <Table
                    ref="dataTableRef"
                    :columns="columns"
                    :dataSource="tableData"
                    :fetch="loadDataList"
                    :initFetch="true"
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
                                <template v-if="(row.fileType == 3 || row.fileType == 1) && row.status !== 0">
                                    <Icon :cover="row.fileCover" :width="40"></Icon>
                                </template>
                                <template v-else>
                                    <Icon v-if="row.folderType == 0" :fileType="row.fileType" :width="40"></Icon>
                                    <Icon v-if="row.folderType == 1" :fileType="0" :width="40"></Icon>
                                </template>
                            </div>
                            <div class="file-info">
                                <span class="file-name" :title="row.fileName">
                                    {{ row.fileName }}
                                </span>
                            </div>
                            <div class="file-actions" v-if="row.showOp">
                                <el-tooltip content="复制链接" placement="top">
                                    <span class="iconfont action-icon icon-link" @click="copy(row)"></span>
                                </el-tooltip>
                                <el-tooltip content="取消分享" placement="top">
                                    <span class="iconfont action-icon icon-cancel" @click="cancelShare(row)"></span>
                                </el-tooltip>
                            </div>
                        </div>
                    </template>
                    <template #expireTime="{ index, row }">
                        <span :class="{'permanent': row.validType == 3}">
                            {{ row.validType == 3 ? "永久" : row.expireTime }}
                        </span>
                    </template>
                </Table>
            </div>
            <div class="no-data" v-else>
                <div class="no-data-inner">
                    <Icon iconName="no_data" :width="150" fit="fill"></Icon>
                    <div class="tips">暂无分享文件</div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import useClipboard from "vue-clipboard3";
const { toClipboard } = useClipboard();

import { ref, reactive, getCurrentInstance, nextTick } from "vue";
const { proxy } = getCurrentInstance();

const api = {
    loadDataList: "/share/loadShareList",
    cancelShare: "/share/cancelShare",
};

const columns = [
    {
        label: "文件名",
        prop: "fileName",
        scopedSlots: "fileName",
    },
    {
        label: "分享时间",
        prop: "shareTime",
        width: 200,
    },
    {
        label: "失效时间",
        prop: "expireTime",
        scopedSlots: "expireTime",
        width: 200,
    },
    {
        label: "浏览次数",
        prop: "showCount",
        width: 200,
    },
];

// 搜索
const search = () => {
    showLoading.value = true;
    loadDataList();
};

const tableData = ref({});
const tableOptions = {
    extHeight: 20,
    selectType: "checkbox",
};

const loadDataList = async () => {
    let params = {
        pageNo: tableData.value.pageNo,
        pageSize: tableData.value.pageSize,
    };
    let result = await proxy.Request({
        url: api.loadDataList,
        params,
    });
    if (!result) {
        return;
    }
    tableData.value = result.data;
};

// 多选 批量选择
const selectIdList = ref([]);
const rowSelected = (rows) => {
    selectIdList.value = [];
    rows.forEach((item) => {
        selectIdList.value.push(item.shareId);
    });
};

const showOp = (row) => {
    tableData.value.list.forEach((item) => {
        item.showOp = false;
    });
    row.showOp = true;
};

const cancelShowOp = (row) => {
    row.showOp = false;
};
// 复制链接
const shareUrl = ref(document.location.origin + "/share/");
const copy = async (data) => {
    await toClipboard(
      `链接:${shareUrl.value}${data.shareId} 提取码:${data.code}`
    );
    proxy.Message.success("复制成功");
};

// 取消分享
const cancelShareIdList = ref([]);
const cancelShareBatch = () => {
    if (selectIdList.value.length == 0) {
        return;
    }
    cancelShareIdList.value = selectIdList.value;
    cancelShareDone();
};

const cancelShare = (row) => {
    cancelShareIdList.value = [row.shareId];
    cancelShareDone();
};

const cancelShareDone = () => {
    proxy.Confirm(`你确定要取消分享吗?`, async () => {
        let result = await proxy.Request({
            url: api.cancelShare,
            params: {
                shareIds: cancelShareIdList.value.join(","),
            },
        });
        if (!result) {
            return;
        }
        proxy.Message.success("取消分享成功");
        loadDataList();
    });
};
</script>

<style lang="scss" scoped>
.share-container {
    padding: 20px;
    height: 100%;
    background-color: #f5f7fa;
    
    .top {
        .top-op {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
            
            .left-buttons {
                .cancel-btn {
                    background-color: #f56c6c;
                    border-color: #f56c6c;
                    &:hover {
                        background-color: #f78989;
                        border-color: #f78989;
                    }
                }
            }
            
            .right-buttons {
                .refresh-btn {
                    cursor: pointer;
                    padding: 8px;
                    border-radius: 4px;
                    background-color: #fff;
                    box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1);
                    transition: all 0.3s;
                    
                    &:hover {
                        background-color: #f5f7fa;
                        transform: rotate(180deg);
                    }
                    
                    i {
                        font-size: 18px;
                        color: #606266;
                    }
                }
            }
        }
    }
    
    .content-area {
        background-color: #fff;
        border-radius: 8px;
        box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1);
        padding: 20px;
        min-height: calc(100vh - 200px);
        
        &.empty {
            display: flex;
            justify-content: center;
            align-items: center;
        }
    }
    
    .file-list {
        .file-item {
            display: flex;
            align-items: center;
            padding: 10px;
            border-radius: 4px;
            transition: all 0.3s;
            
            &:hover {
                background-color: #f5f7fa;
            }
            
            .file-icon {
                margin-right: 15px;
            }
            
            .file-info {
                flex: 1;
                .file-name {
                    font-size: 14px;
                    color: #303133;
                }
            }
            
            .file-actions {
                display: flex;
                gap: 15px;
                
                .action-icon {
                    font-size: 18px;
                    color: #606266;
                    cursor: pointer;
                    transition: all 0.3s;
                    
                    &:hover {
                        color: #409eff;
                    }
                }
            }
        }
    }
    
    .no-data {
        text-align: center;
        padding: 50px 0;
        
        .tips {
            margin-top: 20px;
            color: #909399;
            font-size: 14px;
        }
    }
    
    .permanent {
        color: #67c23a;
        font-weight: 500;
    }
}
</style>

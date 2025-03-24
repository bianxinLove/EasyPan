<template>
    <div>
        <el-dialog
            :show-close="showClose"
            :draggable="true"
            :model-value="show"
            :close-on-click-modal="false"
            :title="title"
            class="cust-dialog"
            :top="top + 'px'"
            :width="width"
            @close="close"
        >
        <div
            class="dialog-body"
            :style="{ 'max-height': maxHeight + 'px', padding: padding + 'px' }"
        >
            <slot></slot>
    </div>
    <template v-if="(buttons && buttons.length > 0) || showCancel">
        <div class="dialog-footer">
            <el-button  link @click="close" v-if="showCancel"> 取消 </el-button>
            <el-button
                class="dialog-buttons"
                v-for="btn in buttons"
                :type="btn.type || 'primary' "
                @click="btn.click"
            >
                {{ btn.text }}
        </el-button>
        </div>
    </template>
    </el-dialog>
    </div>
</template>

<script setup>
const props = defineProps ({
    title: {
        type: String,
    },
    show: {
        type: Boolean,
        default: false,
    },
    showClose: {
        type: Boolean,
        default: true,
    },
    showCancel: {
        type: Boolean,
        default: true,
    },
    top: {
        type: Number,
        default: 50,
    },
    width: {
        type: String,
        default: "30%",
    },
    buttons: {
        type: Array,
    },
    padding: {
        type: Number,
        default: 15,
    },
});

const maxHeight = window.innerHeight - props.top - 100;

const emit = defineEmits();
const close = () => {
    emit("close");
};
</script>

<style lang="scss" scoped>
.dialog-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 999;
    backdrop-filter: blur(4px);
    animation: fadeIn 0.3s ease;

    .dialog-box {
        background: #fff;
        border-radius: 16px;
        width: 90%;
        max-width: 500px;
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
        animation: slideIn 0.3s ease;
        overflow: hidden;
        position: relative;

        .dialog-header {
            padding: 20px 24px;
            border-bottom: 1px solid #f0f0f0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #fafafa;

            .dialog-title {
                font-size: 1.25rem;
                font-weight: 600;
                color: #333;
                margin: 0;
            }

            .dialog-close {
                cursor: pointer;
                width: 32px;
                height: 32px;
                display: flex;
                align-items: center;
                justify-content: center;
                border-radius: 50%;
                transition: all 0.3s ease;

                &:hover {
                    background: #f0f0f0;
                    transform: rotate(90deg);
                }

                .iconfont {
                    font-size: 1.25rem;
                    color: #909399;
                }
            }
        }

        .dialog-body {
            padding: 24px;
            max-height: 70vh;
            overflow-y: auto;
            position: relative;
        }

        .dialog-footer {
            padding: 16px 24px;
            border-top: 1px solid #f0f0f0;
            display: flex;
            justify-content: flex-end;
            gap: 12px;
            background: #fafafa;
            position: sticky;
            bottom: 0;
            z-index: 1;

            .dialog-button {
                padding: 8px 24px;
                border-radius: 6px;
                font-size: 0.875rem;
                transition: all 0.3s ease;
                cursor: pointer;
                border: none;
                outline: none;
                min-width: 80px;
                display: flex;
                align-items: center;
                justify-content: center;

                &.cancel-button {
                    background: #f5f7fa;
                    color: #606266;
                    border: 1px solid #dcdfe6;

                    &:hover {
                        background: #e4e7ed;
                        border-color: #c0c4cc;
                    }
                }

                &.confirm-button {
                    background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
                    color: #fff;
                    border: none;
                    font-weight: 500;
                    box-shadow: 0 2px 6px rgba(64, 158, 255, 0.2);

                    &:hover {
                        background: linear-gradient(135deg, #66b1ff 0%, #409eff 100%);
                        transform: translateY(-1px);
                        box-shadow: 0 4px 12px rgba(64, 158, 255, 0.3);
                    }

                    &:active {
                        transform: translateY(0);
                        box-shadow: 0 2px 4px rgba(64, 158, 255, 0.2);
                    }
                }
            }
        }
    }
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

@keyframes slideIn {
    from {
        transform: translateY(-20px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

// 滚动条样式
.dialog-body::-webkit-scrollbar {
    width: 6px;
}

.dialog-body::-webkit-scrollbar-thumb {
    background: #dcdfe6;
    border-radius: 3px;
}

.dialog-body::-webkit-scrollbar-track {
    background: #f5f7fa;
    border-radius: 3px;
}

// 移动端样式优化
@media screen and (max-width: 768px) {
    .dialog-container {
        .dialog-box {
            width: 100%;
            height: 100%;
            max-width: none;
            border-radius: 0;
            margin: 0;
            display: flex;
            flex-direction: column;
            position: fixed;
            bottom: 0;
            left: 0;
            animation: slideUp 0.3s ease;

            .dialog-header {
                padding: 16px 20px;
                background: #fff;
                border-bottom: 1px solid #f0f0f0;
                position: relative;

                .dialog-title {
                    font-size: 1.1rem;
                    text-align: center;
                    width: 100%;
                    padding-right: 32px;
                }

                .dialog-close {
                    position: absolute;
                    right: 12px;
                    top: 50%;
                    transform: translateY(-50%);
                }
            }

            .dialog-body {
                flex: 1;
                padding: 20px;
                max-height: none;
                overflow-y: auto;
                -webkit-overflow-scrolling: touch;
            }

            .dialog-footer {
                padding: 12px 20px;
                background: #fff;
                border-top: 1px solid #f0f0f0;
                position: relative;
                bottom: 0;
                left: 0;
                right: 0;

                .dialog-button {
                    flex: 1;
                    height: 44px;
                    font-size: 1rem;
                    border-radius: 8px;

                    &.confirm-button {
                        background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
                        box-shadow: 0 4px 12px rgba(64, 158, 255, 0.2);
                    }
                }
            }
        }
    }
}

@keyframes slideUp {
    from {
        transform: translateY(100%);
    }
    to {
        transform: translateY(0);
    }
}

// 适配刘海屏
@supports (padding-top: env(safe-area-inset-top)) {
    .dialog-container {
        .dialog-box {
            padding-top: env(safe-area-inset-top);
            padding-bottom: env(safe-area-inset-bottom);
        }
    }
}
</style>
<template>
    <div :style="{ width: width + 'px', height: width + 'px' }" class="icon-wrapper">
        <div class="icon-container">
            <img 
                :src="getImage()" 
                :style="{ 'object-fit': fit }" 
                class="icon-image"
                :class="{'is-cover': !!cover}"
                @error="handleImageError"
            />
        </div>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance } from "vue";
const { proxy } = getCurrentInstance();

const props = defineProps({
    fileType: {
        type: Number,
    },
    iconName: {
        type: String,
    },
    cover: {
        type: String,
    },
    width: {
        type: Number,
        default: 32,
    },
    fit: {
        type: String,
        default: "cover",
    },
});

const fileTypeMap = {
    0: { desc: "目录", icon: "folder" },
    1: { desc: "视频", icon: "video" },
    2: { desc: "音频", icon: "music" },
    3: { desc: "图片", icon: "image" },
    4: { desc: "exe", icon: "pdf" },
    5: { desc: "doc", icon: "word" },
    6: { desc: "excel", icon: "excel" },
    7: { desc: "纯文本", icon: "txt" },
    8: { desc: "程序", icon: "code" },
    9: { desc: "压缩包", icon: "zip" },
    10: { desc: "其他文件", icon: "others" },
};

const getImage = () => {
    if (props.cover) {
        return proxy.globalInfo.imageUrl + props.cover;
    }
    let icon = "unknow_icon";
    if (props.iconName) {
        icon = props.iconName;
    } else {
        const iconMap = fileTypeMap[props.fileType];
        if (iconMap != undefined) {
            icon = iconMap["icon"];
        }
    }
    return new URL(`/src/assets/icon-image/${icon}.png`, import.meta.url).href;
};

const handleImageError = (e) => {
    // 如果是封面图加载失败，使用默认图片图标
    if (props.cover) {
        e.target.src = new URL(`/src/assets/icon-image/image.png`, import.meta.url).href;
    }
};
</script>

<style lang="scss" scoped>
.icon-wrapper {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    
    .icon-container {
        width: 100%;
        height: 100%;
        border-radius: 4px;
        overflow: hidden;
        background: #f5f7fa;
        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.3s ease;
        
        &:hover {
            background: #e4e7ed;
        }
        
        .icon-image {
            width: 100%;
            height: 100%;
            object-fit: contain;
            padding: 2px;
            
            &.is-cover {
                padding: 0;
                object-fit: cover;
            }
        }
    }
}
</style>
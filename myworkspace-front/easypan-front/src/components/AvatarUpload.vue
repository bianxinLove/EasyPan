<template>
    <div class="avatar-upload">
        <div class="avatar-show" @click="triggerUpload">
            <template v-if="localFile">
                <img :src="localFile" class="avatar-image" />
            </template>
            <template v-else>
                <img
                    :src="`${modelValue.qqAvatar}`"
                    v-if="modelValue && modelValue.qqAvatar"
                    class="avatar-image"
                />
                <img :src="`/api/getAvatar/${modelValue.userId}`" v-else class="avatar-image" />
            </template>
            <div class="avatar-overlay">
                <i class="iconfont icon-camera"></i>
            </div>
        </div>
        <div class="upload-tips">
            <p>支持 jpg、png 格式，最大 5MB</p>
        </div>
        <el-upload
            ref="uploadRef"
            name="file"
            :show-file-list="false"
            accept=".png,.PNG,.jpg,.JPG,.jpeg,.JPEG"
            :multiple="false"
            :http-request="uploadImage"
            class="hidden-upload"
        >
            <el-button class="select-button" type="primary">选择图片</el-button>
        </el-upload>
    </div>
</template>

<script setup>
import { ref, reactive, getCurrentInstance } from "vue";
import { useRouter, useRoute } from "vue-router";
const { proxy } = getCurrentInstance();
const router = useRoute();
const route = useRoute();
const uploadRef = ref(null);

const timestamp = ref("");

const props = defineProps({
    modelValue: {
        type: Object,
        default: null,
    },
});

const localFile = ref(null);
const emit = defineEmits();

const triggerUpload = () => {
    uploadRef.value.$el.querySelector('input').click();
};

const uploadImage = async (file) => {
    file = file.file;
    // 验证文件类型
    if (!['image/jpeg', 'image/png'].includes(file.type)) {
        proxy.Message.error('请上传 jpg 或 png 格式的图片');
        return;
    }
    // 验证文件大小（5MB）
    if (file.size > 5 * 1024 * 1024) {
        proxy.Message.error('图片大小不能超过 5MB');
        return;
    }
    
    let img = new FileReader();
    img.readAsDataURL(file);
    img.onload = ({ target }) => {
        localFile.value = target.result;
    };
    emit("update:modelValue", file);
};
</script>

<style lang="scss" scoped>
.avatar-upload {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    
    .avatar-show {
        position: relative;
        width: 150px;
        height: 150px;
        border-radius: 50%;
        overflow: hidden;
        cursor: pointer;
        background: #f5f5f5;
        transition: all 0.3s ease;
        
        &:hover {
            transform: scale(1.02);
            
            .avatar-overlay {
                opacity: 1;
            }
        }
        
        .avatar-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        .avatar-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            opacity: 0;
            transition: opacity 0.3s ease;
            
            .iconfont {
                font-size: 2rem;
                color: white;
            }
        }
    }
    
    .upload-tips {
        font-size: 0.875rem;
        color: #666;
        text-align: center;
    }
    
    .hidden-upload {
        display: none;
    }
    
    .select-button {
        background-color: #409eff;
        border-color: #409eff;
        color: #fff;
        padding: 8px 20px;
        border-radius: 4px;
        transition: all 0.3s ease;
        
        &:hover {
            background-color: #66b1ff;
            border-color: #66b1ff;
            transform: translateY(-1px);
            box-shadow: 0 2px 8px rgba(64, 158, 255, 0.2);
        }
    }
}
</style>
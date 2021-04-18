<template>
    <!-- {{files}}
  <file-upload
    ref="upload"
    v-model="files"
    :multiple=true
    post-action="/post.method"
    put-action="/put.method"
    @input-file="inputFile"
    @input-filter="inputFilter"
  >
  上传文件
  </file-upload>
  <el-button @click="addText">添加文件</el-button>
  <el-button @click="$refs.upload.active = true">上传</el-button>
  <el-button @click="$refs.upload.active = false">停止上传</el-button> -->
    <div class="filcom">
        <ul class="pre">
            <li v-for="file in files" 
                :key="file" 
                :class="{
                    uploaderror: file.error && !file.active
                }">
                <div class="progress" v-if="file.progress != 0 && file.active">
                    <el-progress 
                        :text-inside="true" 
                        :stroke-width="4" 
                        :percentage="file.progress" 
                        status="exception">
                    </el-progress>
                </div>
                <div class="content">
                    <img v-if="imgReg.test(getUrlLastName(file.name))" style="max-height: 100%" :src="file.blob"/>
                    <span v-else class="showtitle">
                        {{file.name}}
                    </span>
                    <!-- <a target="_blank" :href="file.blob">{{file.name}}</a> -->
                    <!-- <button v-show="!file.active" type="button" @click.prevent="onStart(file)">开始上传</button>
                    <button v-show="file.active" type="button" @click.prevent="onStop(file)">停止上传</button> -->
                </div>
                <span class="del" @click.prevent="onRemove(file)">
                    <i class="el-icon-delete"></i>
                </span>
            </li>
        </ul> 
        <div class="file-upload-content" style="cursor:pointer">
            <span>
                <i class="el-icon-plus"></i>
                <div>Upload</div>
            </span>
            <file-upload
                v-bind="$attrs"
                class="fileinput"
                ref="uploadRef"
                post-action="/post.method"
                put-action="/put.method"
                v-model="files"
                :multiple="true"
                @input-file="inputFile"
                @input-filter="inputFilter"
            >
            </file-upload>
        </div>
    </div>
  
    <span v-show="$refs.upload && $refs.upload.uploaded">全部文件已上传完毕</span>
    <el-button @click="$refs.uploadRef.active = true">全部上传</el-button>
    <el-button @click="onAdd('https://sf3-scmcdn2-tos.pstatp.com/xitu_juejin_web/img/logo.a7995ad.svg')">添加文件</el-button>
    
    <!-- <pre>
        {{files}}
    </pre> -->
</template>



<script lang="ts">
import { defineComponent, ref, watchEffect, watch, toRefs, computed } from "vue";
import VueUploadComponent from 'vue-upload-component'

// 验证是否是图片
const imgReg = /\.(jpeg|jpe|jpg|gif|png|webp|svg)$/;

export default defineComponent({
    components: {
        FileUpload: VueUploadComponent
    },
    props: {
        modelValue: {
            type: Boolean
        }
    },
    emits: ['update:modelValue'],
    setup(props, ctx) {
        // upload ref 
        const uploadRef = ref<typeof VueUploadComponent>();
        const uploaded = computed(() => {
            return uploadRef.value?.updated
        })
        // 监听是否完全上传成功
        watch(() => uploaded.value, (newVal, oldVal) => {
            console.log(newVal, oldVal)
            ctx.emit('update:modelValue', newVal)
        }, {
            immediate: true
        })
        // 文件列表
        const files = ref<any[]>([]);
        // 添加方法
        const onAdd = async (file: any) => {
            if (typeof file === 'string') {
                urlToBlob(file).then(blob => {
                    if (uploadRef.value) {
                        uploadRef.value.add(blob);
                    }
                })
            }
            // uploadRef.value.add(file);
        }
        // 删除方法
        const onRemove = (file: any) => {
            if (!uploadRef.value) return;
            uploadRef.value.remove(file);
        }
        // 更新方法
        const onUpdate = (file: any) => {}
        // 单个文件停止上传
        const onStart = (file: any) => {
            if (!uploadRef.value) return;
            uploadRef.value.update(file, {active: true})
        }
        // 单个文件开始上传
        const onStop = (file: any) => {
            if (!uploadRef.value) return;
            uploadRef.value.update(file, {active: false})
        }
        // 获取地址名称
        function getUrlLastName(url: string) {
            const urlArr = url.split('/');
            return urlArr[urlArr.length - 1];
        }
        // 将url地址转换为blob对象添加到files中
        function urlToBlob(url: string) {
            return new Promise((resolve, reject) => {
                let image = new Image();
                image.setAttribute('crossOrigin', 'anonymous');
                image.src = url;
                console.log(image)
                image.onload = () => {
                    let canvas = document.createElement('canvas')
                    canvas.width = image.width
                    canvas.height = image.height
                    let ctx: any = canvas.getContext('2d');
                    ctx.drawImage(image, 0, 0, image.width, image.height)
                    canvas.toBlob((blob: any) => {
                        // let url = URL.createObjectURL(blob);
                        console.log(blob)
                        blob.name = getUrlLastName(url)
                        resolve(blob)
                        // 用完释放URL对象
                        // URL.revokeObjectURL(url)
                    })
                }
            })
        }
        /**
         * @添加，更新，移除 后执行函数
         * Has changed
         * @param  Object|undefined   newFile   只读
         * @param  Object|undefined   oldFile   只读
         * @return undefined
         */
        const inputFile = (newFile: any, oldFile: any) => {
            if (!uploadRef.value) return; 
            if (newFile && !oldFile) {
                // 添加文件
            }

            if (newFile && oldFile) {
                // 更新文件

                // 开始上传
                if (newFile.active !== oldFile.active) {
                    console.log('Start upload', newFile.active, newFile)

                    // 限定最小字节
                    if (newFile.size >= 0 && newFile.size < 100 * 1024) {
                        newFile = uploadRef.value.update(newFile, {error: 'size'})
                    }
                }

                // 上传进度
                if (newFile.progress !== oldFile.progress) {
                    console.log('progress', newFile.progress, newFile)
                }

                // 上传错误
                if (newFile.error !== oldFile.error) {
                    console.log('error', newFile.error, newFile)
                }

                // 上传成功
                if (newFile.success !== oldFile.success) {
                    console.log('success', newFile.success, newFile)
                }
            }

            if (!newFile && oldFile) {
                // 删除文件
                // 自动删除 服务器上的文件
                if (oldFile.success && oldFile.response.id) {
                // $.ajax({
                //   type: 'DELETE',
                //   url: '/file/delete?id=' + oldFile.response.id,
                // });
                }
            }
            // 自动上传
            if (Boolean(newFile) !== Boolean(oldFile) || oldFile.error !== newFile.error) {
                if (uploadRef.value.active) {
                    uploadRef.value.active = true
                }
            }
        }
            /**
         * Pretreatment
         * @param  Object|undefined   newFile   读写
         * @param  Object|undefined   oldFile   只读
         * @param  Function           prevent   阻止回调
         * @return undefined
         */
        const inputFilter = (newFile: any, oldFile: any, prevent: any) => {
            if (!uploadRef.value) return; 
            if (newFile && !oldFile) {
                // 添加文件

                // 过滤非图片文件
                // 不会添加到 files 去
                // if (!/\.(jpeg|jpe|jpg|gif|png|webp)$/i.test(newFile.name)) {
                //     return prevent()
                // }

                // 创建 `blob` 字段 用于缩略图预览
                newFile.blob = ''
                let URL = window.URL || window.webkitURL
                if (URL && URL.createObjectURL) {
                newFile.blob = URL.createObjectURL(newFile.file)
                }
            }

            if (newFile && oldFile) {
                // 更新文件

                // 增加版本号
                if (!newFile.version) {
                newFile.version = 0
                }
                newFile.version++
            }

            if (!newFile && oldFile) {
                // 移除文件

                // 拒绝删除文件
                // return prevent()
            }
        }
        // 上传方法

        return {
            uploadRef,
            files,
            onAdd,
            onRemove,
            onUpdate,
            onStart,
            onStop,
            inputFile,
            inputFilter,
            urlToBlob,
            getUrlLastName,
            imgReg
        }

    }
})

// export default defineComponent({
//   data: function () {
//     return {
//       files: []
//     }
//   },
//   components: {
//     FileUpload: VueUploadComponent
//   },
//   methods: {
//       addText() {
//       let file = new window.File(['foo'], 'foo.txt', {
//         type: "text/plain",
//       })
//       this.$refs.upload.add(file)
//     },
//     async customAction(file, component) {
//         // return await component.uploadPut(file)
//         return await component.uploadHtml4(file)
//     },
//     /**
//      * Has changed
//      * @param  Object|undefined   newFile   只读
//      * @param  Object|undefined   oldFile   只读
//      * @return undefined
//      */
//     inputFile: function (newFile, oldFile) {
//       if (newFile && oldFile && !newFile.active && oldFile.active) {
//         // 获得相应数据
//         console.log('response', newFile.response)
//         if (newFile.xhr) {
//           //  获得响应状态码
//           console.log('status', newFile.xhr.status)
//         }
//       }
//     },
//     /**
//      * Pretreatment
//      * @param  Object|undefined   newFile   读写
//      * @param  Object|undefined   oldFile   只读
//      * @param  Function           prevent   阻止回调
//      * @return undefined
//      */
//     inputFilter: function (newFile, oldFile, prevent) {
//       if (newFile && !oldFile) {
//         // 过滤不是图片后缀的文件
//         if (!/\.(jpeg|jpe|jpg|gif|png|webp)$/i.test(newFile.name)) {
//           return prevent()
//         }
//       }

//       // 创建 blob 字段 用于图片预览
//       newFile.blob = ''
//       let URL = window.URL || window.webkitURL
//       if (URL && URL.createObjectURL) {
//         newFile.blob = URL.createObjectURL(newFile.file)
//       }
//     }
//   },
//   setup() {
    
//     return {
//     };
//   },
// });
</script>

<style scoped>
ul,li {
    padding: 0;
    margin: 0;
}
.file-upload-content {
    box-sizing: border-box;
    width: 128px;
    height: 128px;
    margin-right: 8px;
    margin-bottom: 8px;
    text-align: center;
    vertical-align: top;
    background-color: #fafafa;
    border: 1px dashed #d9d9d9;
    border-radius: 2px;
    transition: border-color .3s;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    border-radius: 4px;
}
.file-upload-content:hover {
    border-color: #1890ff;
}
.file-upload-content > span > i {
    font-size: 25px;
}
.fileinput {
    position: absolute;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    z-index: 99;
}
.filcom {
    display: flex;
}
.filcom > .pre {
    height: 128px;
    display: flex;
}
.filcom > .pre > li {    
    width: 128px;
    background-color: #fafafa;
    border: 1px solid #d9d9d9;
    display: flex;
    justify-content: center;
    align-items: center;
    box-sizing: border-box;
    margin-right: 10px;
    border-radius: 4px;
    position: relative;
    transition: all .3s;
}
.filcom > .pre > li:hover {
    border-color: #1890ff;
}
.filcom > .pre > li > div.content {
    display: flex;
    overflow: hidden;
    height: 100%;
    align-items: center;
}
.filcom > .pre > li > span.del {
    position: absolute;
    right: 4px;
    top: 4px;
    transition: all .3s;
    opacity: 0;
}
.filcom > .pre > li:hover > span.del {
    opacity: 1;
}
.filcom > .pre > li > span.del:hover {
    cursor: pointer;
    color: red;
}
.showtitle {
    color: #1890ff;
    text-decoration: none;
    background-color: transparent;
    outline: none;
    cursor: pointer;
    font-size: 14px;
    display: inline-block;
    padding: 10%;
    white-space:normal; word-break:break-all;
}
.uploaderror {
    animation: pulsate .5s ease-out;
    animation-iteration-count: infinite;
    animation-direction:alternate;
}


@keyframes pulsate {
	0% {
		border-color: #d9d9d9;
	}
	100% {
		border-color: #ff4d4f;
	}
} 
.progress {
    position: absolute;
    width: 100%;
    left: 0;
    top: 0;
}
</style>

<!--
基于quill-editor的整合Fastdfs上传的自定义组件
-->

<template>
    <div id='quillEditorFastdfs'>
        <!-- 基于elementUi的上传组件 el-upload begin-->
        <el-upload
                class="avatar-uploader"
                :action="uploadUrl"
                :accept="'image/*'"
                :data="FastdfsForm"
                :show-file-list="false"
                :on-success="uploadEditorSuccess"
                :on-error="uploadEditorError"
                :before-upload="beforeEditorUpload">
        </el-upload>
        <!-- 基于elementUi的上传组件 el-upload end-->
        <quill-editor  class="editor"  v-model="content" ref="customQuillEditor" :options="editorOption" >
        </quill-editor>
    </div>
</template>

<script>
    import { quillEditor } from 'vue-quill-editor'


    //自定义编辑器的工作条
    const toolbarOptions = [
        ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
        ['blockquote', 'code-block'],

        [{'header': 1}, {'header': 2}],               // custom button values
        [{'list': 'ordered'}, {'list': 'bullet'}],
        [{'script': 'sub'}, {'script': 'super'}],      // superscript/subscript
        [{'indent': '-1'}, {'indent': '+1'}],          // outdent/indent
        [{'direction': 'rtl'}],                         // text direction

        [{'size': ['small', false, 'large', 'huge']}],  // custom dropdown
        [{'header': [1, 2, 3, 4, 5, 6, false]}],

        [{'color': []}, {'background': []}],          // dropdown with defaults from theme
        [{'font': []}],
        [{'align': []}],
        ['link', 'image', 'video'],
        ['clean']                                         // remove formatting button
    ];
    export default {
        data(){
            return {
                quillUpdateImg:false,
                content:'',
                editorOption:{
                    placeholder:'请填写车辆详情信息',
                    modules: {
                        toolbar: {
                            container: toolbarOptions,  // 工具栏
                            handlers: {
                                'image': function (value) {
                                    if (value) {
                                        document.querySelector('#quillEditorFastdfs .avatar-uploader input').click()
                                    } else {
                                        this.quill.format('image', false);
                                    }
                                }
                            }
                        }
                    }
                },
                FastdfsForm:{
                    'host':''
                },
            }
        },
        props:{
            uploadUrl:String,  //从七牛云上拿到自己的上传地址,类型为String
            host:String  //fastdfs获取文件的主机
        },
        methods:{
            //上传图片之前
            beforeEditorUpload(res, file){

                //显示上传动画
                this.quillUpdateImg = true;
            },

            // 上传图片成功
            uploadEditorSuccess(res, file) {

                //拼接出上传的图片在服务器的完整地址
                let imgUrl = this.FastdfsForm.host+res.data;

                // 获取富文本组件实例
                let quill = this.$refs.customQuillEditor.quill;

                // 获取光标所在位置
                let length = quill.getSelection().index;

                // 插入图片  res.info为服务器返回的图片地址
                quill.insertEmbed(length, 'image', imgUrl)

                // 调整光标到最后
                quill.setSelection(length + 1)

                //取消上传动画
                this.quillUpdateImg = false;

            },

            // 上传图片失败
            uploadEditorError(res, file) {
                //页面提示
                Notification.error({
                    message: '上传图片失败'
                });

                //取消上传动画
                this.quillUpdateImg = false;
            },




        },
        mounted () {
            this.FastdfsForm.host = this.host;
        },
        watch:{
            content(newVal, oldVal) {
                this.$emit('input', newVal);
            }
        }



    }
</script>

<style scoped>
    .editor{
        min-height: 200px;
        margin-bottom: 60px;
    }
</style>
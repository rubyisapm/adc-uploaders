<template>
    <div :class="{'input-group':hasInput}" class="ct-adc-importer">
        <input v-if="hasInput" type="text" class="form-control input-sm" :placeholder="inputPlaceholder" :value="fileName">
        <div :class="{'input-group-addon':hasInput}">
            <span ref="root"
                  class="webuploader-container"
                  :class="disabled || loading ? 'disabledWrap' : ''"
                  :data-placement="direction"
                  :data-original-title="tip">
                <!--webuploader-container是后续上传插件往外层元素span上加入的类名，这里直接绑定到class上是为了避免后续dom渲染时保持span有该类-->
                <i class="glyphicon" :class="hasInput ? 'glyphicon-folder-open' : 'glyphicon-import'"></i>
                <span class="title">{{ buttonText }}</span>
            </span>
        </div>
    </div>
</template>

<script type="text/ecmascript-6">
    import utility from 'ct-utility';
    var ERRORS = {
        Q_EXCEED_NUM_LIMIT: '文件数量超出限制',
        Q_EXCEED_SIZE_LIMIT: '文件总大小超出限制!',
        Q_TYPE_DENIED: '文件类型不正确!'
    };
    export default {
        name: 'importer',
        props: {
            hasInput: {
                type: Boolean,
                default: false
            },
            buttonText: {
                type: String,
                default: '导入文件'
            },
            inputPlaceholder: {
                type: String,
                default: '请上传一个文件'
            },
            server: {
                type: String,
                default: ''
            },
            method: {
                type: String,
                default: 'GET'
            },
            accept: {
                type: Object,
                default(){
                    return {
                        extensions: '',
                        mimeTypes: ''
                    }
                }
            },
            fileSizeLimit: {
                type: Number
            },
            formData: {
                type: Object,
                default () {
                    return {}
                }
            },
            chunked: {
                type: Boolean,
                default: true
            },
            sendAsBinary: {
                type: Boolean,
                default: true
            },
            headers: {
                type: Object,
                default(){
                    return {};
                }
            },
            disabled: {
                type: Boolean,
                default: false
            },
            direction: {
                type: String,
                default: 'top'
            },
            tip: {
                type: String,
                default: ''
            }
        },
        data () {
            return {
                fileName: '',
                loading: false
            };
        },
        mounted () {
            this.initUploader();
            this.initTip();
            if (this.disabled) {
                this.disableUploader();
                this.enableTip();
            } else {
                this.enableUploader();
                this.disableTip();
            }
            this.refresh();
        },
        methods: {
            initUploader(){
                var that = this;
                that.uploader = WebUploader.create({
                    pick: {
                        id: that.$refs.root
                    },
                    auto: true,
                    chunked: that.chunked,
                    server: that.server,
                    method: that.method,
                    accept: that.accept,
                    fileSizeLimit: that.fileSizeLimit
                });
                that.uploader.on('uploadBeforeSend', function(object, data, headers) {
                    utility.base.extend(data, that.formData);
                    utility.base.extend(headers, that.headers);
                })
                that.uploader.on('uploadStart', function() {
                    that.$emit('start');
                    that.loading = true;
                });

                that.uploader.on('uploadSuccess', function(file, response) {
                    if(that.hasInput){
                        that.fileName=file.name;
                    }
                    that.$emit('success', response);
                });

                that.uploader.on('uploadError', function(file, reason) {
                    if(that.hasInput){
                        that.fileName='';
                    }
                    that.$emit('error', reason);
                });

                that.uploader.on('error', function(code) {
                    var msg = ERRORS[code] || '上传失败，请重试！';
                    that.loading = false;
                    if(that.hasInput){
                        that.fileName='';
                    }
                    that.$emit('error', msg);
                });

                that.uploader.on('uploadComplete', function() {
                    that.loading = false;
                    that.uploader.reset();
                })
            },
            refresh () {
                this.uploader.refresh();
            },
            stop(){
                this.loading = false;
                this.uploader.stop(true);
            },
            initTip(){
                $(this.$refs.root).tooltip({
                    title: this.tip,
                    placement: this.direction
                });
            },
            disableTip(){
                $(this.$refs.root).tooltip('disable');
            },
            enableTip(){
                $(this.$refs.root).tooltip('enable');
            },
            destroyTip(){
                $(this.$refs.root).tooltip('destroy');
            },
            disableUploader(){
                var classes = this.$refs.root.querySelector('.webuploader-pick').classList;
                if (!classes.contains('webuploader-pick-disable')) {
                    this.$refs.root.querySelector('.webuploader-pick').classList.add('webuploader-pick-disable');
                }
            },
            enableUploader(){
                var classes = this.$refs.root.querySelector('.webuploader-pick').classList;
                if (classes.contains('webuploader-pick-disable')) {
                    this.$refs.root.querySelector('.webuploader-pick').classList.remove('webuploader-pick-disable');
                }
            }
        },
        watch: {
            accept(){
                this.uploader.destroy();
                this.initUploader();
            },
            fileSizeLimit(){
                this.uploader.destroy();
                this.initUploader();
            },
            chunked(newVal){
                this.uploader.option('chunked', newVal);
            },
            sendAsBinary(newVal){
                this.uploader.option('sendAsBinary', newVal);
            },
            buttonText(newVal){
                var $webuploaderPick = this.$refs.root.querySelector('.webuploader-pick');
                $webuploaderPick.innerHTML = '<i class="glyphicon glyphicon-import"></i><span class="title">' + newVal + '</span>';
            },
            hasInput(newVal){
                var $webuploaderPick = this.$refs.root.querySelector('.webuploader-pick').querySelector('.glyphicon');
                if(newVal){
                    $webuploaderPick.classList.remove('glyphicon-import');
                    $webuploaderPick.classList.add('glyphicon-folder-open');
                }else{
                    $webuploaderPick.classList.remove('glyphicon-folder-open');
                    $webuploaderPick.classList.add('glyphicon-import');
                }
            },
            method(newVal){
                this.uploader.option('method', newVal);
            },
            loading(isLoading) {
                var $webuploaderPick = this.$refs.root.querySelector('.webuploader-pick');
                if (isLoading) {
                    this.disableUploader();
                    $webuploaderPick.innerHTML = '<i class="glyphicon glyphicon-refresh rotate"></i><span class="title">' + '上传中...' + '</span>';
                    $webuploaderPick.nextSibling.style.display = 'none';
                } else {
                    this.enableUploader();
                    $webuploaderPick.innerHTML = '<i class="glyphicon glyphicon-import"></i><span class="title">' + this.buttonText + '</span>';
                    $webuploaderPick.nextSibling.style.display = 'block';
                }
            },
            server: function(server) {
                this.uploader.option('server', server);
            },
            disabled(newVal){
                if (newVal) {
                    this.disableUploader();
                    this.enableTip();
                } else {
                    this.enableUploader();
                    this.disableTip();
                }
            },
            tip(newVal){
                if (newVal === '') {
                    this.disableTip();
                } else {
                    this.enableTip();
                }
            }
        },
        beforeDestroy(){
            this.destroyTip();
        }
    }
</script>

<style>
    .ct-adc-importer.webuploader-container {
        position: relative;
        display: inline-block;
    }

    .ct-adc-importer.disabledWrap label {
        position: absolute;
        z-index: -9999;
    }

    .ct-adc-importer .webuploader-element-invisible {
        position: absolute !important;
        clip: rect(1px 1px 1px 1px); /* IE6, IE7 */
        clip: rect(1px, 1px, 1px, 1px);
    }

    .ct-adc-importer .webuploader-pick {
        position: relative;
        display: inline-block;
        cursor: pointer;
        background: #00b7ee;
        padding: 5px 10px;
        color: #fff;
        text-align: center;
        border-radius: 3px;
        overflow: hidden;
        font-size: 12px;
        line-height: 1.5;
        margin-bottom: 0;
        font-weight: normal;
        vertical-align: middle;
        touch-action: manipulation;
        background-image: none;
        border: 1px solid transparent;
        white-space: nowrap;
        -ms-touch-action: manipulation;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    .ct-adc-importer .webuploader-pick-hover {
        background: #00a2d4;
    }

    .ct-adc-importer .webuploader-pick-disable {
        opacity: 0.6;
        pointer-events: none;
    }

    .ct-adc-importer .webuploader-pick .glyphicon {
        margin-right: 5px;
    }

    .ct-adc-importer .input-group-addon{
        padding:0;
        border:none;
    }
    .ct-adc-importer .input-group-addon .webuploader-pick{
        border-radius: 0 3px 3px 0;
    }
</style>
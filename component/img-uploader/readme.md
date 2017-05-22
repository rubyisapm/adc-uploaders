# 导入文件组件

## 使用

从npm安装`ct-adc-uploaders`
```
npm install ct-adc-uploaders
```
在代码中引用
```
import uploaders from 'ct-adc-uploaders';
var ImgUploader=uploaders.ImgUploader;
Vue.component(ImgUploader.name,ImgUploader);
```
## 参数说明

参数|描述|类型|是否必填|默认值
--- | --- | --- | --- |
thumbnailWidth | 生成缩略图的宽度 | Number | 否 | 110
thumbnailHeight | 生成缩略图的高度 | Number | 否 | 110
imgs | 图片列表，每项为一个图片路径 | Array | 否 |[]
server | 接口地址 | String | 是 | ''
resultFilter | 将响应数据处理为固定格式结果的过滤器，**详细见下方** | Function | 是 | new Function()
method | 上传图片的ajax请求类型 | String | 否 | 'post'
duplicate | 是否允许同样的图片上传两张 | Boolean | 否 | false
accept | 可接受的文件类型 | Object | 否 | **详细见下方**
fileSingleSizeLimit | 单个文件的大小限制，以Byte为单位 | Number | 否 | 2 * 1024 * 1024
fileNumLimit | 文件数量的限制 | Number | 否 | 5
formData | 上传图片时附加的表单数据 | Object | 否 | {}

### resultFilter

该参数用于开发者干预响应报文的解析，告诉组件什么样的数据是正确的。
如:
```
function(res){
    if (res.IsFinish) {
        return {
            status: true,
            path: res.ReturnPath
        };
    }
    return {
        status: false,
        msg: res.msg
    };
}
```
该函数接收服务端的响应数据，并解析出一个结果对象，该结果对象包含：

* status 上传是否成功

* path 上传成功后服务器返回的路径

* msg 上传失败时服务器返回的错误信息

### accept

该参数遵循webuploader的accept参数设置。

默认值为:
```
{
    extensions: 'gif,jpg,jpeg,bmp,png',
    mimeTypes: 'image/gif,image/jpg,image/jpeg,image/bmp,image/png'
}
```
请根据项目需要进行设置。

## 方法

### isPending

参数: 无

结果: Boolean

描述: 组件是否处于上传状态，即其中有没有正在上传的图片。

### getUploadedImgs

参数: 无

结果: Array

描述: 组件中已成功上传的文件，其中每一项为一个[file对象](#jump)

### getErrorImgs

参数: 无

结果: Array

描述: 组件中上传失败的文件，其中每一项为一个[file对象](#jump)

### getPendingImgs

参数: 无

结果: Array

描述: 组件中正在上传的文件，其中每一项为一个[file对象](#jump)

### getUrls

参数: 无

结果: Array

描述: 组件中上传成功的文件，其中每一项为一个图片在服务器上的路径

### refreshUploader

参数: 无

结果: undefined

描述: 刷新图片上传组件


### resetUploader

参数: 无

结果: undefined

描述: 重置图片上传组件

### cancelUpload

参数: 无

结果: undefined

描述: 取消还未上传成功的图片

## 其他

### <span id = "jump">file对象</span>

一个file对象包含以下内容:

* status

    - inited 初始状态
    - queued 已经进入队列, 等待上传
    - progress 上传中
    - complete 上传完成并逻辑上成功
    - error 上传出错，可重试
    - interrupt 上传中断，可续传
    - invalid 文件不合格，不能重试上传。会自动从队列中移除
    - cancelled 文件被移除

* errorText

文件上传失败或上传成功但逻辑上失败时，保存失败信息

* previewStatus

预览图片的生成状态

* previewSrc

预览图片生成的image data。

注：但是如果是外部传入的图片列表时，该值等于图片的真实路径(file.url)

* url

图片在服务器上的路径







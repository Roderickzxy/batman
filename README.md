# POLYV JAVA 视频上传 SDK
Polyv Java 上传 SDK 为您提供上传媒体文件到[保利威云点播平台](http://www.polyv.net/vod/)的开发工具包。


## 功能
- 快捷上传多种格式的媒体文件。
- 支持上传时的各种设置，如文件标题、描述、标签、上传目录、是否开启课件优化处理等。
- 采用分片并发上传的方式，支持**断点续传**。


## 使用方法

### 前提条件
1. 使用本 SDK 前，要先开通**保利威云点播服务**。如果您还不了解该服务，请登录产品主页查看，详见：[云点播](http://www.polyv.net/vod/)。
2. 获取 secretKey 等相关信息用于用户身份校验，您可以在「云点播管理后台 -> 设置 -> API接口」页面中找到相关信息，[点击这里登录后台](http://my.polyv.net/v2/login)。

### java版本支持
- Java 1.8及以上版本

### 集成 SDK
目前sdk还没上传到中央仓库，不支持通过maven配置pom文件添加依赖，只能将sdk项目构建出来的包引入项目中	

这里举个简单的例子：（在windows环境下运行）
1. 准备好项目：
```bash
    TestProject
    ├── src
    │   └── com
    │       └── polyv
    │           └── test
    │               └── TestMain.java
    ├── lib
    │   └── polyv-upload-java-sdk.jar
    └── class
```
2. 编写java程序：

```java	
package com.polyv.test;

import net.polyv.bean.vo.VideoInfo;
import net.polyv.entry.PolyvUploadClient;

/**
 * 续传的时候不能修改分片大小，不然会引起重传
 * 续传的时候标题，描述，分类，标签修改无效
 */
public class TestMain {
    public static void main(String[] args){
        PolyvUploadClient client = new PolyvUploadClient("<userid>", "<secretKey>", 100*1024, "checkpoint_location", 5);

        VideoInfo videoInfo = new VideoInfo();
        videoInfo.setFileLocation("视频路径");
        videoInfo.setCataId(1L);
        videoInfo.setDescrib("描述");
        System.out.println("vid="+client.uploadVideoParts(videoInfo, new UploadCallBack() {
            @Override
            public void start(String s) {
                System.out.println("start="+s);
            }

            @Override
            public void process(String s, long l, long l1) {
                System.out.println("process="+s+",uploaded="+l+", total="+l1);
            }

            @Override
            public void complete(String s) {
                System.out.println("complete="+s);
            }

            @Override
            public void success(String s) {
                System.out.println("success="+s);
            }

            @Override
            public void error(String s, UploadErrorMsg uploadErrorMsg) {
                System.out.println("error="+s+", message="+uploadErrorMsg.getMessage());
            }
        }, false));
    }
}
```
3. 编译java程序到class目录中：
```bash
javac -encoding utf-8  -d  ./class -classpath ./lib/polyv-upload-java-sdk.jar src/com/polyv/test/TestMain.java
```
4. 运行编译完的程序(cd到class目录)
```bash
java -Djava.ext.dirs="../lib/;%JAVA_HOME%/jre/lib/ext/" com.polyv.test.TestMain
```
可以看到打印了回调函数里打印的日志：
````
start=cca90d24f7dfc13602214ea5596012f7_c
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=102400, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=204800, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=307200, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=409600, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=512000, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=614400, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=716800, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=819200, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=892830, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=995230, total=1097630
process=cca90d24f7dfc13602214ea5596012f7_c,uploaded=1097630, total=1097630
complete=cca90d24f7dfc13602214ea5596012f7_c
success=cca90d24f7dfc13602214ea5596012f7_c
[Thread-0] INFO net.polyv.entry.PolyvUploadClient - upload success. cost 3118 ms
vid=cca90d24f7dfc13602214ea5596012f7_c

````

## 快速开始

首先，创建 上传client 实例。  
这里分别传入的参数是polyv用户id，用户的secretkey，分片大小（默认为1MB,大小限定为100KB~5GB），checkpoint文件夹路径，上传线程数
```java
PolyvUploadClient client = new PolyvUploadClient("<userid>", "<secretKey>", 100*1024, "checkpoint_location", 5);
```
创建 VideoInfo 实例。
```java
VideoInfo videoInfo = new VideoInfo();
videoInfo.setFileLocation("视频在本地的位置");
videoInfo.setCataId("分类id");//不填默认为根目录
videoInfo.setDescrib("描述");
videoInfo.setTitle("视频标题");
```
调用 `client.uploadVideoParts` 方法就开始上传视频了。

## sdk类和方法详解
#### 1. 入口类（PolyvUploadClient.java）
##### (1)构造方法：
PolyvUploadClient(String userId, String secretKey, int partitionSize, String checkpoint, int threadNum)  
参数信息：  

|Params|	Description|
| --- | --- |
|userId|	保利威账号id|
|secretKey|	保利威账号的secrekey|
|partitionSize|	分片大小，（单位：字节，默认为1MB,大小限定为100KB~5GB）|
|checkpoint|	断点续传的checkpoint文件夹路径（需要保证文件夹路径已经创建好）|
|threadNum|	上传处理线程数量|

##### (2)方法：
|Modifier and Type|	Method | Description|
| --- | --- | --- |
|String（返回vid）|	uploadVideoParts(VideoInfo videoInfo, UploadCallBack callBack, boolean printProcessLog)| callBack表示上传过程的回调，printProcessLog表示是否需要打印详细的过程信息|


#### 2.视频实体类 （VideoInfo.java）
字段信息：

|字段|	类型|	描述|	是否必填|
| --- | --- | --- | --- |
|title|	String|	视频标题	|true|
|fileSize|	long|	视频大小	|false|
|describ|	String|	视频描述|	false|
|tag|	String|	视频标签|	false|
|cataId|	Long|	分类id|	false（默认为1，表示根目录）|
|luping|	int|	是否录屏优化|	false（默认为0，不使用录屏优化）|
|keepsource|	int|	是否保持源文件播放|	false（默认为0，不使用源文件播放）|
|videoPoolId|	String|	视频id|	false（续传的时候需要传）|
|fileLocation|	String|	视频文件路径|	true|

#### 3.回调函数接口 （UploadCallBack.java）
|Modifier and Type|	Method | Description|
| --- | --- | --- |
|void|	start(String vid)|开始上传的回调事件|
|void|	process(String vid, long hasUploaded, long totalFileSize)| 每一片上传完的回调事件|
|void|	complete(String vid) | 所有片上传完的回调事件|
|void|	success(String vid) | 视频成功上传的回调事件|
|void|	error(String vid, UploadErrorMsg errorMsg) | 上传失败的回调事件 |  

#### 4.回调错误信息枚举类 （UploadErrorMsg.java）
| code | message | describe|
| --- | --- | --- |
| 0 | initial upload task failed. | 上传任务初始化接口调用失败 |
| 1 | upload video partition failed. | 上传视频分片失败 |
| 2 | upload token expired with 3 times retry. | 重试3次获取到的上传token都是无效的|
| 3 | upload video exception happen. | 上传过程抛了异常|


## 示例代码
源代码中包含了demo的类：net.polyv.sample.UploadVideoSample
需要修改里面用户配置信息（userid，secretkey）以及checkpoint文件路径，视频的具体路径才能正常使用。


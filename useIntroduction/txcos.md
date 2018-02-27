# 文件上传以及腾讯云对象存储用法示例
## 引入腾讯云存储相关jar包
```
<dependency>
    <groupId>com.qcloud</groupId>
    <artifactId>cos_api</artifactId>
    <version>4.4</version>
    <!-- 解决：Detected both log4j-over-slf4j.jar AND slf4j-log4j12.jar on the class path, preempting StackOverflowError.  -->
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
## 将文件上传到本地服务器并转存到腾讯云服务器中
```
@RequestMapping(value = "uploadFile", produces = {"application/json; charset=utf-8"})
public String uploadFile(@RequestParam("file") MultipartFile file, HttpServletRequest request){
    String result = "";
    try {
        String fileName = file.getOriginalFilename();
        String filePath = ResourceUtils.getURL("classpath:").getPath() + "upload/";
        String localFile = filePath + fileName;
        FileUtil.uploadFile(file.getBytes(), filePath, fileName);
        String targetFile = txCosConfig.getTargetDir() + fileName;
        TxCosResponse txCosResponse = txCosUtil.upload(localFile,targetFile);
        if(txCosResponse.getCode() == 0){
            result = "文件上传成功:" + txCosResponse.toString();
            //保存source_url到数据库中
        }else{
            result = "文件上传失败:localFile=" + localFile + ",targetFile=" + targetFile + "," + txCosResponse.toString();
        }
    } catch (Exception e) {
        e.printStackTrace();
        result = "服务异常";
    }
    return result;
}
```
## 文件成功上传到腾讯云服务器后将source_url路径保存到数据库中。



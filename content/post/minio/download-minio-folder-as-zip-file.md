---
title: "将MinIO中的文件夹下载为ZIP文件"
date: 2022-08-20T20:35:44+08:00
lastmod: 2022-08-20T20:35:44+08:00
draft: false
keywords: ["MinIO","ZIP","文件夹"]
description: "基于Java代码简要记录将MinIO中的文件夹下载为ZIP文件的代码实现"
tags: ["MinIO","Java"]
categories: ["Java编程"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

公司项目采用[**MinIO**](https://min.io/)作为数据存储，最近遇到一个需求是将其中的某个文件夹的数据基于其原有的结构下载为`ZIP`文件，网上有很多下载为`ZIP`的代码，个人觉得其中好多不够精简，简单记录下自己的实现。

<!--more-->

## 文件夹下载

该文件夹在`MinIO`中存储的路径为`2022-07/80/iodp-dataset-demo`，其下包含多个文件夹

![MinIO文件夹存储](/blog_img/minio/download-minio-folder-as-zip-file/minio-folder-structure.png "MinIO文件夹存储") 

在特定文件夹下包含多个数据文件

![MinIO特定文件](/blog_img/minio/download-minio-folder-as-zip-file/minio-spefic-file-path.png "MinIO特定文件") 

由于在`MinIO`中是按照文件夹存储，一开始自己很显然的是想用直接去`MinIO`官网查看有没有直接下载文件夹的方式，在其官网[**java-client-api-reference**](https://docs.min.io/docs/java-client-api-reference.html)中没有找到支持文件夹下载的方式。

接下来自己通过[**ListObjectsArgs**](https://minio-java.min.io/io/minio/ListObjectsArgs.html)传入文件夹名称`2022-07/80/iodp-dataset-demo`的方式想一次性把文件都获取到，在此种方式下程序执行结果为空。



至此，可知道 **`MinIO`不支持原生的文件夹下载，只能按照文件挨个下载**，为了获取所有文件，需要通过递归程序实现，相关代码如下

```java
@Log4j2
@Service
public class MinioServiceImpl implements IMinioService {

    @Autowired
    private MinioClient mClient;

    @Override
    public InputStream downloadFile(String fileName, String bucket) {
        GetObjectArgs getArgs = GetObjectArgs.builder().bucket(bucket).object(fileName).build();
        try {
            InputStream is = mClient.getObject(getArgs);
            return is;
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException
                 | InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException
                 | XmlParserException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public List<String> listFolderFiles(String folderName, String bucket) {
        List<String> fileList = new ArrayList<>();
        listFolderFiles(folderName, bucket, fileList);
        return fileList;
    }

    private void listFolderFiles(String folderName, String bucket, List<String> fileList) {
        ListObjectsArgs listArgs = ListObjectsArgs.builder().bucket(bucket).prefix(folderName).build();
        Iterable<Result<Item>> results = mClient.listObjects(listArgs);
        try {
            for (Result<Item> result : results) {
                Item item = result.get();
                String objectName = item.objectName();
                if (item.isDir()) {
                    // 采用递归的方式遍历文件夹
                    listFolderFiles(objectName, bucket, fileList);
                } else {
                    fileList.add(objectName);
                }
            }
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException
                 | InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException
                 | XmlParserException e) {
            throw new RuntimeException(e);
        }
    }
}
```

基于上述代码从`MinIO`中返回的文件信息类似如下所示：

![MinIO下载文件夹](/blog_img/minio/download-minio-folder-as-zip-file/minio-file-list.png "MinIO下载文件夹") 

## 写入ZIP文件

由于`MinIO`中返回的文件信息已经具有层级结构，故在进行`ZIP`压缩时可基于文件路径直接设置其存储位置[^1]

```java
@Override
public void downloadDataset(int id, HttpServletResponse response) {
    DatasetModel dsModel = queryDataset(id);
    String path = dsModel.getPath();
    int prefixLen = StringUtils.substringBeforeLast(path, "/").length();
    // 设置文件名
    String zipFileName = StringUtils.substringAfterLast(path, "/") + ".zip";

    response.setCharacterEncoding("UTF-8");
    response.setContentType("application/octet-stream");
    try {
        response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(zipFileName, "UTF-8"));
    } catch (UnsupportedEncodingException e) {
        throw new RuntimeException(e);
    }

    List<String> fileList = minioService.listFolderFiles(path, collectFilesBucket);

    try (OutputStream out = response.getOutputStream();
         ZipOutputStream zs = new ZipOutputStream(new BufferedOutputStream(out))) {
        for (String filePath : fileList) {
            String fileName = filePath.substring(prefixLen + 1);
            InputStream is = minioService.downloadFile(filePath, collectFilesBucket);
            ZipEntry entry = new ZipEntry(fileName);
            zs.putNextEntry(entry);
            IOUtils.copy(is, zs);
            zs.closeEntry();
        }
        zs.setMethod(ZipOutputStream.DEFLATED); //设置压缩方法
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

下载后的`ZIP`文件如下所示，可见其层级结构与`MinIO`中原始存储的层级结构相同，功能正常实现。

![下载后的zip文件](/blog_img/minio/download-minio-folder-as-zip-file/minio-zip-file-structure.png "下载后的zip文件") 

[^1]: 通过`new ZipEntry(fileName)`在`fileName`中指定其完整路径即可实现
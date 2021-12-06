# MinIO--文件上传

```
整合springboot
```

maven依赖

```maven
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>3.0.10</version>
</dependency>
```

application.yml

```yml
spring:
  servlet:
    multipart:
      enabled: true #开启文件上传
      max-file-size: 10MB #限制文件上传大小为10M
      
 minio:
  endpoint: http://baidu.com #你的minio服务器所在的地址
  bucketName: buck #存储桶名称
  accessKey: access #访问的key
  secretKey: secret #访问的秘钥
```

MinController

```java
 private static final Logger log = LoggerFactory.getLogger(MinioController.class);
    @Value("${minio.endpoint}")
    private String ENDPOINT;
    @Value("${minio.bucketName}")
    private String BUCKET_NAME;
    @Value("${minio.accessKey}")
    private String ACCESS_KEY;
    @Value("${minio.secretKey}")
    private String SECRET_KEY;

   
    @PostMapping("/upload")
    public Result upload(@RequestParam("file") MultipartFile file) {
        try {
            //创建一个MinIO的Java客户端
            MinioClient minioClient = new MinioClient(ENDPOINT, ACCESS_KEY, SECRET_KEY);
            boolean isExist = minioClient.bucketExists(BUCKET_NAME);
            if (isExist) {
                log.info("存储桶已经存在！");
            } else {
                //创建存储桶并设置只读权限
                minioClient.makeBucket(BUCKET_NAME);
                minioClient.setBucketPolicy(BUCKET_NAME, "*.*", PolicyType.READ_ONLY);
            }
            String filename = file.getOriginalFilename();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
            // 设置存储对象名称
            String objectName = sdf.format(new Date()) + "/" + filename;
            // 使用putObject上传一个文件到存储桶中
            minioClient.putObject(BUCKET_NAME, objectName, file.getInputStream(), file.getContentType());
            log.info("文件上传成功!");
            MinioUploadDto minioUploadDto = new MinioUploadDto();
            minioUploadDto.setName(filename);
            minioUploadDto.setUrl(ENDPOINT + "/" + BUCKET_NAME + "/" + objectName);
            return Result.success(minioUploadDto);
        } catch (Exception e) {
            log.info("上传发生错误: {}！", e.getMessage());
        }
        return Result.failed();
    }

   
    @PostMapping("/delete")
    public Result delete(@RequestParam("objectName") String objectName) {
        try {
            MinioClient minioClient = new MinioClient(ENDPOINT, ACCESS_KEY, SECRET_KEY);
            minioClient.removeObject(BUCKET_NAME, objectName);
            return Result.success(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return Result.failed();
    }
```

```
最后在postman中测试
在body选中form-data,key中参数为file，value选择你要的文件
===key                          ===value
file                             aaa.png
```


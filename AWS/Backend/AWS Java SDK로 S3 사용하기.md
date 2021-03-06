<h1>AWS S3 Java에서 사용하기</h1>

* AWS S3를 Java에서 사용하기 위해 AWS에서는 AWS-JAVA-SDK를 제공한다.   
  이 중 최근(2020년)에 나온 AWS-JAVA-SDK v2가 있는데,   
  이 라이브러리 중 `S3`를 사용하는 법을 정리한 것이다.

<h2>AWS S3에 Object 업로드 하기</h2>

* 기존 AWS-SDK-JAVA(v1)를 사용하여 구축한 방법은 아래와 같다.

* 우선, gradle dependency는 아래와 같다.
```gradle

// Other dependencies, configurations..

dependency {
    compile('com.amazonaws:aws-java-sdk-s3:1.11.901')
}
```

* 위 dependency는 maven repository에서 Java상에서 AWS S3를 이용하기 위한 모듈이 담겨져 있다.

* 위 모듈이 제공하는 기능을 활용하여 S3에 파일을 업로드하는 코드는 아래와 같다.
```java
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.model.CannedAccessControlList;
import com.amazonaws.services.s3.model.ObjectMetadata;
import com.amazonaws.services.s3.model.PutObjectRequest;
import com.amazonaws.util.IOUtils;
import com.banchango.auth.token.JwtTokenUtil;

import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.annotation.PostConstruct;
import java.io.ByteArrayInputStream;
import java.io.IOException;

@RequiredArgsConstructor
@Service
public class S3UploaderService {

    private AmazonS3 s3Client;

    @Value("${aws.s3.bucket}")
    private String bucket;

    @Value("${aws.access_key_id}")
    private String accessKey;

    @Value("${aws.secret_access_key}")
    private String secretKey;

    @Value("${aws.s3.region}")
    private String region;

    @PostConstruct
    public void setS3Client() {
        AWSCredentials credentials = new BasicAWSCredentials(this.accessKey, this.secretKey);

        s3Client = AmazonS3ClientBuilder.standard()
                .withCredentials(new AWSStaticCredentialsProvider(credentials))
                .withRegion(this.region)
                .build();
    }

    // S3에 파일을 업로드 한 후 업로드된 URL 반환
    private String uploadFile(MultipartFile file) {
        try {
            ObjectMetadata objectMetadata = new ObjectMetadata();
            byte[] bytes = IOUtils.toByteArray(file.getInputStream());
            objectMetadata.setContentLength(bytes.length);
            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);

            String fileName = file.getOriginalFilename();
            PutObjectRequest putObjectRequest = new PutObjectRequest(bucket, fileName, byteArrayInputStream, objectMetadata);
            s3Client.putObject(putObjectRequest.withCannedAcl(CannedAccessControlList.PublicRead));
            return s3Client.getUrl(bucket, fileName).toString();
        } catch(IOException exception) {
            throw new InternalServerErrorException(exception.getMessage());
        }
    }
}
```

* 위 코드는 사실상 문제가 없었지만, 이후 AWS의 다른 서비스를 사용하기 위해 SDK를 찾던 중,   
  SDK v2가 나온 것을 알게 되었다. 그리고 많은 커뮤니티에서 AWS SDK v1을 사용하지 말고,   
  V2를 사용하라는 말들을 보았다.

* 아직 v2 에 대한 코드들이 인터넷에 많이 없고, 나도 API document와 라이브러리를 파고들면서   
  작성한 코드이기에 적는중이다.

* 이제 위 코드를 AWS SDK JAVA v2를 이용하여 작성해보자.
  
* 먼저 gradle dependency 설정은 아래와 같다.

```gradle

// Other gradle dependencies and configurations..

dependencies {
    implementation platform('software.amazon.awssdk:bom:2.15.0')
    implementation('software.amazon.awssdk:ses')
    implementation('software.amazon.awssdk:s3')
}
```

  * AWS SDK v1을 사용할 때의 Spring Boot는 `2.1.7.RELEASE` 버전이었지만, SDK v2 document를 보던 도중,   
    Gradle 버전이 5.0 이상일 때 AWS SDK들의 버전 및 의존 설정을 알아서 해준다고 하길래   
    Spring Boot를 이 글 작성 시점에서의 가장 최신인 `2.4.1`로 업그레이드 하였고   
    Gradle도 마찬가지로 기존 `4.10.2-all`에서 `6.7.1`로 업그레이드 하였다.

* 이제 위에서 작성한 파일을 S3로 업로드하는 코드를 SDK v2에 맞게 다시 작성해보자.
```java
import lombok.RequiredArgsConstructor;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.multipart.MultipartFile;

import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.S3Utilities;
import software.amazon.awssdk.services.s3.model.*;

import javax.annotation.PostConstruct;
import java.io.IOException;

@RequiredArgsConstructor
@Service
public class S3UploaderService {

    private S3Client s3Client;

    @Value("${aws.s3.bucket}")
    private String bucket;

    @Value("${aws.access_key_id}")
    private String accessKey;

    @Value("${aws.secret_access_key}")
    private String secretKey;

    @PostConstruct
    public void setS3Client() {
        s3Client = S3Client.builder()
                .credentialsProvider(() -> AwsBasicCredentials.create(accessKey, secretKey))
                .region(Region.AP_NORTHEAST_2).build();
    }

    private String uploadFile(MultipartFile file) {
        try {
            PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                    .bucket(bucket)
                    .contentLength(file.getSize())
                    .key(file.getOriginalFilename())
                    .acl(ObjectCannedACL.PUBLIC_READ)
                    .build();

            s3Client.putObject(putObjectRequest, RequestBody.fromBytes(file.getBytes()));

            S3Utilities s3Utilities = s3Client.utilities();
            GetUrlRequest getUrlRequest = GetUrlRequest.builder()
                    .bucket(bucket)
                    .key(file.getOriginalFilename())
                    .build();
            return s3Utilities.getUrl(getUrlRequest).toString();
        } catch(IOException exception) {
            throw new InternalServerErrorException(exception.getMessage());
        }
    }
}
```
<hr />

<h2>AWS S3에 있는 Object 삭제하기</h2>

* 마찬가지로 AWS SDK v1의 코드를 V2로 바꿔보자.

* 먼저, v1으로 작성된 코드는 아래와 같다.
```java
@RequiredArgsConstructor
@Service
public class S3UploaderService {

    private AmazonS3 s3Client;
    
    // @PostConstruct

    private void deleteFile(final String fileName) {
        final DeleteObjectRequest deleteObjectRequest = new DeleteObjectRequest(bucket, fileName);
        try {
            s3Client.deleteObject(deleteObjectRequest);
        } catch(Exception exception) {
            throw new InternalServerErrorException(exception.getMessage());
        }
    }
}
```

* 위 코드를 v2로 바꿔보자.
```java
@RequiredArgsConstructor
@Service
public class S3UploaderService {

    private S3Client s3Client;

    // @PostConstruct

    private void deleteFile(final String fileName) {
        DeleteObjectRequest deleteObjectRequest = DeleteObjectRequest.builder()
                .bucket(bucket)
                .key(fileName)
                .build();
        try {
            s3Client.deleteObject(deleteObjectRequest);
        } catch(Exception exception) {
            throw new InternalServerErrorException(exception.getMessage());
        }
    }
}
```
<hr/>

* V1에서 V2로 옮길 때 Static Builder Pattern으로 바뀐 라이브러리가 많아졌다는 것이 느껴졌다.
---
layout: post
title : "Python boto3 복사, 업로드, 동기화 유용한 기능"
subtitle: "Python boto3 copy, upload, invalidation"
categories : devlog
tags : python aws
typora-root-url : ..
---

python boto3 라이브러리로 aws 관련 기능들이 필요하여 가장 중요한 기능들을 정리해 보았습니다.
많은 기능들 중에 아마  `upload`, `copy`, `invalidation`이 가장 많을텐데요. 
구글링을 하여 제 입맛에 맞도록 수정하여 정리 하였습니다.



## Copy folder

동일 버킷, 혹은 다른 버킷으로 파일 및 폴더를 복사 하는 예제입니다. 
`s3://aws_bucket/folderA/folderB/000` 의 폴더를 `folderA/folderB/1111`  로 복사를 한다면,

아래의 예제처럼 작성 할 수 있습니다.

```python
import boto3

s3 = boto3.resource('s3')
src_info = "folderA/folderB/0000"
dst_info = "folderA/folderB/1111"

s3_bucket = s3.Bucket('aws_bucket')

for obj in s3_bucket.objects.filter(Prefix=src_info):
	old_source = {'Bucket': 'aws_bucket', 'Key': obj.key}
	new_key = obj.key.replace(src_info, dst_info, 1)
	print(f"Copy {obj.key} -> {new_key}")
	new_obj = s3_bucket.Object(new_key)
	new_obj.copy(old_source)

```



## Upload folder

boto3 라이브러리에서 폴더를 업로드하는 기능은 지원하지 않아 새로 구현이 필요합니다.

하위 폴더의 재귀적인 호출은 하지 않고 1뎁스 깊이만 폴더 업로드를 지원하는 예제입니다.

```python
import boto3
import os

def upload_dir(profile_name, local_directory, bucket, destination):
    if(False == os.path.isdir(local_directory)):
        return False

    session = boto3.Session(profile_name=profile_name)
    s3_client = session.client('s3')

    for root, dirs, files in os.walk(local_directory):
        for filename in files:
            local_path = os.path.join(root, filename)
            relative_path = os.path.relpath(local_path, local_directory)

            s3_path = f"{destination}/{filename}"

            try:
                print(f"Uploading {s3_path}")
                s3_client.upload_file(local_path, bucket, s3_path, ExtraArgs={
                                      'ACL': 'public-read'})
            except ClientError as e:
                print(e)
                return False

    return True
```



## Upload File

가장 많이 사용되는 파일 업로드 기능입니다.

```python
import boto3

def upload_file(profile_name, file_name, bucket, object_name=None):

    if object_name is None:
        object_name = file_name

    session = boto3.Session(profile_name=profile_name)
    s3_client = session.client('s3')

    try:
        print(f"Uploading  {object_name}")
        response = s3_client.upload_file(
            file_name, bucket, object_name, ExtraArgs={'ACL': 'public-read'})
    except ClientError as e:
        print(e)
        return False
    return True

```

## Cloudfront Invalidation

`DistributionId` 는 미리 생성이 필요합니다.

```python
import boto3

def wait_invalidation(dist_id, files):
    client = boto3.client('cloudfront')

    response = client.create_invalidation(DistributionId=dist_id,
                                          InvalidationBatch={
                                              'Paths': {
                                                  'Quantity': len(files),
                                                  'Items': ['/{}'.format(f) for f in files]
                                              },
                                              'CallerReference': 'my-references-{}'.format(datetime.datetime.now())})

    invalidation_id = response['Invalidation']['Id']
    waiter = client.get_waiter('invalidation_completed')
    waiter.wait(DistributionId=dist_id,
                Id=invalidation_id,
                WaiterConfig={
                    'Delay': 300,
                    'MaxAttempts': 30
                })

    return
```





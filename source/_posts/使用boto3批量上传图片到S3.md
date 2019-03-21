---
title: 使用boto3批量上传图片到S3
comments: true
date: 2018-12-13 18:48:33
tags:
- 上传图片 s3
categories:
- Python
---
最近在工作中需要把本地的图片上传到亚马逊对象存储S3中供外链访问。

为了更快的实现，使用了Python 接口的boto3进行封装，实现批量上传图片到S3

<!--more-->
主要有以下的几个函数：

1、实现S3的连接
```
# coding: utf-8
import boto3
import os

AWS_ACCESS_KEY_ID = 'xxxx'
AWS_SECRET_ACCESS_KEY = 'xxx'
BUCKET_NAME = 'xxxx'
REGION_NAME = 'xxx'
PHOTO_FOLDER = 'xxxx/photo'


def init_s3_logo_photo(region_name=REGION_NAME, aws_access_key_id=AWS_ACCESS_KEY_ID,
                       aws_secret_access_key=AWS_SECRET_ACCESS_KEY):
    s3 = boto3.resource('s3', region_name=region_name, aws_access_key_id=aws_access_key_id,
                        aws_secret_access_key=aws_secret_access_key)
    return s3

```

2、实现单个图片的上传
```
def upload_logo_photo(logo_photo_s3=init_s3_logo_photo(), bucket_name=BUCKET_NAME, photo_name=None):
    with open(os.path.join(PHOTO_FOLDER, photo_name), 'rb') as f:
        photo_stream = f.read()
        try:
            # upload logo to s3
            logo_photo_s3.Bucket(bucket_name).put_object(Key=photo_name, Body=photo_stream)
            print("{} upload done".format(photo_name))
        except Exception as e:
            print("upload {} error:{}".format(photo_name, e))
```
3、实现单个图片的删除
```
def delete_logo_photo_by_name(logo_photo_s3=init_s3_logo_photo(), bucket_name=BUCKET_NAME, photo_name=None):
    try:
        # delete from s3
        logo_photo_s3.Object(bucket_name, photo_name).delete()
        print("Delete {} OK from s3!".format(photo_name))
    except Exception as e:
        print("Delete {} ERROR:{}".format(photo_name, e))
```
4、实现整个bucket的内容的清除
```
def delete_whole_logo():
    # this func will delete all content in bucket data
    s3 = init_s3_logo_photo()
    b = s3.Bucket(BUCKET_NAME)
    for key in list(b.objects.all()):
        delete_logo_photo_by_name(photo_name=key.key)
```
5、实现上传整个文件夹里的图片并做去除非文件
```
def upload_logo_photo_by_folder():
    # 1. to get document list
    os.chdir(PHOTO_FOLDER)
    logo_list = os.listdir(PHOTO_FOLDER)

    # 2. which isn't file will be removed from logo_list
    for logo in logo_list:
        if not os.path.isfile(os.path.abspath(logo)):
            logo_list.remove(logo)
        upload_logo_photo(photo_name=logo)


if __name__ == "__main__":
    upload_logo_photo_by_folder()
```

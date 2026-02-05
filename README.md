#### 更改主机名
```javascript
cat >> /etc/hosts << EOF
172.27.0.6 node1
172.27.0.7 node2
172.27.0.8 node3
EOF
```


### 部署mino1
```javascript
docker run -d --name minio \
  --restart=always --net=host \
  -e MINIO_ACCESS_KEY=minio \
  -e MINIO_SECRET_KEY=minio123 \
  -v /data/minio-data1:/data1 \
  -v /data1/minio-data2:/data2 \
  -v /etc/localtime:/etc/localtime \
  minio/minio server \
  --address 172.27.0.6:9000 \
  http://node{1...3}/data{1...2}
```

### 部署mino2
```javascript
docker run -d --name minio \
  --restart=always --net=host \
  -e MINIO_ACCESS_KEY=minio \
  -e MINIO_SECRET_KEY=minio123 \
  -v /data/minio-data1:/data1 \
  -v /data1/minio-data2:/data2 \
  -v /etc/localtime:/etc/localtime \
  minio/minio server \
  --address 172.27.0.7:9000 \
  http://node{1...3}/data{1...2}
```

### 部署mino3
```javascript
docker run -d --name minio \
  --restart=always --net=host \
  -e MINIO_ACCESS_KEY=minio \
  -e MINIO_SECRET_KEY=minio123 \
  -v /data/minio-data1:/data1 \
  -v /data1/minio-data2:/data2 \
  -v /etc/localtime:/etc/localtime \
  minio/minio server \
  --address 172.27.0.8:9000 \
  http://node{1...3}/data{1...2}
```

### 连接
```javascript
docker run -it --rm --entrypoint=/bin/sh minio/mc
mc config host add minio http://172.27.0.6:9000 minio minio123
mc admin heal -r minio  #删除文件恢复
```


### 部署客户端
```javascript
pip3 install s3cmd

cat /root/.s3cfg 
host_base = 172.27.0.6:9000
host_bucket = 172.27.0.6:9000
use_https = False
access_key =  minio
secret_key = minio123
signature_v2 = False

s3cmd ls   #列出所有对象存储.
s3cmd get s3://harbor/install.sh .  #下载
s3cmd put file s3://harbor      #上传
```

### 创建docker-registry用户并且授权
```javascript
#新建用户
./mc admin user add myminio docker-registry Test@123

#创建授权docker-registry bucket的json文件
cat > bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:CreateBucket"
      ],
      "Resource": ["arn:aws:s3:::*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation",
        "s3:ListBucketMultipartUploads"
      ],
      "Resource": ["arn:aws:s3:::docker-registry"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:AbortMultipartUpload",
        "s3:ListMultipartUploadParts"
      ],
      "Resource": ["arn:aws:s3:::docker-registry/*"]
    }
  ]
}
EOF
#创建叫docker-registry-policy的policy
mc admin policy create myminio docker-registry-policy bucket-policy.json
绑定授权docker-registry-policy policy 给用户
mc admin policy attach myminio docker-registry-policy --user docker-registry



#如果要更新权限，需要解绑再重新添加
mc admin policy detach myminio docker-registry-policy --user docker-registry
mc admin policy remove myminio docker-registry-policy

#重新添加权限
mc admin policy create myminio docker-registry-policy bucket-policy.json
mc admin policy attach myminio docker-registry-policy --user docker-registry


#使用docker-registry用户测试增删改查权限.
./mc alias set myminio http://localhost:9000 admin Test@123
```

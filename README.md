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

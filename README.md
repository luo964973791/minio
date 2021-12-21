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
  -v /data/minio-data2:/data2 \
  minio/minio server \
  --address 172.27.0.6:9000 \
  http://minio-{1...3}/data{1...2}
```

### 部署mino2
```javascript
docker run -d --name minio \
  --restart=always --net=host \
  -e MINIO_ACCESS_KEY=minio \
  -e MINIO_SECRET_KEY=minio123 \
  -v /data/minio-data1:/data1 \
  -v /data/minio-data2:/data2 \
  minio/minio server \
  --address 172.27.0.7:9000 \
  http://minio-{1...3}/data{1...2}
```

### 部署mino3
```javascript
docker run -d --name minio \
  --restart=always --net=host \
  -e MINIO_ACCESS_KEY=minio \
  -e MINIO_SECRET_KEY=minio123 \
  -v /data/minio-data1:/data1 \
  -v /data/minio-data2:/data2 \
  minio/minio server \
  --address 172.27.0.8:9000 \
  http://minio-{1...3}/data{1...2}
```

### 部署客户端
```javascript
mc config host add minio http://172.27.0.6:9000 minio minio123
```

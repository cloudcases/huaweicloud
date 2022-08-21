## [restic 快速安全可靠的数据备份工具](https://www.cnblogs.com/rongfengliang/p/12639794.html)

restic 是基于golang 编写的快速，安全，可靠的数据备份工具,使用简单，同时支持多种后端存储

## 支持的后端存储

- 本地
- sftp （通过ssh）
- http rest server （rest-server restic提供的 ）
- s3 （同时支持minio）
- openstack swift
- backblaze b2
- azure blob storage
- google cloud storage
- 以及其他可以通过rclone 访问的后端存储

## minio 简单集成使用

- 环境准备
  docker-compose, 详细说明参考https://www.cnblogs.com/rongfengliang/p/12639449.html

```
version: '3.7'
services:
  sidekick:
    image: dalongrong/sidekick:v0.1.8
    tty: true
    ports:
    - "80:80"
    command: --health-path=/minio/health/ready --address :80 http://minio{1...4}:9000
  gateway:
    image: minio/minio:RELEASE.2020-04-04T05-39-31Z
    command: gateway s3 http://sidekick
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
    ports:
    - "9000:9000"
  minio1:
    image: minio/minio:RELEASE.2020-04-04T05-39-31Z
    volumes:
      - data1-1:/data1
      - data1-2:/data2
    ports:
      - "9001:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_BROWSER: "off"
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
  minio2:
    image: minio/minio:RELEASE.2020-04-04T05-39-31Z
    volumes:
      - data2-1:/data1
      - data2-2:/data2
    ports:
      - "9002:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_BROWSER: "off"
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
  minio3:
    image: minio/minio:RELEASE.2020-04-04T05-39-31Z
    volumes:
      - data3-1:/data1
      - data3-2:/data2
    ports:
      - "9003:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_BROWSER: "off"
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
  minio4:
    image: minio/minio:RELEASE.2020-04-04T05-39-31Z
    volumes:
      - data4-1:/data1
      - data4-2:/data2
    ports:
      - "9004:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_BROWSER: "off"
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
volumes:
  data1-1:
  data1-2:
  data2-1:
  data2-2:
  data3-1:
  data3-2:
  data4-1:
  data4-2:
 
```

- 安装restic
  参考https://github.com/restic/restic/releases
- 配置s3环境变量

 

```
export AWS_ACCESS_KEY_ID=minio
export AWS_SECRET_ACCESS_KEY=minio123
```

- 初始化restic

```
restic -r s3:http://localhost/backup init
```

效果
![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230036517-1902456900.png)

 

 


![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230046044-1031489580.png)

 

 

- 进行数据备份操作

```
restic -r s3:http://localhost/backup backup ./
```

效果

![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230103044-1198671513.png)

 

 

![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230114689-1749844830.png)

 

 

- 查看备份

 

```
restic -r s3:http://localhost/backup snapshots
```

效果
![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230131583-259764964.png)

 

 

- 查看备份内容

 

```
restic -r s3:http://localhost/backup ls 875a2a32
```

效果
![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230147672-1087584043.png)

 

 

- 数据恢复

 

```
restic -r s3:http://localhost/backup restore 875a2a32 -t ./
```

效果
![img](https://img2020.cnblogs.com/blog/562987/202004/562987-20200405230206138-1849088000.png)

 

 

## 说明

restic 是一个很不错的数据备份方案，rclone是一个不错的数据同步方案，以及minio作为数据存储，集成在一起真的很不错

## 参考资料

https://restic.readthedocs.io/en/stable/
https://github.com/restic/restic
https://github.com/rongfengliang/minio-cluster-sidekick-learning

分类: [minio](https://www.cnblogs.com/rongfengliang/category/1094244.html), [s3](https://www.cnblogs.com/rongfengliang/category/1094245.html), [云运维&&云架构](https://www.cnblogs.com/rongfengliang/category/1107309.html), [灾备处理](https://www.cnblogs.com/rongfengliang/category/1468563.html), [restic](https://www.cnblogs.com/rongfengliang/category/1690424.html), [rclone](https://www.cnblogs.com/rongfengliang/category/1690447.html)
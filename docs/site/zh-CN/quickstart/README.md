# Phantoscope 快速开始
## 开始之前
确认 Phantoscope 所有组件正常：
```bash 
$ docker-compose ps
```

*看到如下输出即表示 Phantoscope 正在运行:*
```
Name                      Command               State                        Ports
----------------------------------------------------------------------------------------------------------------
phantoscope_api_1      /usr/bin/gunicorn3 -w 4 -b ...   Up      0.0.0.0:5000->5000/tcp
phantoscope_milvus_1   /var/lib/milvus/docker-ent ...   Up      0.0.0.0:19530->19530/tcp, 0.0.0.0:8080->8080/tcp
phantoscope_minio_1    /usr/bin/docker-entrypoint ...   Up      0.0.0.0:9000->9000/tcp
phantoscope_mysql_1    docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp
phantoscope_vgg_1      python3 server.py                Up      0.0.0.0:50001->50001/tcp
```


## 准备环境

运行 scripts 文件夹下的 prepare.sh 脚本。该脚本注册了一个 Operator ，并以该 Operator 创建一个 Pipeline，并根据该 Pipeline 创建了一个名为 example_app 的 Application。
```bash
$ chmod +x scripts/prepare.sh
$ ./scripts/prepare.sh
```

## 下载图片数据
本文使用 [Caltech 256 数据集](https://www.kaggle.com/jessicali9530/caltech256)，是加利福尼亚理工学院收集整理的数据集，该数据集选自 Google Image 数据集，并手工去除了不符合其类别的图片。在该数据集中，图片被分为 256 类，每个类别的图片超过 80 张。
```bash
$ curl http://www.vision.caltech.edu/Image_Datasets/Caltech256/256_ObjectCategories.tar -o /tmp/vgg-example.tar
```

## 导入图片数据
```bash
$ tar xvf /tmp/vgg-example.tar -C /tmp
$ pip3 install requests tqdm
$ python3 scripts/load_data.py -s $LOCAL_ADDRESS:5000 -a example_app -p example_pipeline -d /tmp/256_ObjectCategories
```
上传图片根据机器性能不同，时间会有差异。

## 使用 Preview 进行搜索
```bash
docker run -d -e API_URL=http://$LOCAL_ADDRESS:5000 -p 8000:80 phantoscope/preview:latest
```
![Phantoscope Preview 演示图](../../../../.github/preview.gif)


## 使用 curl 导入一张图片
如果你想单独导入一张图片，可以使用如下命令
``` bash
$ curl --location --request POST $LOCAL_ADDRESS':5000/v1/application/example_app/upload' \
--header 'Content-Type: application/json' \
--data "{
    \"fields\": {
        \"example_field\": {
            \"url\": \"https://tse2-mm.cn.bing.net/th/id/OIP.C3pWPyFPhBMiBeWoncc24QHaCq?w=300&h=108&c=7&o=5&dpr=2&pid=1.7\"
        }
    }
}"

```

*预期得到如下响应：*
```json
[
    {
        "_id": 1591585583689787000,
        "_app_name": "example_app",
        "_image_url": "http://host:9000/example/example-19ef9e9ba7f745dd90b2d9373c1aed56",
        "_fields": {
            "example_field": {
                "type": "object",
                "pipeline": "example_pipeline"
            }
        }
    }
]
```

## 使用 curl 进行搜索
```bash
$ curl --location --request POST $LOCAL_ADDRESS':5000/v1/application/example_app/search' \
--header 'Content-Type: application/json' \
--data "{
    \"fields\": {
        \"example_field\": {
            \"url\": \"https://tse2-mm.cn.bing.net/th/id/OIP.C3pWPyFPhBMiBeWoncc24QHaCq?w=300&h=108&c=7&o=5&dpr=2&pid=1.7\"
        }
    },
    \"topk\": 2
}"
```

*预期得到如下响应：*
```json
[
    {
        "_id": "1591584893762549000",
        "_app_name": "example_app",
        "_image_url": "http://host:9000/example/example-b26e52aa65df4c23bbd848e98df1f0a3",
        "_fields": {
            "example_field": {
                "type": "object",
                "pipeline": "example_pipeline"
            }
        }
    },
    {
        "_id": "1591584895837488000",
        "_app_name": "example_app",
        "_image_url": "http://host:9000/example/example-e53e7a233c814b7f825f7b58c2647501",
        "_fields": {
            "example_field": {
                "type": "object",
                "pipeline": "example_pipeline"
            }
        }
    }
]
```

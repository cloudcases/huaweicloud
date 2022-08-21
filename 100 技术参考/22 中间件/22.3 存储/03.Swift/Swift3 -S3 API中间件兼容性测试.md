# Swift3 ——S3 API中间件兼容性测试

在Swift中，Swift3中间件在Object Storage上提供了S3 REST 风格的API。

目前支持的操作有以下几种：

· GET Service

· DELETE Bucket

· GET Bucket (List Objects)

· PUT Bucket

· DELETE Object

· GET Object

· HEAD Object

· PUT Object

· PUT Object (Copy)



**配置注意：**

在proxy-server.conf配置swift3，并确保swift3在auth和其他查找swift请求的中间件之前。具体配置示例如下：

```
[pipeline:main]
pipeline = healthcheck cache swift3 swauth proxy-server

[filter:swift3]
use = egg:swift#swift3
```

以SAIO的配置为例，使用AWS的boto python包来分析S3 API的使用，基本信息：

```
account: test
user:tester
password: testing
```

### **0.** **连接**

文档中仅给出了一个连接例子，代码如下：

```python
connection = boto.s3.Connection(
  aws_access_key_id=*'test:tester'*,
  aws_secret_access_key=*'testing'*,
  port=8080,
  host=*'127.0.0.1'*,
  is_secure=False,
  calling_format=boto.s3.connection.OrdinaryCallingFormat())
```

但是我在测试时，发现存在问题，traceback如下：

```
Traceback (most recent call last):
 File "/home/swift/tmp2.py", line 8, in <module>
  connection = boto.s3.Connection(
AttributeError: 'module' object has no attribute 'Connection'
```

我估计是写错了，boto.s3.connection是一个module，我去阅读了boto的[文档](http://readthedocs.org/docs/boto/en/latest/s3_tut.html)，正确写法应该是：

```python
connection = boto.s3.connection.S3Connection(
  aws_access_key_id=*'test:tester'*,
  aws_secret_access_key=*'testing'*,
  port=8080,
  host=*'127.0.0.1'*,
  is_secure=False,
  calling_format=boto.s3.connection.OrdinaryCallingFormat())
```

创建成功，无任何输出消息。

其中access_key和aws_access_key_id相同。

```
>>> connection.access_key
'test:tester'
>>> connection.aws_access_key_id
'test:tester'
>>> connection.aws_secret_access_key
'testing'
```



### **1.** **Container管理**

Swift中Container的概念与S3中的Bucket相等。

使用以下语句创建一个bucket：

```python
>>> connection.create_bucket('1st')
```

```
<Bucket: 1st>
```

如果再次创建相同名称的bucket，会出现409冲突错误：

```
boto.exception.S3CreateError: S3CreateError: 409 Conflict
```

如果使用了大写字母也会出错，而使用rackspace的API则无此限制：

```python
>>> connection.create_bucket('1sT')
```

```
**boto.exception.BotoClientError: BotoClientError**: Bucket names cannot contain upper-case characters when using either the sub-domain or virtual hosting calling format.
```

使用中文字符创建bucket，出现403 Forbidden错误：

```python
>>> connection.create_bucket(*'结果'*)
```

```
boto.exception.S3ResponseError: S3ResponseError: 403 Forbidden
```

此外，可以单独使用纯数字或下划线_创建bucket，不可以使用!@#$%&等符号。

也可以使用connecttion.create_bucket('media.userdomain.com')的风格来创建bucket。

#### 检索bucket

使用get_all_buckets()来获得所有的buckets信息：

```python
>>> connection.get_all_buckets()
```

```
[<Bucket: 1>, <Bucket: 1st>, <Bucket: SINA>, <Bucket: ___>, <Bucket: a>, <Bucket: aa>, <Bucket: media.user.com>,<Bucket: photos>, <Bucket: sss>, <Bucket: 图片>, <Bucket: 文件>, <Bucket: 视频>]
```

此外，使用get_bucket()来获得指定bucket，这里分别测试了小写字母、大写字母、不存在的、中文的bucket，其中大写字母的bucket可以获得，get不存在的bucket返回400 Bad Request，而get中文字符的bucket仍然返回403 Forbidden：

```python
>>> connection.get_bucket('sss')
```

```
<Bucket: sss>

<Bucket: SINA>

Traceback (most recent call last):
boto.exception.S3ResponseError: S3ResponseError: 400 Bad Request
<?xml version=*"1.0"* encoding=*"UTF-8"*?>
<Error>
 <Code>InvalidBucketName</Code>
 <Message>The specified bucket is not valid</Message>
</Error>

Traceback (most recent call last):
boto.exception.S3ResponseError: S3ResponseError: 403 Forbidden
<?xml version=*"1.0"* encoding=*"UTF-8"*?>
<Error>
 <Code>AccessDenied</Code>
 <Message>Access denied</Message>
</Error>
```



#### 删除bucket

使用delete_bucket()来删除指定的bucket，如果目标bucket不存在，返回400 Bad Request，中文的bucket仍然有问题。

```python
>>> connection.delete_bucket('sss')
>>> 
>>> connection.delete_bucket('sss')
```

```
boto.exception.S3ResponseError: S3ResponseError: 400 Bad Request
<?xml version=*"1.0"* encoding=*"UTF-8"*?>
<Error>
 <Code>InvalidBucketName</Code>
 <Message>The specified bucket is not valid</Message>
</Error>
```

```python
>>> connection.delete_bucket('视频')
```

```
boto.exception.S3ResponseError: S3ResponseError: 403 Forbidden
<?xml version=*"1.0"* encoding=*"UTF-8"*?>
<Error>
 <Code>AccessDenied</Code>
 <Message>Access denied</Message>
</Error>
```



### **2.** **key管理**

####  获得key列表

在swift中使用object的概念与S3中的key对应。

```python
>>> con=connection.get_bucket('photos') #获得名为photos的bucket

>>> con.get_all_keys()  #获得key列表
```

```
[<Key: photos,List>, <Key: photos,lzl.jpg>, <Key: photos,sample>]
```



#### **创建key**

使用以下流程创建了一个key，从本地磁盘上传了一张照片并设置了元数据。

```python
>>> from boto.s3.key import Key 
>>> k=Key(con,'sample')  *#在photos内新建一个名为sample的key*
>>> k
```

```
<Key: photos,sample> 
'sample'
46486L
{'data': '2011'}
```



#### **提取key**

使用

```python
 k.get_contents_to_filename('/home/swift/td.jpg')
```

下载到本地磁盘上。

#### **从字符串获得content**

```python
>>> new_text=Key(con,'List')
>>> new_text.set_contents_from_string('This is a list of samples')
>>> new_text.get_contents_as_string()
```

```
'This is a list of samples'
```




#### **key的重写操作**

由于swift不是文件系统，不支持类似于POSIX文件系统的读写操作，上传一个文件的唯一方式本质上就是重写这个文件。比如，为之前的new_text追加一行字符串，结果发现这个文件的内容被重写了。通过etag可以判断文件的版本信息。

```python
>>> new_text.etag
```

```
*'33e6bddcde5e0615dcd301f8f2e1e683'*
```

```python
>>> new_text.set_contents_from_string(*'New photo added'*)
>>> new_text.get_contents_as_string()
```

```
*'New photo added'*
```

```python
>>> new_text.etag
```

```
*'"6e0c13f0143a54cfa250dbace6da3a5d"'*
```



#### **使用copy复制key**

把pohotos中的sample复制到SINA中并重命名为new_sample

```python
>>> k=connection.get_bucket('photos').get_key('sample')
>>> k
```

```
<Key: photos,sample>
```

```python
>>> new_k=k.copy('SINA','new_sample')
>>> new_k.bucket
```

```python
>>> new_k
<Key: SINA,new_sample>
```



#### **key删除**

使用key的delete()方法就可以删除key本身

```python
>>> k.delete()
>>> k.exists()
```

```
False
```





**参考文档**

http://aws.amazon.com/articles/3998?_encoding=UTF8&jiveRedirect=1

http://readthedocs.org/docs/boto/en/latest/ref/s3.html

None

分类: [Swift云存储](https://www.cnblogs.com/yuxc/category/324510.html)

标签: [Swift](https://www.cnblogs.com/yuxc/tag/Swift/), [openstack](https://www.cnblogs.com/yuxc/tag/openstack/), [对象存储](https://www.cnblogs.com/yuxc/tag/对象存储/), [S3 API](https://www.cnblogs.com/yuxc/tag/S3 API/), [兼容性](https://www.cnblogs.com/yuxc/tag/兼容性/)



https://www.cnblogs.com/yuxc/archive/2012/05/12/2497385.html
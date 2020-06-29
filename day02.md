### 回顾：

```
1、git的三个区域划分
工作区、缓冲区、仓库
2、添加一个新文件到仓库的过程
git add 把新文件添加到缓冲区
git commit 把缓冲区中的所有更新提交到仓库
3、修改文件到仓库的过程
git add
git commit
这个可以简写，git commit -a -m "提交消息" 
4、查看分支
git branch 
5、添加一个分支
git branch 分支名
6、合并分支
git merge 分支A   
```

补充：

在合并分支的时候，如果分支A与分支B都对同一个文件的同一个部分进行了不同修改，会造成冲突，

我们解决这个冲突：

选择A的提交

选择B的提交

也可以手动整合这两个提交

冲突：

```
$ git merge dev
Auto-merging Day628/stuapp/views.py
CONFLICT (content): Merge conflict in Day628/stuapp/views.py #有冲突
Automatic merge failed; fix conflicts and then commit the result.

```

冲突文件展示冲突内容：

```
def index(request):
<<<<<<< HEAD
    return HttpResponse("OK，我在master分支上修改我的东西")
=======
    return HttpResponse("OK，我在我的dev分支上添加新内容")
>>>>>>> dev

```

我们可以选择 是使用当前分支的提交(了解)

```
git checkout --ours   冲突文件
```

还可以选择使用要合并的分支的提交

```
git checkout --theirs 冲突文件
```



可以手动修改这个冲突，然后修改以后执行

```
git add 添加修改
git commit 提交修改
```



git的版本回滚-自己去查看



可扩展字段

1、先预留字段，到时再用。

2、JSON 格式保存。

3、设计单独的表存储





### 用户字段提取

```
用户字段：
ID 用户名 密码 手机号 邮箱 状态
```



### JWT

身份凭证 ： 唯一性、安全性、时效性

jwt可以满足我们的身份凭证特点，目前也是前后端分离开发 非常流畅的身份认证机制。

JWT 英文名是 Json Web Token。 常用在跨域身份验证，SSO单点登录上，也可以携带一些进行数据传递。

#### 组成

- **Header 头部**

头部包含了两部分，token 类型和采用的加密算法

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `typ`: （Type）类型。在JOSE Header中这是个可选参数，但这里我们需要指明类型是`JWT`。
- `alg`: （Algorithm）算法，必须是JWS支持的算法

它会使用 base64url编码组成 JWT 结构的第一部分



- **Payload 载荷/负载**

这部分就是我们存放信息的地方了，你可以把用户 ID 等信息放在这里，JWT 规范里面对这部分有进行了比较详细的介绍，JWT 规定了7个官方字段，供选用

```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

同样的，它会使用 base64url 编码组成 JWT 结构的第二部分

- **Signature 签名**

 签名的作用是保证 JWT 没有被篡改过

前面两部分都是使用 base64url 进行编码的，即前端可以解开知道里面的信息。Signature 需要使用编码后的 header 和 payload 以及我们提供的一个密钥，这个密钥只有服务器才知道，不能泄露给用户，然后使用 header 中指定的签名算法（HS256）进行签名。

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。



#### 2.2.1 DRF框架生成JWT  ★★★★

这个是我们在P5的时候使用的jwt生成方式，jwt与djangorestframework相结合，有一个django-rest-framework-jwt模块 帮助我们进行jwt的生成、验证操作。

这里有一个注意事项：要使用drf提供的jwt 则用户model必须继承django提供的AbstractUser类，使用django一套身份认证规则。

```
from django.contrib.auth.models import AbstractUser

class UserModel(AbstractUser):
    STATUS_CHOICES = ((1, "未激活"), (2, "激活"), (3, "拉黑"))
    phone = models.CharField(max_length=11)
    status = models.SmallIntegerField(choices=STATUS_CHOICES)

    class Meta:
        db_table = "users"

需要在settings中配置UserModel
AUTH_USER_MODEL = "userapp.UserModel"
```



#### 2.2.2 pyjwt生成JWT  ★★★★

这一个是是有独立的jwt模块来自己生产jwt令牌、验证jwt令牌，不需要凭借django的认证规则，使用更轻便。

去下载pyjwt

```
pip install pyjwt
```

```
dic = {
    'exp': datetime.datetime.now() + datetime.timedelta(days=1),  # 过期时间
    'iat': datetime.datetime.now(),  #  开始时间
    'iss': 'lianzong',  # 签发人
    'data': {  # 内容，一般存放该用户信息
        'uid': 1,
        'username': ,
    },
}

s = jwt.encode(dic, 'secret', algorithm='HS256')  # 加密生成字符串
print(s)
s = jwt.decode(s, 'secret', issuer='lianzong', algorithms=['HS256'])  # 解密，校验签名
print(s)
print(type(s))
```



```
import jwt
import datetime

dic = {
    "exp": datetime.datetime.now() + datetime.timedelta(hours=1),
    "iat": datetime.datetime.now(),
    "data": {
        "uid": 10,
        "username": "张三"
    }
}

# 生成一个jwt令牌token
s = jwt.encode(dic, "abcdef", algorithm="HS256")
print(s)
# s = s + b"123"
# 我们解析令牌token
r = jwt.decode(s, "abcdef", algorithms=["HS256"])

print(r)

```



#### 2.2.3 JWT的生命周期  ★★★



#### 2.2.4 JWT的容器(Web Storage和Cookie)  ★★★



### 作业

```
开发登录接口、生成jwt令牌，中间件校验令牌
```


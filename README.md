# pod基本操作流程

### 1. 编写文件，创建代码库

### 2. 编写代码并上传

如果上传不成功，提示某些信息已修改，需要先pull下代码，看下是否有冲突再决定上传
git pull

### 3. 编写podspec文件，关联到代码仓库

#### ---------编写podspec文件
* git上创建库
* 克隆到本地
``` obj
git clone http:/// 地址.git
```
* 每一个pods代码库都必须有一个和库名保持一致的`.podsspec`的描述文件

``` obj
pod spec create 库名
```
* 编写描述文件【下示例】

``` obj
Pod::Spec.new do |s|

s.name         = "UCSConvenientView"
s.version      = "1.0.1"
s.summary      = "快速通过storyboard设置一些操作"
s.description  = "快速通过storyboard设置一些常用的操作"
s.homepage     = "http://172.17.16.23:3000/ucs_lius/UCSConvenientView"
s.license      = "MIT"
s.author       = { "ucs_lius" => "linfengwenyou@sina.com" }
s.platform     = :ios, "8.0"
s.source       = { :git => "http://172.17.16.23:3000/ucs_lius/UCSConvenientView.git", :tag => s.version }
s.source_files  = "UCSConvenientView/*.{h,m}"
s.requires_arc = true

s.subspec 'InspectView' do |ss|
ss.source_files = 'UCSConvenientView/InspectView/*.{h,m}'
end

s.subspec 'JMPassword' do |ss|
ss.source_files = 'UCSConvenientView/JMPassword/*.{h,m}'
end

end
```

* 验证文件是否有错

``` obj
pod lib lint  #出问题多对照几遍，后面有附碰到过的几个问题
```

* 验证成功后需要提交到描述库上

``` obj
git add -A && git commit -m “add pod files”
git push origin master
```


#### --------- 创建本地私有库 【git上的库与本地私有库关系等下分析】

* 添加个本地私有库名称与git上的描述库关联起来
``` obj
pod repo add repoName http://*****.git
```

* 将你的库描述文件`.podspec`添加到本地私有库【这一步可能出现问题，看附注问题】
``` obj
pod repo push repoName ****.podspec  # repoName,上一步设置的本地私有库名称 .podspec 私有库描述文件
pod search repoName  #会优点耗时，这个玩不玩无所谓
```


* 操作本地描述库【按需操作】
``` obj
pod repo #查看当前所有的本地库
pod repo remove [name] #移除指定name的本地库
```


### 4. 更新代码发布版本
``` obj
git tag -m “发版本” 1.1.1 #前面提示信息，后面版本号
git push —-tags #将版本发布到git上
```

### 5. 更新私有库支持的版本【维护私有库需要】


初次添加：
``` obj
pod repo push repoName UCSC****.podspec # repoName本地私有库名称  后面是要关联的代码库
```
如果是后续更改，直接去载下私有库，提交更新操作即可：
``` obj
git add . 
git commit -m “” 
git push origin master #origin 地址  master:分支
```
如果想再发一个版本，需要自己进入clone下来的私有Spec库

*  创建文件夹【版本号命名】
*  将调整好版本号或其他属性的.spec文件拷贝过来， push到git上即可


### 6. 测试
编写`Podfile`文件,执行pod install【出现问题看附注】
``` obj
Source ‘https://github.com/CocoaPods/Specs.git’ #github上的，没用到可以不写
Source ‘https:….’ // 私有仓库  #私有库用到的

use_frameworks!
Target ’**Demo’ do
Pod ‘UCSTextField’,'0.0.1'
end
```

******

### 附：基本语法

#### 基本操作
``` obj
git add .                           #标记上传
git commit -m "update description"  #提交描述
git push origin master              #推送到指定分支
git repo                            #查看本地代码库

```

#### 命令容易错的点
单字符命令用`-`  eg: `-r` `-A`

多字符命令用`--` eg: `--verbose` `--all`

#### 区别理解
``` obj
git add . -A -u 区别：
git add . ：他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。
git add -u ：他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）
git add -A ：是上面两个功能的合集（git add --all的缩写）
```

***

### 附：常见问题及错误

#### 1. `pod lib lint 出现的问题`
* 没有将.podspec放到克隆下的库文件夹里面
* s.name没有后缀
* s.source_files 文件要添加进去,第一个路径为库名下你用的文件夹
假如库名为UITextField
如果里面还有一个UITextField文件夹，并且你想加入的代码都在里面，那么配置为：
"UCSTextField/*.{h,m}” 就行了，就已经代表在当前仓库中
* 描述文件要比摘要长一些


提示失败：
``` obj
RPC failed; HTTP 403 curl 22 The requested URL returned error:403 Forbidden. 
```
###### 账户问题，登录了多个账户，但终端默认记住了其中一个，需要从钥匙串中移除对应网站的用户名密码。


#### 2. `pod repo push repoName ***.podsepec` 提示
``` obj
ERROR | [iOS] unknown: Encountered an unknown error ([!] /usr/bin/git clone http://172.17.16.23:3000/test01/UCSTextField.git /var/folders/44/p497c_kn1yjfd8kpd55snlk80000gn/T/d20170310-3511-b578up --template= --single-branch --depth 1 --branch 1.0.0

```
###### 我这边找到的问题是必须要设置tag，但设置了tag就发了版本，勇哥的貌似不需要设置tag就行，还没看到。后续补充！
操作如下：
``` obj
git tag -m “注释” 1.0.0
git push —-tags
```


#### 3. `pod install`提示
`Unable to find a specification for ***`
##### 解决方式
1. 更新本地私有库  【为什么要更新，看👇】
``` obj
pod repo update —verbose
```
2. 安装pod文件
``` obj
pod install
```
### git上的库与本地私有库关系分析【个人理解】： 
`pod install` 是先去本地查找是否存在相应版本号的代码库，找到之后便会根据地址去网络上下载，所以如果找不到，而网络上有，就是因为没有将网络上的改变同步到本地私有库。




---
title: github+hexo搭建个人免费博客
date: 2018-09-18 14:04:43
toc: true
tags:
  - hexo
  - blog
  - github
---
github作为全球最大的开源软件交流平台，除了能够在这里学习、交流、研究IT各个领域的最新技术与成果以外，其实还可以使用github pages服务搭建属于自己的个人免费博客。
这种个人博客不依赖于后台服务，访问的都是静态资源，所以速度很快。

### 准备工作
* 有属于自己的个人github账号，没有的话可以去这里申请一下<https://github.com/join?source=header-home>
* 安装了node.js,官网下载<http://nodejs.cn/>
* 安装了npm，关于npm的介绍可以参考官网<https://www.npmjs.com.cn/>
* 安装了git客户端，下载地址是<https://gitforwindows.org/>

<!--more-->

### 一、安装hexo
###### 1.打开命令行窗口
通过鼠标右键选择Git Bash Here,会弹出git的命令行窗口

![git 命令行窗口](/assets/img/gitbash.png)

或者 win + r 输入cmd，打开Windows自带的命令行窗口

![wind + r](/assets/img/windr.png)

![doc命令行窗口](/assets/img/doc.png)

###### 2.使用npm命令安装hexo

``` bash
$ npm install -g hexo
```
![npm install hexo](/assets/img/npmInstall.png)

由于国内网络问题肯能会安装失败，可以设置淘宝镜像使用cnpm命令安装
``` bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
$ cnpm insatll -g hexo-cli
$ cnpm install hexo --sava
```

![cnpm 设置](/assets/img/cnpmInstall1.png)

![cnpm install hexo](/assets/img/cnpmInstall2.png)

###### 3.验证hexo安装成功
```bash
$ hexo -v
```
![hexo version](/assets/img/hexoVersion.png)

### 二、初始化Hexo
###### 1.新建文件夹
创建用于创建博客的文件夹，这里我用的blog的名字，你可以取其他的文件夹名
![blog文件夹](/assets/img/blogFile.png)

###### 2.输入hexo初始化命令
```bash
$ hexo init
```
![hexo init](/assets/img/initHexo.png)

初始化完成时候的目录结构
![hexo 初始目录结构](/assets/img/construction.png)

###### 3.修改配置文件
使用编辑器修改_config.yml配置文件，这里我使用的是editplus，文件保存编码选择UTF-8
![编辑配置文件](/assets/img/editConf.png)

###### 4.本地部署
```bash
$ hexo g
```
![本地部署](/assets/img/localPub.png)

###### 5.本地启动测试
```bash
$ hexo s
```
![本地启动](/assets/img/localStart.png)

浏览器访问<localhost:4000>
![hexo首页](/assets/img/hexoIndex.png)

如果无法打开首页，可能是端口被占用了，可以切换端口启动

```bash
$ hexo s -p 端口号
```
###### 6.切换主题
如果默认主题不喜欢，可以去[官网](https://hexo.io/themes/)选择喜欢的主题进行修改
这里我自己选择的是[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia),所以已这个主题为例

```bash
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
![yilia主题](/assets/img/yiliaTheme.png)

克隆后的文件都放在themes文件夹下，结构如图
![yilia主题文件](/assets/img/yilia.png)

需要将根目录下的_config.yml修改主题，如果有中文语言应该修改为zh-CN
![修改主题](/assets/img/editConf1.png)

![修改语言](/assets/img/editConf2.png)

需要修改yilia文件夹下的配置文件，可以参照作者的[示例](https://github.com/litten/BlogBackup)修改
![yilia主题配置](/assets/img/yiliaConf.png)

重新部署启动后的主页是这样的
![yilia主页](/assets/img/yiliaIndex.png)

### 三、部署到github上
###### 1.新建仓库
登录github账号点击new repository，新建仓库

![新建仓库](/assets/img/newRep.png)

###### 2.关联仓库
修改hexo根目录_config.yml配置文件，关联新建的仓库

![关联仓库](/assets/img/linkRep.png)

###### 3.安装hexo发布插件
```bash
$ npm install hexo-deployer-git --save
```
![安装hexo发布插件](/assets/img/hexoGit.png)

###### 4.配置生成SSH key
ssh是本地仓库和你的远程仓库的管理密匙，右键打开git bash窗口输入命令
```bash
$ ssh-keygen -t rsa -C "你的github注册邮件地址"
```
连续3次回车，会在用户目录生成文件目录
![ssh文件目录](/assets/img/sshFile.png)

###### 4.github账号添加ssh
登录github，选择setting->SSH and GPG keys -> new SSH key
![seeting](/assets/img/setting.png)

![sshKeys](/assets/img/sshKeys.png)

![拷贝ssh](/assets/img/copySSH.png)

在git bash中验证是否添加成功
```bash
$ ssh -T git@github.com
```
如图提示则表示验证成功

![验证](/assets/img/sshSuccess.png)

###### 5.设置用户名邮箱
```bash
$ git config --global user.name "your name"  
$ git config --global user.email "your_email@youremail.com"
```

### 四、上传博客
如果顺利到这一步，恭喜你马上就要成功了，只需要输入一下命令上传博客即可
```bash
$ hexo g 
$ hexo d
```

# 结束
完成上面的所有步骤，属于你自己的免费博客就搭建成功了，你可以直接使用自己的访问地址访问自己发布的博客了，
这是我的[博客](https://jordanhank.github.io/),欢迎你来访问，如果你觉得这篇博客对你有所帮助，可以点击下面的打赏，支持一下！
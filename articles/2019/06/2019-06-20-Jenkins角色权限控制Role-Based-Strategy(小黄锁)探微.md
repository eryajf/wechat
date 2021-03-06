Jenkins角色权限控制Role-Based-Strategy（小黄锁）探微

- - -

## title: **Jenkins角色权限控制Role-Based-Strategy（小黄锁）探微**

- description: 利用Jenkins自带插件实现小规模（两百人以内）权限管理。
- author: eryajf
- original: [原文链接](http://www.eryajf.net/1445.html "原文链接")
- date: 2019-06-20
- tags: Jenkins，plugin

- - -

![](http://t.eryajf.net/imgs/2019/05/2edabcc2be850178.jpg)

如果公司比较小，有可能所有环境（此处的环境指测试，预发，线上）的Jenkins都在一台之上，那么在这种情况下，做好Jenkins项目视图以及权限的控制就显得非常重要了。

如果公司稍微大一些，项目可能七七八八有不少，如此多项目的情况下，如何让负责`开发a`的同学就只能看到`a项目`，负责`开发b`的同学只能看到`b项目`呢，这就要请出我们今天的主角了，Jenkins当中的`角色控制`。

## 1，环境说明

- 主机：CentOS-7.5
- Jdk：jdk1.8.0_192
- Tomcat：8
- Jenkins：2.177
- Role-Based Strategy：2.10

如上环境的准备工作本文就不赘述了。

## 2，安装插件。

Jenkins的角色控制依赖于插件：[Role-Based Strategy](https://wiki.jenkins.io/display/JENKINS/Role+Strategy+Plugin "Role-Based Strategy")。点击链接可以跳转到插件对应的官方文档。

直接在系统管理当中选择插件管理，搜索之后直接安装即可。因为在Jenkins配置当中插件对应的图标是一个黄色的小锁，因此我亲切的将之称为`小黄锁`。

## 3，启用此功能。

正常的使用方式就是在系统管理界面有一个`Manage and Assign Roles`。

但是仅仅安装完插件是不能看到这个功能的，需要到`系统管理`的`全局安全配置界面`进行一下配置。

系统默认的是`登录用户可以做任何事情`，现在改成用`角色控制`的方案，上边也可以开启`允许用户注册`。

详细情况如下图所示：

![](http://t.eryajf.net/imgs/2019/06/06391c8c413dce31.png)

配置完成之后点击保存，再去`系统配置`里边就能看到`小黄锁`（Manage and Assign Roles）出现啦。

## 4，视图规划。

今天我们来从头到尾详细梳理一下，拿真实的例子来进行一波演练。

先看我准备的一些项目。

![](http://t.eryajf.net/imgs/2019/06/70e7121dfbdff7c1.png)

当然常规来说，我们肯定都是一个项目一个项目创建，然后创建的时候就已经进行了规划分类，现在为了讲解，我先创建了这么8个项目。

大致可以分出四个视图：

* 1，a-test
* 2，a-online
* 3，b-test
* 4，b-online

可以直接点击左侧的新建视图来进行项目的分类管理。

![](http://t.eryajf.net/imgs/2019/06/4e0a933473da1be6.png)

如上操作，创建视图：

![](http://t.eryajf.net/imgs/2019/06/201f782a19a9024a.png)

将对应的项目选中即可，虽然这步操作与今天的主题关系不大，但是也是日常管理的一个重要项，整理完毕之后，如下图：

![](http://t.eryajf.net/imgs/2019/06/c66c3bb3a27140f1.png)

## 5，管理角色。

如上边操作的，我们已经将不同的项目进行归类，此时对于运维人员来说，a项目的开发人员只需要能够看到`a-test` 视图下的项目，然后能够进行构建用于测试即可，线上的不需要看到，当然，更不能让a项目的开发人员看到b的项目，想要完成这些操作，就需要先在管理角色界面进行一些配置了。

接下来进入插件配置的环节。

### 1，Global roles

是最高统领的一个权限管理，配置某用户（这个用户类似于gitlab当中的master与developer的意义）的权限是什么。

默认的admin就是拥有所有权限，我们可以创建一个开发用户并配置其权限。

![](http://t.eryajf.net/imgs/2019/06/fb22a4ac5be62c4e.png)

以上配置的意思是`develop用户`对Jenkins全局都是`只能看不能摸`的权限。

### 2，Project roles

这个是详细项目权限的管理。

此处可以添加项目，通过`正则`进行匹配，从而达到不同的项目以及不同的权限的目的。

如下图所示：

![](http://t.eryajf.net/imgs/2019/06/187e0ce143aa3371.png)

现在插件最新版本已经支持点击正则规则查看匹配到的项目，从而验证自己所写的规则是否符合需求所要求的。

现在点击一下，可以看到符合我们需要的规则：

![](http://t.eryajf.net/imgs/2019/06/ae51ab9aa43621d1.png)

一般情况下，只要建立项目的时候`名称足够规范`，那么这里的权限设置也都比较简单的，通过对项目进行正则匹配即可，权限的话，酌情进行分配，如上所分配的权限，是最基础的读，构建，以及取消的权限，就足够日常开发使用了。

### 3，Slave roles

顾名思义，这是有了Jenkins集群之后，进行的权限控制，这里先不多谈，等以后谈到Jenkins集群部署的时候，再来说这个东东，或者就不再说了，因为基本上在工作当中不会用到。

## 6，用户管理。

公司新来了小伙伴，或者你的Jenkins刚刚做好，需要让大家都能够登陆，然后看到其对应的项目，那么第一步就是先来创建用户，当然也可以在公司群里吼一声，让大家各自进行注册，然后再来进行统一权限分配管理。

点击系统管理，管理用户，即可进入用户数据库，用户注册页面如下：

![](http://t.eryajf.net/imgs/2019/06/f0262f56ea049f42.png)

* 1，用户名：个人用户名的中文拼音。如张三：zhangsan，李四：lisi。
* 2，密码：自定义。
* 3，全名：可以沿用用户名，不过此处建议写成自己名字的中文。
* 4，邮箱地址。

可能刚刚这段说明有些小啰嗦，但是正是因为对简单的东西的啰嗦，才形成了规范化的一个进展，比如，我在权限分配的时候，不用问你你注册的名字是啥，就直接能够给你授权了，如果张三起了个Tom，李四起了个Jerry，那你运维去吧，够你运维的了。

现在我就创建了a开发人员张三以及b开发人员李四。

如下图：

![](http://t.eryajf.net/imgs/2019/06/39a975fc1991136d.png)

## 7，分配角色。

我们进入到小黄锁的第二个选项当中。

先上最终配置，然后再来讲解：

![](http://t.eryajf.net/imgs/2019/06/85e30b0be7d2b69d.png)

当我们需要对一个新注册用户授权的时候，需要做两件事情：

* 第一在全局权限里，给`Anonymous`用户添加选中`develop`的权限。
在分配角色的global role里边，配置`Anonymous`拥有`develop`的那个权限，这样以来，默认所有用户都会拥有首页读的权限，而不会有项目的权限，然后接着把用户在下边针对项目授权即可，从而不需要将每个用户都授权`develop`的权限了。
当然，可能这样“裸奔”会被认为不够安全，尤其是针对那些开放了外网访问的Jenkins来说，别急，我这里还有一个办法，根据官方文档描述说：
    * Jenkins有两个内置角色
        * `Anonymous`：尚未登录的用户
        * `authenticated` ： 登录用户
        如此以来，我们这里只需配置`authenticated`拥有`develop`的那个权限，然后把`Anonymous`的权限取消掉，这样以来，所有未经登陆的用户，就无法查看任何内容啦（关于这一点，就留给大家自己去测试了）！
* 第二就是在项目权限里添加，让其对具体项目拥有具体权限。

配置完成，别忘了点击最下边的`save`进行保存。

通过这张图，我们应该可以非常清楚理解为什么，神通广大的a开发人员张三，在运维人员的控制之下，却只能看到a-test的项目了吧。

究竟是不是如我所说的这样呢，我们登陆一下张三的账号看下是不是真的。

![](http://t.eryajf.net/imgs/2019/06/d9c346367a48cb9a.png)

接着再来看看李四的情况。

![](http://t.eryajf.net/imgs/2019/06/1b983f225b12c7cb.png)

ok，到这里，基本上经过这样一趟洗礼之后，如果你看的认真并照做了的话，相信你就已经掌握Jenkins当中的角色控制啦！！

如果还有什么疑惑，以及在工作中有什么坑之类的，欢迎您在下方留言区域一起交流！！！

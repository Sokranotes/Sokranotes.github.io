---
layout: post
title: Hello World!
date: 2023-10-1 20:52:19 +0800
Author: Sokranotes
tags: [blog building, recording, learning]
comments: true
categories: recording blog_building
toc: true
typora-root-url: ..
---

# Hello, world!


![Sokranotes](/assets/img/2021-07-13-hello jekyll/avatar.jpg)


## 博客本地搭建过程

使用Windows10 LTSC在虚拟机环境下验证了整个过程

Windows10 LTSC版本：SW_DVD9_WIN_ENT_LTSC_2021_64BIT_ChnSimp_MLF_X22-84402.iso

### 1 安装依赖

1.1 [RubyInstaller](https://jekyllrb.com/docs/installation/windows/)及相关依赖

（1）安装rubyinstaller-devkit-3.2.2-1-x64.exe

安装最后一步，勾选该项：Run 'ridk install' to set up MSYS2 and the development toolchain. MSYS2 is required to install gems with C extensions.

（2）在弹出的cmd命令行中**输入3**并回车（This is needed for installing gems with native extensions.）

等待cmd命令运行完成，回车退出命令行

（3）随后打开新的cmd命令行，运行下面的命令安装Jekyll和Bundler

``````cmd
gem install jekyll bundler
``````

1.2 [Node.js](https://nodejs.org/en)

（1）安装node-v18.18.0-x64.exe

> Automatically install the necessary tools. Note that this will also install Chocolatey. The script will pop-up in a new windows after the installation completes.

暂时没勾选该项目，勾选上安装失败，推测是网络原因。

> **[参考：](https://www.cnblogs.com/liangxfng/p/12675115.html)**
>
> 构建工具是因为一些npm模块需要使用C/C++编译，如果想要编译这些模块，则需要安装这个工具。如果不安装这个构建工具，在之后使用 npm 安装模块的时候，会报错：gyp ERR! find Python

1.3 [Git](https://git-scm.com/download/win)

安装Git-2.37.0-64-bit.exe

1.4 安装可选工具[Github Desktop](https://desktop.github.com/)、[VS Code](https://code.visualstudio.com/)等

### 2 初始化项目

2.1 fork [cotes2020/jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)项目并clone到本地

这里使用Github Desktop将fork的代码clone下来（注意网络环境）

2.2 打开Git Bash，在项目根目录下运行该命令，执行tools目录下的init脚本进行初始化（pwd可查看当前路径）

``````bash
bash tools/init
``````

2.3 安装依赖项

```bash
bundle
```

2.4 在cmd or Git Bash中运行该命令进行本地预览

``````bash
bundle exec jekyll s
``````

## Github Pages

本地将修改commit之后，push至Github，push会自动触发Github Action，进行build和deploy。也可以在Github项目的repo下，Action --> Build and Deploy中Run workflow手动进行。

## Bug及可能会遇到的问题

#### Github Actions过程，build

（1） Test site中，提示 internal script reference /assets/js/dist/***.min.js does not exist

故障原因：/assets/js/dist/路径下缺失6个文件：misc.min.js、commons.min.js、post.min.js、home.min.js、categories.min.js、page.min.js。

解决办法：参考[rootjhon.github.io](https://github.com/Rootjhon/rootjhon.github.io)项目，手动添加. (notice that the config of .gitignore may ignore the files added)

（2） Test site中，提示 http://xxx is not an HTTPS link

解决办法：将对应链接修改为https链接.

（3） Test site中，提示 image xxx.png does not have an alt attribute

解决办法：给图片添加alt属性，空属性也可.

（4）Test site中，提示bundler: failed to load command: htmlproofer 

解决办法：[change history](https://github.com/Sokranotes/Sokranotes.github.io/commit/e6ada962e32b58d54332bc8e30fb222ba5e40bac#diff-5a50c15a0022b9d638b5f1f0e6505871f100bfa56611c6a89242c566143cde01)

ref: [bug issues](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1233), [pages-deploy.yml](https://github.com/Rootjhon/rootjhon.github.io/blob/master/.github/workflows/pages-deploy.yml), [change history](https://github.com/Rootjhon/rootjhon.github.io/commit/924347630fde7a0ab9f787763eb8be789fd1c10c)

#### Github Actions过程，deploy

Branch "xxx" is not allowed to deploy to github-pages due to environment protection rules.

故障原因：environment protection rules限制，branch改名未及时修改environment protection rules也会出现该问题。

解决办法：Github Repo的Setting中， Code and automation下的Environment，github-pages protection rule中，将需要deploy的branch name添加到Deployment branches中。

## Git使用相关

#### 1 合并不同repo中存在conflict的代码

fork的upstream(master)进行了更新，自己fork的origin(main)也进行了更新，现在需要将upstream repo的master中的commit同步到origin repo的main branch，Github提示有conflict，Can’t automatically merge.

理论流程：将upstream repo的master branch fetch到origin main上，解决conflict后，进行commit和push。

解决：在Github Desktop中切换到origin repo的main branch，菜单栏选择branch --> Merge into current branch... 或者是 Rebase current branch...进行同步操作，当存在conflict时，会提示进行修改，确定一个最终状态修改好之后进行commit和push。

参考：[git分支与冲突](https://www.bilibili.com/video/BV1c54y1H7j7/)，[git rebase和 git merge](https://www.bilibili.com/video/BV1VG411F7rB/)

#### 2 撤回push的commit

commit了两次，push到了Github，然后发现commit有问题，该如何撤回commit和push

在本地打开Git Bash

使用```git reflog```确定自己想要回到的节点的commit及版本号

假设想要回到的commit版本号位```123456b```，通过下面的命令直接将本地代码回到那个版本（直接改变源码，后面的修改在本地直接丢失）

``````bash
git reset --hard 123456b
``````

然后push本地的代码，这样就回到了```123456b```版本号对应的commit位置。

``````
git push repo_name branch_name --force
``````

参考：[git push提交成功后如何撤销回退](https://blog.csdn.net/guozhaohui628/article/details/78922946)

3 参考：[git commit 提交规范](https://zhuanlan.zhihu.com/p/90281637)

## 文章头部格式说明（need revise）

```
---
layout: post
title: Hello World!
date: 2023-10-1 20:52:34 +0800
Author: Sokranotes
tags: [blog building, ]
comments: true
categories: technology blog_building
toc: true
typora-root-url: ..
---
```


说明：

1. title：文章标题
2. data：文章显示的日期，日期 时间 时区(默认为0时区)
3. toc：文章侧边目录是否开启（need confirm）
4. categories: 设置categories，子目录直接在后面添加，以空格隔开，最多支持二级目录（need confirm）
5. 图片位置说明：上传图片不能置于/_post路径下，需放在/assets目录下（need confirm）
6. typora-root-url：设置在typora中的根目录，方便typora中图片显示

## 个性化修改

#### done

1 replace the pics with the same size pics at /assets/img/favicons/

2 chage LICENSE from MIT to [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

3 change the title, tagline, github info, social info, avatar(the avatar save at /assets/img/favicons/avatar.jpg) and disqus in _config.yml

4 update the authors.yml, share.yml in dir /_data/

5 update the sidebar.html in dir /_includes/.(add a pic of love)

6 update about.md in dir /_tabs/

7 update the content of previous articles

#### previous (todo)

在Jekyll的主题Chirpy的基础之上进行了一点小修改（之前的个性化修改待同步）

1. 侧边栏颜色及文字颜色，修改配色
2. tags中展开显示，提高可读性，借鉴了 [LOFFER](https://github.com/FromEndWorld/LOFFER) 的设计
3. 侧边栏下方图标修改，去掉多余的图标，并进行了分隔
4. 文章分享修改
5. 添加MathJax以更好的支持LaTeX公式，[给 Jekyll 博客添加 Latex 公式支持](https://bryceyang.github.io/add-eqution-support-in-jekyll/)
6. 开启基于Disqus的评论（need confirm）

#### plan (todo)

关闭图片链接上的shimmer效果（new）

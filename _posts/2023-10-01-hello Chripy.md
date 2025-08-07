---
layout: post
title: 博客搭建及使用说明
date: 2023-10-1 20:52:19 +0800
Author: Sokranotes
tags: [blog building, recording, learning]
comments: true
categories: recording blog_building
toc: true
typora-root-url: ..
---


![Sokranotes](/assets/img/2021-07-13-hello jekyll/avatar.jpg)

## 一、Chirpy主题博客环境部署

**20250807测试有效**

如果你也想基于Chirpy构建自己的博客，建议参考[Chirpy Getting Started](https://chirpy.cotes.page/posts/getting-started/)，使用starter模板后通过docker构建环境
注意：如果网络环境为大陆，Using Dev Containers的第三步，clone repo的过程中，建议先clone到本地，而后在vscode中选择reopen in container；不建议按照官方[clone your repo in a container volume](https://code.visualstudio.com/docs/devcontainers/containers#_quick-start-open-a-git-repository-or-github-pr-in-an-isolated-container-volume)，容易出现网络问题。

## 二、本博客使用说明

**20250807记录**

- push触发Github Actions进行自动构建：博客文章写好并commit后，直接push至Github，push会自动触发Github Actions，进行build和deploy。

- 手动进行自动构建：在Github项目的repo下，Action --> Build and Deploy中Run workflow手动进行。

### 当前版本主要修改部分
```
_posts/*
assets/*
_config.yml
README.md
_tabs/about.md
_data/share.yml
_data/contact.yml
_data/authors.yml（starter版本中不显式存在）
_includes/sidebar.html（starter版本中不显式存在）
```

### Obsidian撰写文章与博客联动
注意：Obsidian需要做相关设置，以便启用md标准链接格式
![Obsidian Settings](/assets/img/2021-07-13-hello jekyll/Obsidian Settings.png)
```python
import shutil
from pathlib import Path
import re
import subprocess

SOURCE_OBSIDIAN = Path("/GitHub Repositories/ObsidianRepo/博客文章库/对外发布文章")
TARGET_BLOG = Path("/GitHub Repositories/Sokranotes.github.io")

def convert_links_to_blog_format(file_path: Path):
    if file_path.suffix.lower() != ".md":
        return
    with file_path.open("r", encoding="utf-8") as f:
        content = f.read()

    # 1. 转换资源路径 ../assets/ -> /assets/
    content = re.sub(r"(?<!!)\(\.\./assets/", r"(/assets/", content)
    content = re.sub(r'src=["\']\.\./assets/', r'src="/assets/', content)

    # 2. 资源链接中的 %20 转换为空格
    content = re.sub(r"(\]\(|src=['\"]/assets/)(.*?)([)'\"])", 
                     lambda m: f"{m.group(1)}{m.group(2).replace('%20', ' ')}{m.group(3)}", 
                     content)

    with file_path.open("w", encoding="utf-8") as f:
        f.write(content)

def convert_links_to_obsidian_format(file_path: Path):
    if file_path.suffix.lower() != ".md":
        return
    with file_path.open("r", encoding="utf-8") as f:
        content = f.read()

    # 1. 转换资源路径 /assets/ -> ../assets/
    content = re.sub(r"(?<!!)\(/assets/", r"(/assets/", content)
    content = re.sub(r'src=["\']/assets/', r'src="/assets/', content)

    # 2. 空格替换为 %20
    content = re.sub(r"(\]\(|src=['\"])\.\./assets/(.*?)([)'\"])",
                     lambda m: f"{m.group(1)}../assets/{m.group(2).replace(' ', '%20')}{m.group(3)}", content)

    with file_path.open("w", encoding="utf-8") as f:
        f.write(content)

def copy_and_convert(source: Path, target: Path, direction: str):
    if not source.exists():
        print(f"Source path does not exist: {source}")
        return

    # _posts 和 _tabs：直接整体覆盖
    for item in ['_posts', '_tabs']:
        src_path = source / item
        tgt_path = target / item

        if tgt_path.exists():
            shutil.rmtree(tgt_path)
        shutil.copytree(src_path, tgt_path)

        print(f"Copied {src_path} → {tgt_path}")

        # 转换 markdown 文件中的资源路径
        for file in tgt_path.rglob("*.md"):
            if direction == "to_blog":
                convert_links_to_blog_format(file)
            elif direction == "to_obsidian":
                convert_links_to_obsidian_format(file)

    # assets 目录：只覆盖 img 和 music，其它内容保留
    src_assets = source / "assets"
    tgt_assets = target / "assets"
    tgt_assets.mkdir(parents=True, exist_ok=True)

    for subdir in ["img", "music"]:
        src_sub = src_assets / subdir
        tgt_sub = tgt_assets / subdir

        if tgt_sub.exists():
            shutil.rmtree(tgt_sub)
        if src_sub.exists():
            shutil.copytree(src_sub, tgt_sub)
            print(f"Copied {src_sub} → {tgt_sub}")

    # 转换 markdown 文件中的资源路径
    for file in tgt_assets.rglob("*.md"):
        if direction == "to_blog":
            convert_links_to_blog_format(file)
        elif direction == "to_obsidian":
            convert_links_to_obsidian_format(file)

def git_commit_and_push(repo_path: Path):
    try:
        # 检查是否有变化
        result = subprocess.run(["git", "status", "--porcelain"], cwd=repo_path, capture_output=True, text=True)
        if not result.stdout.strip():
            print("ℹ️  没有检测到变更，跳过提交和推送。")
            return

        subprocess.run(["git", "add", "."], cwd=repo_path, check=True)

        print("\n📝 请填写符合 commitlint 规范的提交信息：")
        print("常用类型：")
        print("  • feat: 新增文章或功能")
        print("  • docs: 修改或补充文章内容（不影响功能）")
        print("  • fix: 修复错误")
        print("  • style: 格式更新（如空格、缩进）")
        print("  • chore: 杂项或脚本自动更新\n")

        summary = input("✏️  Summary（例如 feat: update blog posts）：").strip()
        if not summary:
            print("❌ 提交信息不能为空，已中止提交。")
            return
        description = input("📄 Description（可选，可回车跳过）：").strip()

        full_message = summary if not description else f"{summary}\n\n{description}"

        subprocess.run(["git", "commit", "-m", full_message], cwd=repo_path, check=True)
        subprocess.run(["git", "push"], cwd=repo_path, check=True)
        print("✅ Git push 成功。")

    except subprocess.CalledProcessError as e:
        print("❌ Git 操作失败，请手动检查：", e)

def main():
    print("\n📌 请选择要执行的操作：")
    print("1. 🔁 从 Obsidian 转换并同步到博客（默认）")
    print("2. ⬅️ 仅将博客内容导入到 Obsidian（一次性）")
    print("3. 🚀 仅推送博客仓库更新到远程")
    print("0. ❌ 退出")

    choice = input("\n请输入序号回车（默认 1）: ").strip() or "1"

    if choice == "1":
        copy_and_convert(SOURCE_OBSIDIAN, TARGET_BLOG, direction="to_blog")
        git_commit_and_push(TARGET_BLOG)

    elif choice == "2":
        copy_and_convert(TARGET_BLOG, SOURCE_OBSIDIAN, direction="to_obsidian")

    elif choice == "3":
        git_commit_and_push(TARGET_BLOG)

    elif choice == "0":
        print("✅ 已退出。")
    else:
        print("❌ 无效选择。")

    input("\n🔚 操作完成，按回车键退出...")

if __name__ == "__main__":
    main()

```

效果：
![script work](/assets/img/2021-07-13-hello jekyll/script work.png)

## 三、文章格式说明

### （一）文章头部格式说明

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

### （二）注意事项

**1、图片必须要有alt属性，哪怕为空**。

**2、http禁用，必须使用https。**

**3、提交commit必须遵循commitlint规范。**

否则会报husky - commit-msg script failed (code 1)错误。

type: fix, docs, feat, style等

subject:描述，必须小写

示例如下：type的冒号后面必须要有空格

```
fix: fix a bug of stack over flow
```


## 四、Git使用相关参考

### （一）合并不同repo中存在conflict的代码

fork的upstream(master)进行了更新，自己fork的origin(main)也进行了更新，现在需要将upstream repo的master中的commit同步到origin repo的main branch，Github提示有conflict，Can’t automatically merge.

理论流程：将upstream repo的master branch fetch到origin main上，解决conflict后，进行commit和push。

解决：在Github Desktop中切换到origin repo的main branch，菜单栏选择branch --> Merge into current branch... 或者是 Rebase current branch...进行同步操作，当存在conflict时，会提示进行修改，确定一个最终状态修改好之后进行commit和push。

参考：[git分支与冲突](https://www.bilibili.com/video/BV1c54y1H7j7/)，[git rebase和 git merge](https://www.bilibili.com/video/BV1VG411F7rB/)

### （二）撤回push的commit

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

本地branch提交云端（本地退回之后强制覆盖云端）

```
git push origin --force
```

### （三）dev部署成功后同步至master分支

查看分支

```
git branch
```

切换至master分支

```
git checkout master
```

将dev分支合并到master分支

```
git merge dev
```

将本地的master分支push至云端

```
git push origin --force
```

### （四）git参考

1、[git push提交成功后如何撤销回退](https://blog.csdn.net/guozhaohui628/article/details/78922946)

2、[git commit 提交规范](https://zhuanlan.zhihu.com/p/90281637)

## 五、CHANGELOG
- 20250807: 随项目更新，相关依赖的升级和更新，旧版本本地测试环境失效，多次尝试在本地搭建预览环境未果后，决定按照官方指引依赖Docker在本地部署环境。
	- 尝试本地部署依赖环境，发现Windows11下存在依赖及网络环境等问题，如：Ruby安装后ridk install过程，bundle install过程。

### Todo
- 将Chirpy作为git submodule，基于装饰器模式（Decorator Pattern）的思想，将个性化的内容作为装饰层，既方便跟踪Chirpy的更新，也保留自己的更改，同时避免修改造成的冲突。
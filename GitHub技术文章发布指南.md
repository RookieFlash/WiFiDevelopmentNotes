# GitHub 技术文章发布指南

| 项目 | 说明 |
| --- | --- |
| 适用对象 | 刚注册 GitHub 账号、想把个人技术文章发布到 GitHub 的用户 |
| 目标 | 将本地技术文章发布到 GitHub,在网页 / 客户端正常显示,并可进一步搭建博客网站 |
| 运行环境 | Ubuntu / Linux 桌面(其他系统差异见附录 C) |
| 文档版本 | v2.0(2026-09-06,新增第三章多账号专章) |

---

## 目录

- [第一章 方案原理与整体流程](#第一章-方案原理与整体流程)
- [第二章 本地环境准备(单账号,一次性)](#第二章-本地环境准备单账号一次性)
- [第三章 两个 GitHub 账号的完整配置(多账号专章)](#第三章-两个-github-账号的完整配置多账号专章)
- [第四章 在 GitHub 创建仓库(一次性)](#第四章-在-github-创建仓库一次性)
- [第五章 克隆仓库到本地(一次性)](#第五章-克隆仓库到本地一次性)
- [第六章 文章的准备与组织](#第六章-文章的准备与组织)
- [第七章 提交并推送到 GitHub](#第七章-提交并推送到-github)
- [第八章 在网页和客户端查看](#第八章-在网页和客户端查看)
- [第九章 进阶:开启 GitHub Pages 博客站](#第九章-进阶开启-github-pages-博客站)
- [第十章 日常维护流程](#第十章-日常维护流程)
- [第十一章 常见问题 FAQ](#第十一章-常见问题-faq)
- [附录 A:Git 常用命令速查](#附录-agit-常用命令速查)
- [附录 B:Markdown 语法速查](#附录-bmarkdown-语法速查)
- [附录 C:其他操作系统说明](#附录-c其他操作系统说明)

---

## 第一章 方案原理与整体流程

### 1.1 为什么选择 GitHub 存放技术文章

- **原生渲染 Markdown**:GitHub 会自动把 `.md` 文件渲染成排版良好的网页,标题、代码块、表格、图片都能正常显示,不需要任何额外配置;
- **免费**:公开仓库免费无限使用;
- **版本管理**:每次修改都有历史记录,误删误改可以找回,这就是 Git 的核心能力;
- **可升级为博客**:开启 GitHub Pages 后,免费获得一个 `https://用户名.github.io` 的正式网站,适合做成技术博客。

### 1.2 两种使用层次

| 层次 | 做法 | 效果 | 适合场景 |
| --- | --- | --- | --- |
| 层次一:仓库直读 | 文章直接放进仓库 | 在 github.com 上点开文章文件即可阅读 | 只想有个地方存放和分享文章 |
| 层次二:博客网站 | 额外开启 GitHub Pages + MkDocs | 获得独立网址、带侧边导航/搜索的博客站 | 想长期经营技术博客 |

**建议**:先完成层次一(第二~八章),看到效果后再按第九章升级到层次二,不要一步到位。

### 1.3 整体流程一览

```text
┌─────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ 本地装 git   │ → │ 配置 SSH 密钥 │ → │ 网页创建仓库  │ → │ 克隆到本地     │
└─────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
        ↓
┌─────────────┐   ┌──────────────┐   ┌──────────────────────────┐
│ 文章放入仓库 │ → │ 提交并推送    │ → │ 网页/客户端查看(可升级博客) │
└─────────────┘   └──────────────┘   └──────────────────────────┘
```

### 1.4 一次性工作 vs 日常重复工作

- **一次性**(第二章~第五章、第九章):装 git、配密钥、建仓库、克隆、开 Pages
- **日常重复**(第七章、第十章):写新文章 → 复制进仓库 → `git add` → `git commit` → `git push`,共三条命令

---

## 第二章 本地环境准备(单账号,一次性)

> 本章面向**单账号**用户。同时使用多个 GitHub 账号的用户,请直接阅读第三章"两个 GitHub 账号的完整配置"。

### 2.1 安装 Git

```bash
sudo apt install git
git --version          # 能显示版本号即安装成功
```

### 2.2 配置提交身份

> 本节针对只有一个 GitHub 账号的情况。**多账号用户请跳过本节全局配置**,直接按第三章"两个 GitHub 账号的完整配置"进行配置,以免全局配置串号。

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "注册GitHub时使用的邮箱"
```

说明:

- `user.name` 只是提交记录上显示的署名,与实际账号认证无关(认证靠 2.3 的 SSH 密钥);
- 不想暴露真实邮箱,可用 GitHub 提供的匿名邮箱:`用户名@users.noreply.github.com`(在 GitHub 网页 Settings → Emails 里可以查到自己对应的匿名地址);
- 验证配置:`git config --global --list`。

### 2.3 配置 SSH 密钥(推荐)

GitHub 自 2021 年起禁止用密码推送代码,SSH 密钥是开发者最常用的认证方式,配置一次以后推送免输密码。

#### 2.3.1 生成密钥

```bash
ssh-keygen -t ed25519 -C "注册GitHub时使用的邮箱"
# 三个提示都直接按回车,使用默认路径 ~/.ssh/id_ed25519,不设密码
```

> 多账号提醒:如果 `~/.ssh/` 下已存在其他 GitHub 账号的密钥(即 `id_ed25519` 已存在),回车生成时会提示是否覆盖,**切勿覆盖**——请改用第三章的方法,用独立文件名为新账号生成密钥。

查看并复制公钥内容:

```bash
cat ~/.ssh/id_ed25519.pub
# 输出形如 ssh-ed25519 AAAA...xxxx 邮箱 的一整行,全部复制
```

#### 2.3.2 将公钥添加到 GitHub

1. 浏览器打开 <https://github.com/settings/keys>
2. 点击 **New SSH key**
3. **Title** 随意填写(如"我的 Linux 电脑",用于辨认是哪台机器)
4. **Key type** 保持默认 Authentication Key
5. **Key** 粘贴上一步复制的公钥
6. 点击 **Add SSH key**,可能需要输入账号密码确认

#### 2.3.3 测试连接

```bash
ssh -T git@github.com
# 首次连接询问指纹是否可信,输入 yes
# 成功输出:Hi 你的用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

出现 `Hi 你的用户名!` 即表示打通。

#### 2.3.4 SSH 常见报错处理

| 报错 | 原因 | 处理 |
| --- | --- | --- |
| `Permission denied (publickey)` | 公钥没添加成功 / 连接地址写错 | 重新执行 2.3.2;确认测试地址是 `git@github.com` 而非其他主机名 |
| `Host key verification failed` | 首次连接没有确认指纹 | 重新执行 `ssh -T git@github.com` 并输入 yes |
| 公司/代理网络下连不通 | 22 端口被网络屏蔽 | 在 `~/.ssh/config` 中加入:<br>`Host github.com`<br>`  Hostname ssh.github.com`<br>`  Port 443` |

### 2.4 备选方案:HTTPS + 个人访问令牌(PAT)

如果 SSH 配置遇到无法解决的网络问题,可以用 HTTPS 方式:

1. GitHub 网页 → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)
2. 勾选 `repo` 权限,生成令牌并复制保存(只显示一次)
3. 克隆时使用 HTTPS 地址:`git clone https://github.com/用户名/仓库名.git`
4. 首次 push 时,用户名填 GitHub 用户名,**密码填令牌**(不是账号密码)
5. 免重复输入,可让 git 记住令牌:`git config --global credential.helper store`

> 安全提示:`credential.helper store` 会把令牌明文保存在 `~/.git-credentials`,仅建议在个人电脑上使用。

---

## 第三章 两个 GitHub 账号的完整配置(多账号专章)

本章针对同时使用两个(或多个)GitHub 账号的场景,完整覆盖:密钥生成、`~/.ssh/config` 配置、仓库级身份、以及提交失败时的排查方法。单账号用户可跳过本章。

### 3.1 核心原理:提交身份与认证是两回事

多账号配置容易乱,是因为把两件事混在一起了:

| | 提交身份 | 认证 |
| --- | --- | --- |
| 配置项 | `user.name` / `user.email` | SSH 密钥 |
| 作用 | 决定提交记录上显示的署名 | 决定"以哪个账号的身份操作 GitHub" |
| 影响范围 | 只影响显示,不影响权限 | 决定能不能推送某个仓库 |
| 多账号做法 | 每个仓库单独设置(见 3.7 节) | 每个账号一把独立密钥(见 3.3 节) |

由此得出多账号配置的三个组成部分:**仓库级身份 + 每账号独立密钥 + SSH 别名路由**。

### 3.2 配置前盘点:确认两边账号的真实信息

开始配置前,先确认三件事:

1. **两个账号的真实登录名**。以 GitHub 自己的输出为准:浏览器登录后点右上角头像,菜单里显示的名字就是登录名;`ssh -T` 成功后 `Hi` 后面的名字也是。**注册时以为的用户名和真实登录名可能不一致**(实战案例中曾把账号记成 `username02_2022`,实际登录名是 `username02`)。
2. **现有密钥盘点**,避免后面生成时覆盖:

```bash
ls -l ~/.ssh/
```

   已有 `id_ed25519` 的话,用 `ssh -T git@github.com` 验证它属于哪个账号(`Hi` 后面的名字)。
3. **准备新账号的邮箱**:生成密钥时写入注释用。

### 3.3 为新账号生成独立密钥

```bash
ssh-keygen -t ed25519 -C "新账号的邮箱" -f ~/.ssh/id_ed25519_github2
# -f 指定独立文件名;不要用默认名,以免覆盖旧账号的 id_ed25519
```

生成两个文件:私钥 `id_ed25519_github2`(自己保留,不外传)、公钥 `id_ed25519_github2.pub`(上传给 GitHub)。查看公钥:

```bash
cat ~/.ssh/id_ed25519_github2.pub
```

### 3.4 把公钥添加到对应账号

1. 登录**新账号**,打开 <https://github.com/settings/keys>
2. New SSH key → Title 随意(如"我的 Linux 电脑")→ 粘贴公钥 → Add SSH key

> 最容易踩的坑:**公钥加到了哪个账号,这把密钥就代表哪个账号**。给新账号配的密钥却粘贴到了旧账号的 settings 里,是排查时最常见的错误。

### 3.5 配置 ~/.ssh/config:别名路由

编辑(或新建)`~/.ssh/config`,完整示例:

```text
# 账号一(默认):走 github.com
Host github.com		/* 这里的 github.com 可以改成 github 账号名，便于区分 */
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519	/* 这里改为对应账号的key文件名 */

# 账号二(本技术文章账号):走别名 github2
Host github2
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github2
```

四个字段逐个说明:

| 字段 | 说明 | 能否改 |
| --- | --- | --- |
| `Host` | **本地别名**,只在本机使用。可自由命名,包括直接用账号名(如 `Host username02`);但不能是真实域名(尤其不能两个账号都写 `Host github.com`),不能重复 | 可自定 |
| `HostName` | 真实连接地址,固定为 `github.com` | 不能改 |
| `User` | GitHub SSH 服务的固定登录名,**永远是 `git`**,不要改成你的账号名(账号身份由密钥决定,不由登录名决定) | 不能改 |
| `IdentityFile` | 指向该别名要用的密钥文件,决定以哪个账号认证;用绝对路径(如 `/home/rookie/.ssh/id_ed25519_github2`)更保险 | 指向对应账号的密钥 |

原理:git 地址写 `git@github2:用户名/仓库.git` 时,SSH 按 `Host` 别名 `github2` 匹配这一段落,实际仍连接 github.com,但使用该段落指定的密钥认证。

补充说明:

- 别名的**三处一致**:`~/.ssh/config` 里的 `Host`、仓库 remote 地址里的主机部分、`ssh -T git@别名` 测试,必须完全一致;
- 不配别名的连接走默认规则:主机名 `github.com` 没有匹配段落时,SSH 尝试默认密钥 `~/.ssh/id_ed25519`,所以"账号一"也可以不显式配置;
- 文件权限:若 ssh 报 `Bad owner or permissions on ~/.ssh/config`,执行 `chmod 600 ~/.ssh/config`。

### 3.6 验证:两条 ssh -T 各归各位

```bash
ssh -T git@github.com     # 预期:Hi 账号一登录名!
ssh -T git@github2        # 预期:Hi 账号二登录名!
```

`Hi` 后面的名字就是该密钥对应的**真实 GitHub 登录名**,以此为准。两条都返回各自账号的名字,路由就配置成功了。

第二条测试的排错:

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 输出的还是账号一的名字 | `Host github2` 段落没生效,或密钥文件路径写错 | 检查 `IdentityFile` 路径与 `ssh-keygen -f` 生成的文件名是否一致 |
| `Permission denied (publickey)` | 新公钥没添加到新账号(或加错了账号) | 登录**新账号**打开 settings/keys,确认 `id_ed25519_github2.pub` 内容已添加 |
| `Could not resolve hostname github2` | config 里没有该 Host 段落 | 检查 `~/.ssh/config` 第二个段落是否保存完整 |

### 3.7 仓库级身份配置(user.name / user.email)

进入仓库目录,**去掉 `--global`**,只对该仓库生效:

```bash
cd ~/你的仓库目录
git config user.name "账号二的登录名"
git config user.email "账号二的邮箱"
git config --list    # 验证:仓库级值覆盖全局值
```

进阶:想让某目录下所有仓库自动使用账号二身份,在 `~/.gitconfig` 末尾加条件包含:

```ini
[includeIf "gitdir:~/你的仓库目录/"]
    path = ~/.gitconfig-account2
```

再新建 `~/.gitconfig-account2`,写入账号二的 `user.name` / `user.email`,进入对应目录自动生效。

### 3.8 仓库操作:用别名克隆 / 切换远程地址

```bash
# 新克隆:主机部分用别名,冒号后写真实登录名
git clone git@github2:账号二登录名/仓库名.git

# 已克隆的仓库:切换远程地址
cd ~/已有仓库目录
git remote set-url origin git@github2:账号二登录名/仓库名.git
git remote -v    # 验证
```

> 关键:地址中**冒号前是别名**(对应 ssh config 的 `Host`),**冒号后是真实登录名**(对应 GitHub 账号)。两者可以不同——例如别名叫 `username02`,登录名实际是 `username02`,地址就是 `git@username02:RookieFlash/WiFiDevelopmentNotes.git`。

### 3.9 HTTPS 方式的多账号(备选)

HTTPS 认证与 URL 中的用户名绑定,切换账号直接改远程地址,git 会按不同 URL 分别记住凭证:

```bash
git remote set-url origin https://账号二登录名@github.com/账号二登录名/仓库名.git
```

### 3.10 提交失败排查(决策流程)

**第一步:读报错,定位问题**

| 报错 | 含义 | 排查方向 |
| --- | --- | --- |
| `Permission to 仓库.git denied to Y` | 本次认证的账号 Y 没有该仓库写权限(仓库存在且公开,但 Y 不是所属账号) | 密钥走错了账号,或仓库归属搞错 |
| `Repository not found` | 仓库不存在,或为私有且该账号无权限 | 核实仓库路径与归属账号 |
| `Could not resolve hostname 别名` | 远程地址里的别名在 config 中没有对应 Host 段 | 检查 ~/.ssh/config |
| `Permission denied (publickey)` | 密钥没被 GitHub 识别(未添加/加错账号/路径错) | 检查公钥添加与 IdentityFile |
| `Bad owner or permissions on ~/.ssh/config` | 文件权限过宽 | `chmod 600 ~/.ssh/config` |

**第二步:三个检查命令**

```bash
git remote -v                 # 主机部分是哪个别名?冒号后是哪个账号名?
ssh -T git@github.com         # 默认密钥对应哪个账号?
ssh -T git@别名                # 新账号密钥对应哪个账号?
```

**第三步:按原则修复**。核心原则:**推送用的密钥必须属于仓库所属的账号**。仓库在哪个账号名下,remote 地址的主机部分就用那个账号的别名,且 `ssh -T git@该别名` 必须输出该账号的登录名。

**实战案例回顾**:push 报 `Permission to RookieFlash/WiFiDevelopmentNotes.git denied to username01`,排查结论:(1) remote 地址主机部分是 `github.com`,走了第一把密钥(username01),而仓库属于 `RookieFlash`;(2) 真实登录名不是以为的 `username02_022`,而是 `ssh -T` 输出的 `RookieFlash`。修复只需一条命令:

```bash
git remote set-url origin git@username02:RookieFlash/WiFiDevelopmentNotes.git
git push -u origin main
```

---

## 第四章 在 GitHub 创建仓库(一次性)

### 4.1 创建步骤

1. 登录 GitHub,点击右上角 **+** 号 → **New repository**
2. **Repository name** 填写 `tech-notes`(或自定,见 4.2 命名规则)
3. **Description** 选填,如"我的技术笔记合集"
4. 可见性选择 **Public**(公开)
5. 勾选 **Add a README file**
6. 点击 **Create repository**

### 4.2 各项设置说明

- **仓库命名规则**:只能包含英文字母、数字、连字符 `-`;不区分大小写但会保留书写形式;
- **可见性**:
  - Public:任何人可查看,免费版 GitHub Pages 只能用于公开仓库;
  - Private:仅自己(和邀请的人)可查看,更私密,但免费账号不能开 Pages;
- **Add a README file**:勾选后创建时会自动生成 `README.md`,推荐勾选(省去手动创建初始提交);
- **.gitignore / License**:对文章仓库不是必需的,忽略即可。

---

## 第五章 克隆仓库到本地(一次性)

### 5.1 克隆命令

```bash
cd ~
git clone git@github.com:你的GitHub用户名/tech-notes.git
cd tech-notes
```

- 地址必须与网页上建的仓库名**完全一致**(含大小写),仓库页面上有绿色的 Code 按钮,可以直接复制 SSH 地址;
- 多账号用户:把地址中的 `github.com` 换成第三章配置的别名,如 `git@github2:你的用户名/tech-notes.git`,确保用新账号的密钥连接;
- 用 HTTPS 方式的,复制 HTTPS 地址;
- 克隆完成后,`~/tech-notes` 就是一个普通的本地文件夹,仓库的全部内容都在里面。

### 5.2 本地目录结构规划

建议采用如下结构(层次一阶段,文章先平铺在根目录):

```text
~/tech-notes/
├── .git/                  # git 内部数据,不要动
├── README.md              # 仓库说明,可写仓库简介和文章索引
└── 文章一.md
```

升级到博客站后(第九章),文章移入 `docs/` 子目录:

```text
~/tech-notes/
├── docs/
│   ├── index.md           # 网站首页
│   └── 各篇文章.md
└── mkdocs.yml             # 网站配置
```

---

## 第六章 文章的准备与组织

### 6.1 统一使用 Markdown 格式

GitHub 对 Markdown(`.md` 文件)的渲染支持最好,建议所有文章统一为 Markdown 格式。它比 Word 更适合技术文章:纯文本、代码块友好、天然支持版本对比。

### 6.2 写作工具推荐

| 工具 | 费用 | 特点 |
| --- | --- | --- |
| Obsidian | 免费 | 本地 Markdown 笔记软件,所见即所得,自带文件管理器,适合写作 |
| VS Code | 免费 | 安装 Markdown Preview Enhanced 插件后即可边写边预览,程序员首选 |
| Typora | 一次买断 | 最顺滑的所见即所得 Markdown 编辑器 |

### 6.3 文件名与目录命名规范

- **文件名用英文小写加连字符**,如 `vmware-wifi-driver.md`、`git-common-commands.md`;
- **不要用中文文件名**:GitHub 网页上中文文件名在 URL 中会变成一串 `%E4%B8%AD%E6%96%87` 转义码,链接又长又难分享;
- **文章标题写在文件内部**:文件第一行用一级标题写中文标题,如 `# VMware USB 无线网卡驱动安装笔记`,网页上显示的就是中文标题;
- 文章较多时可用子目录分类,如 `linux/`、`network/`、`vmware/`。

### 6.4 文章中的图片处理

- 图片放进仓库内,文章用**相对路径**引用,网页上即可正常显示:

```markdown
![拓扑图](images/network-topology.png)
```

- 对应目录结构:

```text
~/tech-notes/
├── images/
│   └── network-topology.png
└── vmware-wifi-driver.md
```

- 升级博客站后,图片目录放在 `docs/images/`,引用写法不变;
- 单个文件不要超过 50 MB(GitHub 警告线),100 MB 会被拒绝推送;截图建议压缩后再放。

### 6.5 把已有文章转为 Markdown

- **Word 文档**:用 pandoc 一键转换

```bash
sudo apt install pandoc
pandoc 文档.docx -o 文档.md --extract-media=images
# --extract-media 会把文档内图片提取到 images/ 目录,相对路径自动生成
```

- **语雀/Notion 等**:各平台自带"导出为 Markdown"功能;
- **纯文本/网页复制的内容**:直接用 6.2 的编辑器新建 `.md` 文件粘贴重排即可。

---

## 第七章 提交并推送到 GitHub

### 7.1 首次推送完整流程

```bash
cd ~/tech-notes
cp ~/文档/vmware-wifi-driver.md ~/tech-notes/   # 示例:复制一篇文章进仓库
git status                 # 查看当前变更,确认文件已出现
git add .                  # 把所有变更加入暂存区
git commit -m "添加 VMware 无线网卡驱动笔记"      # 提交,生成一个版本
git push                   # 推送到 GitHub 远程仓库
```

### 7.2 三个核心命令的职责

| 命令 | 作用 | 类比 |
| --- | --- | --- |
| `git add .` | 把文件加入暂存区(标记"这次要提交它") | 把货物搬上打包台 |
| `git commit -m "说明"` | 在本地生成一个版本快照 | 打包封箱、贴标签 |
| `git push` | 把本地版本同步到 GitHub | 把箱子发往仓库 |

`git add` 和 `git commit` 只是本地操作,`git push` 之后网页上才看得到变化。

### 7.3 提交信息写法建议

- 用一句话说明这次做了什么,中文即可;
- 常用前缀约定:`新增文章:xxx`、`更新文章:xxx`、`删除文章:xxx`、`修正:xxx`。

---

## 第八章 在网页和客户端查看

### 8.1 网页查看(主要方式)

打开 `https://github.com/你的用户名/tech-notes`,点击文章文件即可看到渲染后的效果。每个文件右上角还有:

- **Raw**:查看原始文本
- **Blame**:逐行查看修改历史
- **History**(文件历史):查看该文件的全部修改记录

### 8.2 手机客户端

应用商店搜索 **GitHub** 安装官方 App,登录后可浏览仓库和文章渲染效果,但阅读体验不如网页。

### 8.3 桌面客户端

- Linux:没有官方桌面客户端,直接用网页;
- Windows/macOS:可安装 GitHub Desktop,图形化完成提交推送,无需记忆命令。

---

## 第九章 进阶:开启 GitHub Pages 博客站

### 9.1 什么是 GitHub Pages

GitHub 提供的免费静态网站托管服务,开启后仓库内容会自动构建成网站,获得正式网址:

```text
https://你的用户名.github.io/tech-notes/
```

每次推送文章,网站自动更新,无需服务器、无需备案(github.io 域名)。

**前提条件:免费账号的 Pages 只支持公开仓库**,若仓库当前是 Private,需先在仓库 Settings → General → Danger Zone 中改为 Public。

### 9.2 方案选型对比

| 方案 | 语言依赖 | 难度 | 特点 |
| --- | --- | --- | --- |
| **MkDocs + Material(推荐)** | Python | 低 | 技术文档界最流行;自动生成侧边导航、全文搜索、代码高亮,最像专业文档站 |
| Jekyll | Ruby | 中 | GitHub 官方默认方案,模板多但导航需自己配置 |
| Hugo | 单文件二进制 | 中 | 构建极快,中文博客圈流行 |

下文按 MkDocs + Material 展开。

### 9.3 MkDocs Material 完整部署

#### 9.3.1 安装工具

```bash
sudo apt install python3-pip
pip3 install mkdocs mkdocs-material
```

#### 9.3.2 调整目录结构

```bash
cd ~/tech-notes
mkdir docs
mv *.md docs/          # 把根目录所有文章移入 docs/
# 若之前用了 images/ 目录,一并移动:mv images docs/
```

#### 9.3.3 编写网站配置 mkdocs.yml

在仓库**根目录**创建 `mkdocs.yml`(用 `nano mkdocs.yml` 或编辑器均可):

```yaml
site_name: 我的技术笔记
site_description: 个人技术文章合集
theme:
  name: material
  language: zh
```

> 这是最小配置,网站即可正常工作。左侧导航会自动按 `docs/` 目录结构生成,文章标题自动取每篇文件的第一个一级标题。

#### 9.3.4 编写首页 docs/index.md

```markdown
# 欢迎

这里是我的技术笔记合集,左侧列表可以浏览所有文章。
```

#### 9.3.5 配置 GitHub Actions 自动构建

创建 `.github/workflows/publish.yml`(目录需逐级新建):

```bash
mkdir -p .github/workflows
nano .github/workflows/publish.yml
```

文件内容:

```yaml
name: publish
on:
  push:
    branches:
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

> 注意:`branches` 写 `main`(2020 年后新建仓库默认分支就是 main)。若你的仓库用的是 `master` 分支,请改为 `master`。

#### 9.3.6 推送并在网页端开启 Pages

```bash
git add .
git commit -m "启用 MkDocs 博客"
git push
```

然后:

1. 打开仓库网页 → **Settings** → 左侧 **Pages**
2. **Source** 选择 **Deploy from a branch**
3. **Branch** 选择 **gh-pages**,目录选 **/ (root)**,点击 **Save**
4. 回到仓库 **Actions** 标签页,可看到名为 publish 的构建任务在运行
5. 构建完成后(首次约 1~2 分钟),访问 `https://你的用户名.github.io/tech-notes/`

之后每次 `git push`,Actions 会自动重新构建,网站自动更新,无需任何人工操作。

#### 9.3.7 验证网站

- 首页正常显示,左侧出现文章导航列表;
- 点开任意文章,排版、代码块、图片正常;
- 顶部搜索框可全文搜索(需要 GitHub 构建完成后才生效,本地 `mkdocs serve` 也支持)。

### 9.4 本地预览

推送前想先看效果:

```bash
cd ~/tech-notes
mkdocs serve
# 浏览器打开 http://127.0.0.1:8000
# 修改文件后保存,页面自动刷新;Ctrl+C 退出
```

### 9.5 后续可选优化

**固定文章顺序**:文件名加数字前缀,如 `01-环境搭建.md`、`02-网络配置.md`(网页导航不显示前缀)。

**开启更多功能**:在 `mkdocs.yml` 的 `theme` 下添加:

```yaml
  features:
    - navigation.top          # 滚动时显示"回到顶部"
    - content.code.copy       # 代码块一键复制按钮
    - search.suggest          # 搜索自动补全
```

**自定义域名**:Settings → Pages → Custom domain 填入自有域名,再按提示配置 DNS;注意绑定国内服务器域名需 ICP 备案,`github.io` 免备案。

---

## 第十章 日常维护流程

### 10.1 新增文章

```bash
cd ~/tech-notes
cp ~/新文章.md docs/        # 未开博客站则直接放仓库根目录
git add .
git commit -m "新增文章:xxx"
git push
# 等待约半分钟,网页自动更新
```

### 10.2 修改文章

```bash
cd ~/tech-notes
# 用编辑器直接修改 docs/ 下的文件,保存
git add .
git commit -m "更新文章:xxx"
git push
```

### 10.3 删除/重命名文章

```bash
git rm docs/旧文章.md                 # 删除
git mv docs/旧名.md docs/新名.md      # 重命名
git commit -m "删除文章:xxx"          # 或 "重命名文章"
git push
```

> 历史版本都保留在 git 中,误删可通过 GitHub 网页上的 History 找回。

### 10.4 多设备同步

换一台电脑时:

1. 重复第二章(单账号)或第三章(多账号)的 SSH 密钥配置(用新设备生成新密钥并添加到 GitHub);
2. `git clone` 即可拿到全部文章;
3. 在新设备改动并 `git push` 后,旧设备先 `git pull` 再编辑,避免冲突。

---

## 第十一章 常见问题 FAQ

**Q1:push 时报 `Permission denied (publickey)`?**
SSH 密钥未配置成功,按 2.3 节重查:公钥是否粘贴完整、测试命令是否返回 `Hi 用户名`。

**Q2:网页上为什么看不到刚推送的内容?**
先强制刷新(Ctrl+F5)。博客站的话,检查仓库 Actions 标签页的构建是否成功;构建需要几十秒,失败会有红色叉号,点进去看日志。

**Q3:私有仓库能开 Pages 吗?**
免费账号不行,Pages 需要仓库公开;付费的 GitHub Pro 支持私有仓库 Pages。

**Q4:能不能用中文文件名?**
GitHub 支持显示中文文件名,但分享链接会变成转义码,建议按 6.3 节用英文文件名。

**Q5:提交记录里名字/邮箱不对?**
单账号场景执行 `git config --global user.name "..."` 和 `git config --global user.email "..."` 重新设置;多账号场景用仓库级配置(见第三章 3.7 节)。注意修改只影响之后的提交,历史提交的署名不会变。

**Q6:push 提示文件超过 100 MB?**
GitHub 硬性限制单文件 100 MB。压缩图片,或把大文件移出仓库改用网盘链接引用。

**Q7:clone 或 push 很慢/超时?**
多为网络原因,重试几次,或参考 2.3.4 改用 443 端口;SSH 不通时换 2.4 节的 HTTPS + PAT 方式。

**Q8:git 常用命令记不住怎么办?**
见附录 A 速查表;日常只需 `add`、`commit`、`push` 三条。

**Q9:push 报 `Permission to 用户名/仓库名.git denied to 另一个名字`?**
含义:本次推送认证的账号(`denied to` 后面的名字)没有该仓库的写权限,最常见原因是推送走错了密钥。完整排查流程见第三章 3.10 节,核心原则:**推送用的密钥必须属于仓库所属的账号**。

---

## 附录 A:Git 常用命令速查

| 命令 | 作用 |
| --- | --- |
| `git status` | 查看当前变更状态 |
| `git add .` / `git add 文件名` | 加入暂存区 |
| `git commit -m "说明"` | 提交版本 |
| `git push` | 推送到 GitHub |
| `git pull` | 拉取远端最新内容(多设备协作前必做) |
| `git log --oneline` | 查看提交历史 |
| `git diff` | 查看未提交的具体改动 |
| `git rm 文件` / `git mv 旧 新` | 删除 / 重命名并纳入版本管理 |
| `git restore 文件` | 撤销工作区改动(回到上次提交状态) |
| `git clone 地址` | 克隆仓库到本地 |
| `git remote -v` | 查看远程仓库地址 |
| `git remote set-url origin 新地址` | 修改远程仓库地址(多账号切换常用) |

---

## 附录 B:Markdown 语法速查

```markdown
# 一级标题(文章标题用它)
## 二级标题
### 三级标题

**粗体**  *斜体*  `行内代码`

- 无序列表项
1. 有序列表项

> 引用块

[链接文字](https://example.com)
![图片描述](images/xxx.png)

```代码块:语言标注后可高亮```

| 表格 | 第二列 |
| --- | --- |
| 单元格 | 单元格 |

---      ← 分割线
```

代码块写法示例(首尾各三个反引号,首个后面写语言名):

````text
```bash
sudo apt install git
```
````

---

## 附录 C:其他操作系统说明

| 项目 | Windows | macOS |
| --- | --- | --- |
| 安装 git | <https://git-scm.com/downloads> 下载安装包,一路默认 | `brew install git` |
| 终端 | 开始菜单 → Git Bash(后续命令相同) | 系统终端 |
| SSH 密钥 | 生成后 `cat ~/.ssh/id_ed25519.pub`,命令与 Linux 一致 | 与 Linux 完全一致 |
| 图形客户端 | GitHub Desktop 官方支持 | GitHub Desktop 官方支持 |

除安装方式外,本文所有步骤在三个平台上完全通用。

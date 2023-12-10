---
layout: post
title: "利用 git-crypt 实现 github 文件的自动加解密"
date: 2023-12-10 22:00:00 +0800
tags: Git
---

![metting](/assets/images/2023-12-10-github_crypt_note_1.png)

- 利用 github 保存代码或文档时，有些内容(如密码)想以加密形式保存
- 即便是 private 仓库，万一账号被盗或在 github 内部被泄露，都可能导致灾难性后果
- 希望实现在本地明文、在 github 上是加密状态，万一被人获得，只要没有密码就是安全的

## 方案原理

- 利用 git-crypt 工具，它可以在 git 仓库中添加加密文件
- 在 git 提交时加密，在 git 拉取时解密，这样就实现了在本地明文、在 github 上加密的效果
- 自动加解密是基于 git 的`gitattribute`工作机制，每次提交和拉取时，都会自动执行 git-crypt 命令
- 在外部文件夹或网络保存秘钥，可以在本地执行`git crypt lock`和`git crypt unlock /key/path`加密解密当前 repo，避免本机内容泄露

## 初次安装

- mac 安装`brew install git-crypt`
- 然后在仓库根目录执行`git-crypt init`

  - 执行后生成一个秘钥：`.git/git-crypt/keys/default`
  - 在`.git/config`中加入相关配置，利用 gitattribute 属性实现自动加解密

    ```conf
    [filter "git-crypt"]
    smudge = \"/usr/local/bin/git-crypt\" smudge
    clean = \"/usr/local/bin/git-crypt\" clean
    required = true
    [diff "git-crypt"]
    textconv = \"/usr/local/bin/git-crypt\" diff
    ```

- 将秘钥导出保存到外部文件夹`git-crypt export-key /path/to/key`以便未来解密使用，原目录中的可能被删除
- 添加`.gitattributes`配置文件，与`.gitignore`类似，添加加密文件范围

  ```conf
  # avoid accidentally encrypt files
  # NOTES: need to be at last to have highest priority
  .gitignore !filter !diff
  .gitattributes !filter !diff

  # encrypt all files in diary directory
  diary/** filter=git-crypt diff=git-crypt
  ```

- 然后在 diary 文件夹下添加一个新文件，如`diary.txt`，然后`git commit -Am "add diary"`，`git push`
- 这时查看 github 的文件，显示为加密后的乱码；而本地可以正常的查看编辑
- `git crypt lock` 加密 repo，删除 default key。之后将看到和 github 一样的加密效果，本地也无法查看编辑。git clone 下来也是这样的状态。
- `git crypt unlock /path/to/key` 解密 repo，导入新的 key 为 default key

## 常用命令

- `git crypt init`
  初始化加密配置及秘钥

- `git-crypt export-key /path/to/key`
  导出秘钥到文件

- `git crypt status`
  查看被加密的文件范围

  - `-e`
    排除未被加密的文件

- `git crypt lock`
  锁定本地 repo，删除 default key

- `git crypt unlock /path/to/key`
  解锁本地 repo，导入新的 key

## 其它

- 可以利用网络密码机制保存秘钥，这样就更安全了

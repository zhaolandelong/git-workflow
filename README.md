# git-workflow

## 介绍
参考 Gitflow、GitHub Flow、GitLab Flow 实现的一套 Git Workflow 规范，并用脚本实现了可交互半自动化。目前只支持用 GitHub 做代码管理的项目（因为有命令行工具 [gh](https://cli.github.com/)）。可实现的功能有：
- 自动创建符合规范的分支；
- 自动创建 PR，并提取 commit list 作为 body 内容，让每个 PR 都具有的优雅的可读性；
- PR 合并之后，自动删除本地和远程的分支，并自动做功能分支的同步；
- Hotfix、Deploy 合并后，会自动触发：**version 的更新、打相应版本的 tag、生成 CHANGELOG、发布一个 RELEASE、同步功能分支代码**；

更详细的信息见：[Gitflow 太繁琐？为什么不自动化呢](https://juejin.cn/post/7056410651563917326)。

## 准备

1. 安装 [gh](https://cli.github.com/)，安装方法见官网；
2. 安装 [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) 并初始化，命令如下：
```bash
# 1. 安装 conventional-changelog-cli
npm i -D conventional-changelog-cli

# 2. 在 .npmrc 中增加配置，统一 tag 的表现
echo -e "\ntag-version-prefix=\"\"\nmessage=\"chore(release): %s :tada:\"" >> .npmrc

# 3. 将 CHANGELOG.md 加入到 .gitignore 中，防止多人提交产生冲突
echo -e "\n# Git workflow log\nCHANGELOG.md" >> .gitignore

# 4. 在 package.json 中增加 version 命令，触发 npm version 的 hook
#  "scripts": {
#    "version": "conventional-changelog -p angular -o CHANGELOG.md"
#  },

# 5. 复制 gitflow 到本地目录，并修改其权限
curl https://raw.githubusercontent.com/zhaolandelong/git-workflow/main/gitflow > gitflow && chmod +x ./gitflow
```

3. 切好 master、release、develop 3 个功能分支，并推到远端。名字可以变，只要功能对应上即可，**但要记得手动修改脚本中的变量**；

4. 修改 package.json 的 version 至合理版本，将以上所有变化提交 commit。即 `git add . && git commit -m 'chore: gitflow init'`
5. 打上相应版本 tag，把当前变化与 tag 都同步到远端。即 `git tag x.x.x && git push && git push --tags`

## 用法
运行 `./gitflow` 即可，会有可交互的提示。举例：

### Feature Start
假如想开始一个 feature，运行 `./gitflow` 会看到
```
What's the type? (Notice: Admin Permission needed with UAT and DEPLOY)
1) feature
2) bugfix
3) hotfix
4) UAT
5) DEPLOY
#?
```
输入 1，会看到
```
#? 1
- Type: feature
What's the name (description for short. eg: btn-display-error)?
```
输入 changelog-init，会看到
```
changelog-init
- Name: changelog-init
What's the method?
- start: Start a work and create a new branch
- submit: Submit a PR
- finish: Merge the PR after CR and delete branch
1) start
2) submit
3) finish
#?
```
输入 1，会看到
```
#? 1
- Method: start
Already on 'develop'
Your branch is ahead of 'origin/develop' by 1 commit.
  (use "git push" to publish your local commits)
Current branch develop is up to date.
Switched to a new branch 'feature/changelog-init'
```
已经切好分支。

### Feature Submit
功能开发完成，提交了若干 commit 后，在当前分支运行 `./gitflow`，会看到
```
Do you want to use current params?
- Type: feature
- Name: changelog-init
1) yes
2) no
```
脚本做了交互优化处理，会根据当前分支名自动判断是否还需要再次输入参数，如果选 2，会进入普通步骤。我们这里选 1，会看到
```
#? 1
What's the method?
- start: Start a work and create a new branch
- submit: Submit a PR
- finish: Merge the PR after CR and delete branch
1) start
2) submit
3) finish
```
直接进入选择 method 的选项，这时选 submit，会自动生成 PR，并填充 body。

### Feature Finish
运行步骤没有什么特别，但是要注意，使用此命令的人是否有对应 target 分支的合并权限，具体请见 [Gitflow 太繁琐？为什么不自动化呢](https://juejin.cn/post/7056410651563917326) 中的表格。

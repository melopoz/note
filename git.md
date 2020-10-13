fetch 抓取远程仓库内容

pull 拉取代码到本地

merge 合并

rebase 不分叉合并



##### 初始化成git仓库

`git init`

##### 绑定到远程仓库

`git remote add origin https://xxx.git`



---

用` reset `回滚，参数：

> git reset --soft						  保留工作目录，并把新的全部差异放进暂存区
>
> > 原节点和Reset节点之间的所有差异都会放到暂存区中
>
> ​			原节点和**Reset**节点之间的所有差异都会放到暂存区中
>
> git reset (git reset --mixed)	保留工作目录，清空暂存区
>
> git reset --hard						 清空工作目录，清空暂存区，全部和git log 中的上n次commit内容相同

>  soft 和 mixed 区别应该就是 mixed 不需要再手动 add 而 soft 需要自己再手动 add。



https://www.jianshu.com/p/c2ec5f06cf1a



## 修改commit的msg
`git commit --amend` 
会进入vim，修改保存就行了
### 如果commit已经被覆盖
`git rebase -i HEAD~n` n是逆推几次
```
pick 07bca843 添加 广告线索来源构成分析、概览 接口 /source-ad/constitute、/source-ad/list
pick 67d1e638 移除/income/top
```
将开头的pick修改为edit就可以编辑了
然后git会提示
```
You can amend the commit now, with
  git commit --amend
Once you are satisfied with your changes, run
  git rebase --continue
```
`git commit --amend` 进入修改
完事continue rebase 即可
`git rebase --continue`
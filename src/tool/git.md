#### [常用git命令](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)



#### 使用Git Submodule管理子模块
子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立

##### 添加子模块
`git submodule add [url] [name]`

##### 克隆含有子模块的仓库
1. `git clone [url]`
2. `git submodule init`
3. `git submodule update`

或者
1. `git clone --recursive [url]`

##### 在子模块项目中修改提交同步父项目
1. 子模块中：`cd [子模块]`
2. git pull
3. 回到父模块：`cd ../`
4. `git add [子模块]`
5. git commit -m ''
6. git push

或者
1. `git submodule foreach git pull`
2. 同上述4 5 6

父项目中不会包含子模块中的提交信息，但每次上述操作之后会有一个新的提交记录，变更子模块当前提交ID
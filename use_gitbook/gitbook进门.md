# gitbook 进门指导

## 环境配置

1. nodejs 因为 GitBook 是基于 Node.js，所以我们首先需要安装 Node.js（[下载地址](https://nodejs.org/en/download/)，找到对应平台的版本安装即可。
2. gitbook 现在安装 Node.js 都会默认安装 npm（node 包管理工具），所以我们不用单独安装 npm，打开命令行，执行以下命令安装 GitBook：

   ```js
   npm install -g gitbook-cli
   ```

   安装完之后，就会多了一个 gitbook 命令（如果没有，请确认上面的命令是否加了 -g）。

3. VSCode，这是书写 markdown 文件的工具，无所不能的编辑器。安装插件`MakkdownAll in One`。

## 使用步骤

1. 在文件系统中新建任意名称的目录，这是后面需要放置 gitbook 的位置。我的路径为`D:\05-gitbook\TIL`
2. 使用 VSCode 打开这个目录，现在还是空的，打开 VSCode 内置终端，输入`git init`，gitbook 会生成两个空 md 文件，`SUMMARY.md`和`README.md`。SUMMARY 是正本书的目录，而 README 则说明了这本书的简介。当然，这些需要人工补充。
3. 在`SUMMARY.md`中添加如下文字，当然了，这只是我目前已有的 md 文件，是为了后面的演示添加的：

   ```markdown
   # TIL_SUMMARY

   - [目录](SUMMARY.md)

   - [Introduction](README.md)

   - [use_gitbook](use_gitbook/01_summary.md)
   - [gitbook 进门](use_gitbook/gitbook进门.md)
   - [cpp](cpp/01_summary.md)
   - [C++template 入门一](cpp/C++template入门一.md)
   - [C++template 入门二](cpp/C++template入门二.md)
   - [现代 C++中的 weak_ptr](cpp/现代C++中的weak_ptr.md)
   - [reading](readings/01_summary.md)
   - [书解《图形思考与表达的 20 堂课》](readings/书解《图形思考与表达的20堂课》.md)
   - [writing](writings/01_summary.md)
   - [开启新的记录](writings/使用Typora写文档.md)
   - [使用这个输入法](writings/使用这个输入法.md)
   ```

4. 现在有 4 个子目录，里面有各自的文档
5. 执行`gitbook build`生成`_book`目录，里面会生成相应的 html 文件
6. 开启 gitbook 服务，执行`gitbook serve`.执行结果如果是

   ```cmd
    Starting server ...
    Serving book on http://localhost:4000
   ```

   则说明服务开启成功！

7. 在浏览器查看我们的 book，输入`http://localhost:4000`即可。
8. 将本地的 gitbook 推送到 github 上
   - 在 github 上新建一个仓库，我的是`https://github.com/lcksfa/TIL.git`
   - 在本地的 gitbook 目录`D:\05-gitbook\TIL`下打开`powershell`
   - 输入`git remote add origin https://github.com/lcksfa/TIL.git`，将本地的目录与远端关联
   - 输入 `git push -u origin master`即可将本地 git 管理的文档推送到 github
   - 下次本地更新了文档，先在本地使用`git commit` ，再直接使用`git push`命令即可推送。
9. gitbook 插件
   修改了`book.json`文件，

   ```json
   {
     "title": "Today I Learned",
     "author": "Lcksfa",
     "plugins": ["page-treeview", "summary3"],
     "pluginsConfig": {
       "page-treeview": {
         "copyright": "Copyright &#169; aleen42",
         "minHeaderCount": "2",
         "minHeaderDeep": "2"
       }
     }
   }
   ```

   使用`gitbook install`安装配置的插件即可，现在，我们的 summary 文件可以有插件`summary3`自由生成了；
   这里，我还另外添加了一个 treeview 插件，用于在文章的片头生成大概。

10. 以后会保持一日一提交的频率。

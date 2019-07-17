## hexo搭建博客

#### 1.安装nodejs
#### 2.npm install -g hexo
#### 3.hexo init <path>
#### 4.npm install hexo-renderer-less --save
#### 5.npm install hexo-generator-feed --save
#### 6.npm install hexo-generator-json-content --save
#### 7.npm install hexo-helper-qrcode --save
#### 8.hexo g
#### 9.hexo server -d

##### p.s. 
> * 若提示WARN  No layout: index.html 应为主题丢失，克隆git clone git@github.com:yscoder/hexo-theme-indigo.git themes/indigo可解决
> * 若hexo d时提示spawn failed，这个错误是因为本地的博客版本与远程的版本不一致，解决方法是删除博客目录下的.deploy_git文件夹

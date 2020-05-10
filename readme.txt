跟换环境后，继续博客：
一.在新的环境上配置好hexo环境
1.cnpm install -g hexo
2.hexo init
3.cnpm install hexo-deployer-git --save

二.检出仓库中的hexo分支代码，其中包含了之前历史所有的博客文档。在其基础上继续写

三.新建博客
1.hexo new post "testblog"
会自动在./source/_posts下生成日期格式的目录,并且有testblog.md文件生成
2.编辑testblog.md文件

四,发布博客
1.hexo clean
2.hexo g
3.hexo s
4.hexo deploy
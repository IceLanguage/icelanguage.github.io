---
layout: page
title: 报错记录Another git process seems to be running in this repository
    - blogs
---

这种报错是因为一个项目同时用2个不同的git管理工具或git.exe程序、

解决方法
删除文件/git/index.lock（也就是你项目git目录下的index.lock文件）
![这里写图片描述](http://img.blog.csdn.net/20171126124845662?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171126124856494?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果看不到git目录，请查看文件的详细信息
如下点击查看，再点击详细信息
![这里写图片描述](http://img.blog.csdn.net/20171126004740996?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

基本到上一步都能解决问题了
如果依旧失败，请重启后再尝试

若再度失败，请打开任务管理器把所有git相关进程全部都结束任务，再尝试commit
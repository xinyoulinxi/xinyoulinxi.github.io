
# [个人博客](https://xinyoulinxi.github.io/)
- 会记录一些计算机相关的东西，但不是教程
- 会记录生活，但不是日记

## 本地调试博客
```
bundle install # 下载插件等，通过GemFile
bundle add webrick # 用于解决require的错误堆栈
bundle exec jekyll serve # 启动本地调试，一般是在本地的4000端口，也就是http://127.0.0.1:4000/
```

```
# 发布博客
rake post title="Hello 2015" subtitle="Hello World, Hello Blog"
```
# Ideastudios模板

### [我的博客在这里 &rarr;](http://ideastudios.github.io)


### 关于收到"Page Build Warning"的email

由于jekyll升级到3.0.x,对原来的pygments代码高亮不再支持，现只支持一种-rouge，所以你需要在 `_config.yml`文件中修改`highlighter: rouge`.另外还需要在`_config.yml`文件中加上`gems: [jekyll-paginate]`.

同时,你需要更新你的本地jekyll环境.

使用`jekyll server`的同学需要这样：

1. `gem update jekyll` # 更新jekyll
2. `gem update github-pages` #更新依赖的包

使用`bundle exec jekyll server`的同学在更新jekyll后，需要输入`bundle update`来更新依赖的包.

参考文档：[using jekyll with pages](https://help.github.com/articles/using-jekyll-with-pages/) & [Upgrading from 2.x to 3.x](http://jekyllrb.com/docs/upgrading/2-to-3/)


## 关于模板

该模板基于Hux的博客进行的修改，[Hux的博客在这里 &rarr;](http://huxpro.github.io) 大家可以直接fork模板——`huxblog-boilerplate`,要改的地方作者都说明了，或者可以直接下载zip到本地自己去修改，感谢Hux，给他点32个赞！！

```
$ git clone git@github.com:Huxpro/huxblog-boilerplate.git
```







## 支持

* 如果你喜欢Hux的这个博客模板，请在`huxpro.github.io`这个repository点个赞——右上角**star**一下，当然了，我是已经点了赞了。

## 说明文档(该模板全部基于Hux的博客进行修改，[详细说明文档 &rarr;](https://github.com/Huxpro/huxpro.github.io/blob/master/README.zh.md))


## 致谢

1. 这个模板是从这里[Huxpro/huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)  fork 的。 感谢这个作者胡克斯
2. 感谢Hux感谢的人

3. 感谢 Jekyll、Github Pages 和 Bootstrap!




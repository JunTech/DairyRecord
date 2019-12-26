# hexo博客优化

## 1、图片优化

打开图片会消耗很多时间，所以保存位置及缩小尺寸尤为重要，下面介绍几种方法：

1.使用七牛云，存储图片，可获得cdn加速，博客瞬间飞起，

参考：<https://www.baidu.com/link?url=FWd23ZLs82IehhD2Bsp8LSYJQ_MTW3TcD7U1COUzxpO2p9bBWVmeS9UdkymB-JdG&wd=&eqid=c16257c8000cd75f000000045d8d8707>

2、白嫖的：自己也在使用的：

网址：<https://img.vim-cn.com/>，可解决图片不显示问题，及缩小服务器图片所占空间问题

3、图片懒加载

**思路**

在hexo中编写博客，一般以markdown或html的img标签来添加图片 ，只要在渲染之前将所有img标签替换成适合LazyLoad的格式就可以了

1

在你使用的主题文件夹找到 layout > _partial > footer.ejs 文件（或者 head.ejs也可以），在后面加入以下代码

```
//引用了jquery百度的 cdn 源，也可替换其他的
 <script type="text/javascript" src="http://libs.baidu.com/jquery/1.11.1/jquery.min.js"></script>
 //也可替换其他的lazyload源
<script type="text/javascript" src="http://apps.bdimg.com/libs/jquery-lazyload/1.9.5/jquery.lazyload.min.js"></script>
    <script type="text/javascript">
      $(function() {  
      //对所有 img 标签进行懒加载        
          $("img").lazyload({
          //设置占位图，我这里选用了一个 loading 的加载动画
            placeholder:"/img/loading.gif",
            //加载效果
              effect:"fadeIn"
            });
          });
    </script>
```

2

在主题文件夹下的scripts文件夹里，增加一个 js 文件，命名随意，我这里命名 lazyload-core.js,里面内容如下

```
'use strict';
var cheerio = require('cheerio'); 
  
function lazyloadImg(source) {
    var LZ= cheerio.load(source, {
        decodeEntities: false
    });
    //遍历所有 img 标签，添加data-original属性
    LZ('img').each(function(index, element) {
        var oldsrc = LZ(element).attr('src');
        if (oldsrc) {
            LZ(element).removeAttr('src');
            LZ(element).attr({
                
                 'data-original': oldsrc
            });
            
        }
    });
    return LZ.html();
}
//在渲染之前，更改 img 标签
hexo.extend.filter.register('after_render:html', lazyloadImg);
```

## 2、seo

> SEO（Search Engine Optimization）:汉译为搜索引擎优化。是一种方式:利用搜索引擎的规则提高网站在有关搜索引擎内的自然排名。目的是：为网站提供生态式的自我营销解决方案，让其在行业内占据领先地位，获得品牌收益；SEO包含站外SEO和站内SEO两方面；为了从搜索引擎中获得更多的免费流量，从网站结构、内容建设方案、用户互动传播、页面等角度进行合理规划，还会使搜索引擎中显示的网站相关信息对用户来说更具有吸引力 [=>百度百科](https://baike.baidu.com/item/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E%E4%BC%98%E5%8C%96/3132?fromtitle=seo&fromid=102990&fr=aladdin)

### 百度收录站点

登录[百度站长平台](http://zhanzhang.baidu.com/)，在用户中心 => 站点管理添加你的站点网址

![img](https://upload-images.jianshu.io/upload_images/3850436-42f832ac05001ce7?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Hexo-SEO设置01

配置完站点属性后，进入最后一步：验证网站。有三种方式：文件验证、HTML标签验证、CNAME验证，文件验证和CNAME验证都比较简单，也有相对应的帮助文本，在此我选择的是HTML标签验证。

![img](https://upload-images.jianshu.io/upload_images/3850436-2947491429808daf?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Hexo-SEO设置01

1. 在主题的_config.yml文件中，设置：baidu_site_verification: true，如果没有该字段就手动添加。
2. 在themes/next/layout/_partials/head.swig文件中添加下列代码

```
// 每个人的content值都不一致，请注意更换成你的content值

{% if theme.baidu_site_verification %}
  <meta name="baidu-site-verification" content="6K5YmdKWEx" />
{% endif %}
```

1. 配置好后，重新发布站点，在百度站长页面完成验证。

### 百度链接提交

> 链接提交工具是网站主动向百度搜索推送数据的工具，本工具可缩短爬虫发现网站链接时间，网站时效性内容建议使用链接提交工具，实时向搜索推送数据。本工具可加快爬虫抓取速度，无法解决网站内容是否收录问题

由于github屏蔽百度的爬虫，所以使用github page服务的站点的链接无法被抓取，可用[coding](https://coding.net/)的page服务。

#### 主动推送

> 最为快速的提交方式，建议您将站点当天新产出链接立即通过此方式推送给百度，以保证新链接可以及时被百度收录。

1. 安装百度链接提交插件

```
npm install hexo-baidu-url-submit --save 
```

1. ​

```
# 百度链接自动提交
baidu_url_submit:
  count: 6 # 提交最新的链接数量
  host: http://lianghuii.com # 在百度站长平台中注册的域名
  token:  # 请注意这是您的秘钥， 所以请不要把博客源代码发布在公众仓库里!
  path: baidu_urls.txt # 文本文档的地址， 新链接会保存在此文本文档里
```

1. 设置deploye

```
deploy:
  - type: git
    repo:
      github: git@github.com:MeanMouse/MeanMouse.github.io.git
      coding: git@git.coding.net:MeanMouse/blog.git
  - type: baidu_url_submitter
```

#### 自动推送

> 是轻量级链接提交组件，将自动推送的JS代码放置在站点每一个页面源代码中，当页面被访问时，页面链接会自动推送给百度，有利于新页面更快被百度发现。。

1. 在主题配置文件将baidu_push设置为true
2. 在路径themes\next\layout_scripts\下创建baidu_push.swig 文件，文件内容如下

```
{% if theme.baidu_push %}
<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';        
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>
{% endif %}
```

#### sitemap

> 您可以定期将网站链接放到Sitemap中，然后将Sitemap提交给百度。百度会周期性的抓取检查您提交的Sitemap，对其中的链接进行处理，但收录速度慢于主动推送。

使用npm自动生成网站的sitemap，然后将生成的sitemap提交到百度和其他搜索引擎

1. 安装sitemap插件

```
npm install hexo-generator-sitemap --save     
npm install hexo-generator-baidu-sitemap --save
```

1. 修改hexo配置文件

```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://lianghuii.com
root: /
permalink: :title/
permalink_defaults:
```

执行完之后就会在网站根目录生成sitemap.xml文件和baidusitemap.xml文件,其中sitemap.xml文件是搜索引擎通用的文件，baidusitemap.xml是百度专用的sitemap文件。

1. 将生成的sitemap文件提交到百度站长

![img](https://upload-images.jianshu.io/upload_images/3850436-47cc13d7d2825cc1?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Hexo-SEO设置03

#### 手动提交

> 如果您不想通过程序提交，那么可以采用此种方式，手动将链接提交给百度。



## 3、seo继续优化

### 文章页面配置

#### 缩短页面url路径

hexo 中文章页面url默认为：sitename/year/mounth/day/title，由于url层级过多爬虫不容易爬到文章，所以将页面url缩简为：sitename/title，在hexo 配置文件中修改permalink 配置项。

```
url: https://juntech.top/
root: /
# permalink: :year/:month/:day/:title/
permalink: posts/:abbrlink.html
permalink_defaults:

# permalink: :title/  将之前的注释掉permalink: archives/:abbrlink.html
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

#### 文章增加keywords、description等标识字段

在hexo 根目录中scaffolds文件中的文章模版文件中增加。

## 4、压缩静态文件

有2种方法：neat 和gulp

### neat:

### 安装

$ npm install hexo-neat --save

### 配置

```
neat_enable: true
# 压缩html
neat_html:
  enable: true
  exclude:
# 压缩css
neat_css:
  enable: true
  exclude:
  - '**/*.min.css'
  - '**/needsharebutton.css'
# 压缩js
neat_js:
  enable: true
  mangle: true
  output:
  compress:
  exclude:
  - '**/*.min.js'
  - '**/jquery.fancybox.pack.js'
  - '**/index.js'
  - '**/waifu-tips.js'
  - '**/iframe.js'
  - '**/fireworks.js'

```

### gulp

### 安装

package.json加入：

```
        "gulp": "^3.9.1",
        "gulp-concat": "^2.6.1",
        "gulp-htmlclean": "^2.7.22",
        "gulp-htmlmin": "^5.0.1",
        "gulp-imagemin": "^6.1.0",
        "gulp-minify-css": "^1.2.4",
        "gulp-uglify": "^3.0.2",
```

然后在cnpm install

### 配置：

gulp.js放在根目录下：

```
var gulp = require('gulp'),
    uglify = require('gulp-uglify'),
    cssmin = require('gulp-minify-css'),
    imagemin = require('gulp-imagemin'),
    htmlmin = require('gulp-htmlmin'),
    htmlclean = require('gulp-htmlclean');
concat = require('gulp-concat');
//JS压缩
gulp.task('uglify', function() {
    return gulp.src(['./public/js/**/.js', '!./public/js/**/*min.js']) //只是排除min.js文件还是不严谨，一般不会有问题，根据自己博客的修改我的修改为return gulp.src(['./public/**/*.js','!./public/zuoxi/**/*.js',,'!./public/radio/**/*.js'])
        .pipe(uglify())
        .pipe(gulp.dest('./public/js')); //对应修改为./public即可
});
//public-fancybox-js压缩
gulp.task('fancybox:js', function() {
    return gulp.src('./public/vendors/fancybox/source/jquery.fancybox.js')
        .pipe(uglify())
        .pipe(gulp.dest('./public/vendors/fancybox/source/'));
});
// 合并 JS
gulp.task('jsall', function() {
    return gulp.src('./public/**/*.js')
        // 压缩后重命名
        .pipe(concat('app.js'))
        .pipe(gulp.dest('./public'));
});
//public-fancybox-css压缩
gulp.task('fancybox:css', function() {
    return gulp.src('./public/vendors/fancybox/source/jquery.fancybox.css')
        .pipe(cssmin())
        .pipe(gulp.dest('./public/vendors/fancybox/source/'));
});
//CSS压缩
gulp.task('cssmin', function() {
    return gulp.src(['./public/css/main.css', '!./public/css/*min.css'])
        .pipe(cssmin())
        .pipe(gulp.dest('./public/css/'));
});
//图片压缩
gulp.task('images', function() {
    gulp.src('./public/images/*.*')
        .pipe(imagemin({
            progressive: false
        }))
        .pipe(gulp.dest('./public/images/'));
});
// 压缩 public 目录 html文件 public/**/*.hmtl 表示public下所有文件夹中html，包括当前目录
gulp.task('minify-html', function() {
    return gulp.src('./public/**/*.html')
        .pipe(htmlclean())
        .pipe(htmlmin({
            removeComments: true,
            minifyJS: true,
            minifyCSS: true,
            minifyURLs: true,
        }))
        .pipe(gulp.dest('./public'))
});
gulp.task('build', ['uglify', 'cssmin', 'fancybox:js', 'fancybox:css', 'jsall', 'images', 'minify-html']);

//, 'minify-html'
```

## 5、google广告

新的一年开始，由于个人的博客站点：https://juntech.top 已经建立几个月，一直安静地躺在那里做美男子。就想着接点小广告，赚一点睡后收入。于是搜索发现了Google AdSense ，发现它可以在hexo博客上挂上广告位进行展示，于是乎注册了一个账号，没想到今早通过了审核。
今天又弄了一下广告位的布局，总共commit了差不多10次左右，终于把广告位置排的比较合理了。

下面分享一下具体的执行步骤

注册AdSense账号
注册链接：AdSense
ps:需要 V屁N（vpn），请自行找梯子。。。

或者使用google helper: 地址：<https://github.com/haotian-wang/google-access-helper>

![](https://img-blog.csdnimg.cn/2019010212452549.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N0b3JtZG9ueQ==,size_16,color_FFFFFF,t_70)

点击“SIGN UP NOW”,进行注册。如果你有谷歌账号那就很方便了，直接点击右上角的“SIGN IN”就好。

详细的注册步骤就不多做介绍，注意填写信息的时候，一定要谨慎，否则可能审核通不过，这样大大的浪费了你的时间。

填写完信息之后，需要将谷歌提供的代码放置到你的博客中。请参考下一个步骤

添加广告代码
AdSense要求要在 <head> 标记中添加了自动广告代码,只需在\themes\next\layout\_partials\head.swig中添加任意一个位置添加你获取到的代码。

例如：我将AdSense的代码和google分析的代码放置在一块,这样渲染的时候就会自动放置在某一个位置了。

```
{% if theme.google_site_verification %}

  <meta name="google-site-verification" content="dYiRwj1ulGaUvTQRrCjJA9YKnNF8JN4wRKXzdbE6wBc" />

{% endif %}

<!-- 添加获取的代码 -->

<script async src="http://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>

<script>

(adsbygoogle = window.adsbygoogle || []).push({

google_ad_client: "ca-pub-123456789",

enable_page_level_ads: true

});
</script>
```

静候结果
添加了代码之后，如果顺利的话，google AdSense会发邮件到你注册的邮箱，在收到邮箱之后，登录AdSense,就可以而根据自己的博客站点选择相应的广告单元了。

![](https://img-blog.csdnimg.cn/20190102124613131.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N0b3JtZG9ueQ==,size_16,color_FFFFFF,t_70)


个性化配置博客
此处以 NexT 主题为例，介绍自定义配置的设置方式。

新建 `theme/next/layout/_custom/google_adsense.swig`，将 AdSense 上的代码粘贴进去
在 `theme/next/layout/_custom/head.swig` 中也粘贴一份
如果在每篇博客里也想看到广告的话，在 `theme/next/layout/post.swig` 里中在希望看到的地方加上:

```
{% include '_custom/google_adsense.swig' %}
```

例如：我的博客中将一个自定义的广告快放置到了留言板下面
在`\themes\next\layout\_partials\comments.swig`中将提供的代码放置进去

详情页面：<https://juntech.top/>

## 6、注意事项

**在成功接入AdSense广告之后，有以下几点需要记住的。而且Google也会根据几种方式和数据判断广告点击是否作弊，从而注销你的账号。所以不要心存侥幸心理，好好发原创文章，提高网站的质量才是王道。**

**作弊广告点击者的IP地址与你Adsense账户登录IP地址相同**
**作弊广告点击的CTR数据太高**
**作弊广告点击者的IP地址来自同一个地理区域**
**根据Cookies判断作弊Adsense广告点击**
**作弊广告点击者页面停留时间太短**
**直接访问者的广告点击率过高**
**流量小但广告点击率高**
**在网页上用文字提示请求鼓动点击广告**

## 7、美化主题

本博主使用的是：魔改next主题，省的瞎搞，浪费很多不必要的时间，效果如下：

https://juntech.top

如果喜欢的可以去给作者点个赞：传送门：


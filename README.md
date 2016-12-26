# SparkYuan.github.io

##添加“置顶”功能的方法
把 node_modules/hexo-generator-index/lib/generator.js 文件修改如下, 只需要在front-matter中设置需要置顶文章的top值，将会根据top值大小来选择置顶顺序。（大的在前面）

```js
'use strict';

var pagination = require('hexo-pagination');

module.exports = function(locals) {
  var config = this.config;
  var posts = locals.posts;

    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排

    });

  var paginationDir = config.pagination_dir || 'page';

  return pagination('', posts, {
    perPage: config.index_generator.per_page,
    layout: ['index', 'archive'],
    format: paginationDir + '/%d/',
    data: {
      __index: true
    }
  });
};
```

##Instagram

[同步你的instagram图片](https://github.com/litten/hexo-theme-yilia/wiki/%E5%90%8C%E6%AD%A5%E4%BD%A0%E7%9A%84instagram%E5%9B%BE%E7%89%87)

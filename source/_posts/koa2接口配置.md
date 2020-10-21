---
title: koa2接口配置
date: 2020-09-16 23:31:08
tags: ["koa2","接口配置"]
top_img: /img/koa2接口配置/1.PNG
cover: /img/koa2接口配置/1.PNG
categories: 配置教程
copyright: false
---

## 创建koa2项目
1. 安装koa-generator
```
npm install koa-generator -g
```
2. 使用koa-generator生成koa2项目
```
koa2 flea-market-koa //（koa2 “项目名称”）
```

3. 安装依赖包
```
cd flea-market-koa
npm install
```

4. 启动服务
```
npm start
```
打开浏览器：http://localhost:3000/会看到“Hello Koa 2!”就是启动成功了！
![](/img/koa2接口配置/1.PNG)

## 配置Sequelize操作数据库
1. 安装Sequelize
```
npm install sequelize --save 
//安装mysql，mysql2
npm install mysql mysql2 --save
```

2. 配置Sequelize
根目录下新建一个config文件，在config文件下新建一个db.js文件，这个文件就是用来建立连接mysql数据库的:
在db.js文件写入代码：
```
const Sequelize = require('sequelize');
const sequelize = new Sequelize('数据库名', '本地数据库用户名', '本地数据库密码', {
    host: 'localhost',
    dialect: 'mysql',
    operatorsAliases: false,
    dialectOptions: {
        // 字符集
        charset: "utf8mb4",
        collate: "utf8mb4_unicode_ci", 
        supportBigNumbers: true,
        bigNumberStrings: true
    },
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    },
    timezone: '+08:00' //东八时区
});
module.exports = {
    sequelize
}
```

3. 整理项目结构
根目录新增三个文件夹：
schema（数据表模型）：创建数据表
```
const moment = require('moment');
module.exports = function (sequelize, DataTypes) {
    return sequelize.define('article', {
        // 文章ID
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            allowNull: true,
            autoIncrement: true,
        },
        // 文章标题
        title: {
            type: DataTypes.STRING,
            allowNull: false,
            field: 'title',
        },
        // 文章作者
        author: {
            type: DataTypes.STRING,
            allowNull: false,
            field: 'author'
        },
        // 文章内容
        content: {
            type: DataTypes.TEXT,
            allowNull: false,
            field: 'content'
        },
        // 文章分类
        category: {
            type: DataTypes.STRING,
            allowNull: false,
            field: 'category'
        },
        // 创建时间
        createdAt: {
            type: DataTypes.DATE,
            get() {
                return moment(this.getDataValue('createdAt')).format('YYYY-MM-DD HH:mm:ss');
            }
        },
        // 更新时间
        updatedAt: {
            type: DataTypes.DATE,
            get() {
                return moment(this.getDataValue('updatedAt')).format('YYYY-MM-DD HH:mm:ss');
            }
        }
    }, {
        // 如果为 true 则表的名称和 model 相同，即 user
        // 为 false MySQL创建的表名称会是复数 users
        // 如果指定的表名称本就是复数形式则不变
        freezeTableName: true
    })

}
```
modules（模型）：操作数据表（增删改查）
```
// 引入刚刚建立连接mysql数据库的db.js文件
const db = require('../config/db');
// 引入Sequelize对象
const Sequelize = db.sequelize;
// 引入上一步的文章数据表模型文件
const Article = Sequelize.import('../schema/article');
// 自动创建表
Article.sync({force: false});
class ArticleModel {
    /**
     * 创建文章模型
     * @param data
     * @returns {Promise<*>}
     */
    static async createArticle(data) {
        return await Article.create({
            title: data.title, // 文章标题
            author: data.author, // 文章作者
            content: data.content, // 文章内容
            category: data.category, // 文章分类
        })
    }
    /**
     * 查询取文章详情数据
     * @param id  文章ID
     * @returns {Promise<Model>}
     */
    static async getArticleDetail(id) {
        return await Article.findOne({
            where: {
                id,
            },
        })
    }
}
module.exports = ArticleModel
```

controllers（控制器）：控制处理数据
```
const ArticleModel = require('../modules/article')

class articleController {
    /**
     * 创建文章
     * @param ctx
     * @returns {Promise.<void>}
     */
    static async create(ctx) {
        // 接收客服端
        let req = ctx.request.body;
        if (req.title // 文章标题
            && req.author // 文章作者
            && req.content // 文章内容
            && req.category // 文章分类
        ) {
            try {
                // 创建文章模型
                const ret = await ArticleModel.createArticle(req);
                // 把刚刚新建的文章ID查询文章详情，且返回新创建的文章信息
                const data = await ArticleModel.getArticleDetail(ret.id);

                ctx.response.status = 200;
                ctx.body = {
                    code: 200,
                    msg: '创建文章成功',
                    data
                }

            } catch (err) {
                ctx.response.status = 412;
                ctx.body = {
                    code: 200,
                    msg: '创建文章失败',
                    data: err
                }
            }
        } else {
            ctx.response.status = 416;
            ctx.body = {
                code: 200,
                msg: '参数不齐全',
            }
        }

    }

    /**
     * 获取文章详情
     * @param ctx
     * @returns {Promise.<void>}
     */
    static async detail(ctx) {
        let id = ctx.params.id;

        if (id) {
            try {
                // 查询文章详情模型
                let data = await ArticleModel.getArticleDetail(id);
                ctx.response.status = 200;
                ctx.body = {
                    code: 200,
                    msg: '查询成功',
                    data
                }

            } catch (err) {
                ctx.response.status = 412;
                ctx.body = {
                    code: 412,
                    msg: '查询失败',
                    data
                }
            }
        } else {
            ctx.response.status = 416;
            ctx.body = {
                code: 416,
                msg: '文章ID必须传'
            }
        }
    }
}

module.exports = articleController
```

4. 配置路由（接口）
```
const Router = require('koa-router')
const ArticleController = require('../controllers/article')

const router = new Router({
    prefix: '/api/v1'
})

// 创建文章接口（路由）
router.post('/article', ArticleController.create);
// 获取文章详情接口（路由）
router.get('/article/:id', ArticleController.detail);

module.exports = router
```

5. 配置跨域
```
npm install koa-cors --save
//在根目录app.js文件下引入koa-cors
const cors = require('koa-cors');
// 使用koa-cors
app.use(cors());
```




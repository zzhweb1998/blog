---
title: sqlite3配置到koa2
date: 2020-10-24 16:22:17
tags: ["sqlite3","koa2"]
categories: 练习笔记
copyright: false
---
## sqlite3配置到koa2项目
本次项目通过sqlite3嵌入到koa2项目中
1. 下载所需的插件，这边是通过sqlite3+sequelize配合使用
npm install sequelize@^5.x.x --save（sequelize5版本以上使用import会报：sequelize.import is not a function）
npm install --save sqlite3

2. 文件
config>sqlite.js：配置连接数据库
```
const path = require('path')
const Sequelize = require('sequelize')
const sqlite3 = require('sqlite3').verbose()

const sequelize = new Sequelize(undefined, undefined, undefined, {
  // sqlite的存储位置,仅sqlite有用
  storage: path.join(__dirname, '../db/database.sqlite'),
  // 自定义链接地址，可以是ip或者域名，默认本机：localhost
  host: 'localhost',
  // 数据库类型，支持: 'mysql', 'sqlite', 'postgres', 'mssql'
  dialect: 'sqlite',
  dialectModule: sqlite3,
  operatorsAliases: false,
  // 连接池配置
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },
  // 数据库默认参数,全局参数
  define: {
    underscored: false,
    freezeTableName: false,
    charset: 'utf8',
    dialectOptions: {
      collate: 'utf8_general_ci'
    },
    timestamps: true
  },
  // 是否开始日志，默认是用console.log
  logging: true,
}) // sqlite： 前三个参数传 undefined。


module.exports = {
  sequelize
}
```
db>database.sqlite：sqlite3数据库

schema>actionsLog.js：配置数据表
```
const moment = require('moment');
module.exports = function (sequelize, DataTypes) {
    return sequelize.define('actionsLog', {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            allowNull: true,
            autoIncrement: true,
        },
        pid: {
            type: DataTypes.INTEGER,
            allowNull: true,
            field: 'pid',
        },
        action_host: {
            type: DataTypes.STRING(50),
            allowNull: true,
            field: 'action_host',
        },
        action_port: {
            type: DataTypes.INTEGER,
            allowNull: true,
            field: 'action_port',
        },
        action_bytes: {
            type: DataTypes.INTEGER,
            allowNull: true,
            field: 'action_bytes',
        },
        action_socketFd: {
            type: DataTypes.INTEGER,
            allowNull: true,
            field: 'action_socketFd',
        },
        action_type: {
            type: DataTypes.INTEGER,
            allowNull: true,
            field: 'action_type',
        },
        timestamp: {
            type: DataTypes.STRING(100),
            allowNull: true,
            field: 'timestamp',
        },
        uuid: {
            type: DataTypes.STRING(100),
            allowNull: true,
            field: 'uuid'
        },
        withAttachment: {
            type: DataTypes.BOOLEAN,
            allowNull: true,
            field: 'withAttachment'
        },
    }, {
        // 如果为 true 则表的名称和 model 相同，即 user
        // 为 false MySQL创建的表名称会是复数 users
        // 如果指定的表名称本就是复数形式则不变
        freezeTableName: true
    })

}
```
modules>actionsLog.js：操作数据表
```
const db = require('../config/sqlite');
// 引入Sequelize对象
const Sequelize = db.sequelize;
// 引入上一步的文章数据表模型文件
const ActionsLog = Sequelize.import('../schema/actionsLog');
// 自动创建表
ActionsLog.sync({
    force: false
});

class ActionsLogModel {
    /**
     * 创建文章模型
     * @param data
     * @returns {Promise<*>}
     */
    static async createActionsLog(data) {
        let arr = []
        for(let con of data){
            arr.push({
                pid: con.pid,
                action_host: con.action.host || '',
                action_port: con.action.port || null,
                action_bytes: con.action.bytes || null,
                action_socketFd: con.action.socketFd,
                action_type: con.action.type,
                timestamp: con.timestamp,
                uuid: con.uuid,
                withAttachment: con.withAttachment,
            })
        }
        return await ActionsLog.bulkCreate(arr)
    }

    static async getActionsLogAll() {
        return await ActionsLog.findAll()
    }
    static async delActionsLogAll() {
        return await ActionsLog.destroy({ 
            where: {}, 
            truncate: true 
        }) 
    }
}

module.exports = ActionsLogModel
```
controllers>actionsLog.js：处理数据表
```
const ActionsLogModel = require('../modules/actionsLog')
const childProcess = require('child_process');
const path = require('path');

const ffi = require('ffi-napi');
const fs = require('fs')

const exec = childProcess.exec

class ActionsLogController {
    /**
     * 创建文章
     * @param ctx
     * @returns {Promise.<void>}
     */
    static async create(ctx) {
        // 接收客服端
        try {
            let pid = 13772;
            let arr = []
            let con = fs.readFileSync('C:/Users/zhouz/Documents/WinMonitor/' + pid + '/actions.log', 'utf-8');
            con = con.split("\n")
            for (let key in con) {
                if (key < con.length - 1) {
                    con[key] = JSON.parse(con[key])
                    con[key]['pid'] = pid
                    // 创建文章模型
                    arr.push(con[key])
                }
            }
            let data = await ActionsLogModel.createActionsLog(arr);

            ctx.response.status = 200;
            ctx.body = {
                code: 200,
                msg: '新建成功',
                data
            }

        } catch (err) {
            ctx.response.status = 412;
            ctx.body = {
                code: 412,
                msg: '新建失败',
                data: err
            }
        }

    }



    static async listAll(ctx) {
        try {
            // 查询文章详情模型
            let data = await ActionsLogModel.getActionsLogAll();
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
                data:err
            }
        }
    }


    static async delete(ctx) {
        try {
            // 查询文章详情模型
            let data = await ActionsLogModel.delActionsLogAll();
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
                data:err
            }
        }
    }


    static async ffiNapi(ctx) {
        try {
            let data;
            let pid = 13772;
            const ffipath = path.join(__dirname, '../dll/DllInjectHelperDll.dll')
            const DllInjectHelperDll = new ffi.Library(ffipath, {
                'InjectDll': ['int', ['int', 'string']]
            });
            let code = DllInjectHelperDll.InjectDll(pid, 'D:/privateCode/koa2/koa2_demo/dll/FunctionHookRemoveHelper32.dll')
            if (code === 1) {
                data = fs.readFileSync('C:/Users/zhouz/Documents/WinMonitor/' + pid + '/actions.log', 'utf-8');
                data = data.split("\n")
                for (let key in data) {
                    if (key < data.length - 1) {
                        data[key] = JSON.parse(data[key])
                    }
                }
            } else {
                data = "监听失败，检查监听的程序是否是32位"
            }
            ctx.response.status = 200;
            ctx.body = {
                code: 200,
                msg: '查询成功',
                data: {
                    data,
                    dir: ffipath
                }
            }

        } catch (err) {
            ctx.response.status = 412;
            ctx.body = {
                code: 412,
                msg: '查询失败',
                data: err
            }
        }

    }
}

module.exports = ActionsLogController
```
其中：controllers调取modules返回的数据并处理返回接口数据的格式，modules拿到schema
表并对数据表进行增删改查等操作

<center><font size="5">浅入浅出，node实现简单的接口</font></center>  
<br />  

> 从未深入研究node，感觉做好一个接口已经足够满足一只前端jbdog对node浅层次的理解。  

一、 demo环境：centos 7, mysql 5.7, node 8.10.0, node-forever, express
二、 步骤：
1. 构建服务器环境。
2. 数据库建表
```
    所有的表
    （ 关联关系: user_info中store_id关联表store_info的id，area_id关联表area_info的id；store_info中的area_id关联area_info中的id ）
    +-------------------------+
    | Tables_in_demo          |
    +-------------------------+
    | area_info               |
    | store_info              |
    | user_info               |
    +-------------------------+  

    user_info表结构
    +----+--------------+-----------------+---------------+---------+----------+---------------------+
    | id | user_name    | institution     | person_liable | area_id | store_id | entry_time          |
    +----+--------------+-----------------+---------------+---------+----------+---------------------+
    |  1 | 陈老师        | ***公司          | Chanphy       |       1 |        1 | 2019-01-26 17:02:54 |
    +----+--------------+-----------------+---------------+---------+----------+---------------------+   

    store_info表结构
    +----+---------+------------+
    | id | area_id | store_name |
    +----+---------+------------+
    |  1 |       1 | 广州        |
    +----+---------+------------+  

    area_info表结构
    +----+-----------+
    | id | area_name |
    +----+-----------+
    |  1 | 华南       |
    +----+-----------+   
```
3. [使用Express 应用程序生成器生成项目应用](http://www.expressjs.com.cn/starter/generator.html)
4. [项目增加sequelize](http://docs.sequelizejs.com/manual/installation/getting-started.html),sequelize为node的一个orm插件，主要用来操作数据库，[为什么要使用orm](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&tn=baidu&wd=%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E4%BD%BF%E7%94%A8orm&oq=markdown%2520%25E8%25AF%25AD%25E6%25B3%2595%2520%25E8%25A1%25A8%25E6%25A0%25BC&rsv_pq=95bde1d100028cbb&rsv_t=23c11cemyUo0KFz4ZQgFxvg03YbzC5khsbXB%2F12n7swWWh8%2F93RdDkr74Jc&rqlang=cn&rsv_enter=1&inputT=4142&rsv_sug3=42&rsv_sug1=32&rsv_sug7=100&rsv_sug2=0&rsv_sug4=4142)
5. 使用node-forever开启node进程守护  

三、如何在express中书写一个接口
    1. 接口统一在/routes中进行管理，新建一个get_user_list.js
```javascript
    // 引入express
    let express = require('express');
    // 引入路由
    let router = express.Router();
    // 引入sequelize
    const Sequelize = require('sequelize');
    // 建立连接
    const sequelize = new Sequelize('yourDataBaseName', 'yourUserName', 'yourPassword', {
        host: 'localhost',
        dialect: 'mysql',
        port: 3306
    })
    // 创建接口信息
    router.get('/', async (req, res) => {
        const user = sequelize.define('user_info', {},
        {
        // 不加会导致出现复数users未找到的错误
        freezeTableName: true,
        timestamps: false
        });
        let usersList;
        // 找出用户信息
        await user.findAll().then(users=> {
            usersList = users;
        });
        // 关联查询
        const area = sequelize.define('area_info', {},
        {
            // 不加会导致出现复数users未找到的错误
            freezeTableName: true,
            timestamps: false
        });
        for(let item of usersList) {
            await area.findAll({
                where: {
                    id: item.area_id
                }
            }).then(areaList => {
                item.area_name = areaList[0].area_name;
            })
        }
        // 关联查询
        const area = sequelize.define('store_info', {},
        {
            // 不加会导致出现复数users未找到的错误
            freezeTableName: true,
            timestamps: false
        });
        for(let item of usersList) {
            await area.findAll({
                where: {
                    id: item.store_id
                }
            }).then(storeList => {
                item.area_name = storeList[0].store_name;
            })
        }
        res.send({usersList});
    })
    module.exports = router;
```
2. 在/app.js中引入
```javascript
    ...  

    var getTotal = require('./routes/get_user_list');
    app.use('/user_list', getTotal);  

    ...
```  
四、 最后使用forever开始进程守护
```
    $ forever start ./bin/www
```

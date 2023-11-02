# 深入掌握全栈开发：基于Vue3和Node.js的个人博客系统


### 前言

时间回到2023.03.16号傍晚，下着浠沥沥的小雨，街道上的灯光在雨滴中泛着微弱的光芒，拖着疲惫的身体漫无目的地散步，雨水洒落在脸上，沉浸在自己的思考中，回顾着过去，展望着未来。感觉身体的清凉和思考的愈加清澈：不管怎样，这个时刻是属于我的，留下了关于生活的瞬间回忆。这种瞬间回忆使我意识到生活中的美好和无限可能。或许正是这种内心的冥想和反思，激发了自己的创造力和决心，希望去实现一个独特而有意义的项目。今后回首也能看到年轻的自己留下了一个特别的礼物，也是我青春的见证。

于是计划着**基于Vue3、JavaScript、Node.js、MongoDB以及UniApp这些强大的技术工具，打造一个个人博客全栈全端系统**。

### 项目预览

- 前台主页：[ZHOUYI-个人主页](https://www.zhouyi.run/#/index)
- 管理端主页：[ZHOUYI's ADMIN](http://admin.zhouyi.run/#/index)
- 小程序

![img](https://picx.zhimg.com/80/v2-2ab950f79989d01cb5b1f1e03bdcdd99_720w.png?source=d16d100b)





微信小程序

- 源码：[ZHOUYI/vue3个人博客主页前后端分离系统](https://gitee.com/Z568_568/ZHOUYI-Homepage.git)

### **技术栈选择**

- **前端框架：** Vue3 - 强大的JavaScript框架，提供了响应式数据绑定和组件化开发。
- **后端框架：** Express - 基于Node.js的后端框架，用于处理HTTP请求和路由。
- **数据库：** MongoDB - 非关系型数据库，用于存储博客文章、用户信息等数据。
- **移动端开发：** UniApp - 跨平台移动应用框架，构建博客微信小程序平台。

### **系统架构**

![img](https://picx.zhimg.com/80/v2-475ae022a96eea9b51607b536b2f4bdd_720w.png?source=d16d100b)





> 博客系统的架构分为前端、后端和数据库三个主要部分，前端由Vue3和UniApp构建，后端使用Express处理HTTP请求，MongoDB用于数据存储。

### **前端开发**

1. 管理端和前台项目目录和截图

![img](https://pic1.zhimg.com/80/v2-ea4ba3b97e18b3454a1a8cdef78df3af_720w.png?source=d16d100b)



![img](https://pica.zhimg.com/80/v2-2829c695e252a983efa2b9edaad531e4_720w.png?source=d16d100b)



![img](https://pic1.zhimg.com/80/v2-7fc50cb2641e779015ec504d767a4a47_720w.png?source=d16d100b)





![img](https://pic1.zhimg.com/80/v2-49d9f8c9dfe2e87135be49d0bc1a0a2e_720w.png?source=d16d100b)





![img](https://picx.zhimg.com/80/v2-b239ab52867748893654205b2296650d_720w.png?source=d16d100b)



2. 微信小程序项目目录和截图

![img](https://picx.zhimg.com/80/v2-c238f6bd3095fe401fed5444ce3d8dd9_720w.png?source=d16d100b)



![img](https://picx.zhimg.com/80/v2-30b9374ff8aa1a8a238e8deaf8f1ae87_720w.png?source=d16d100b)



### **后端开发**

![img](https://pic1.zhimg.com/80/v2-edc1a2bec7195128d821a81a53a588e1_720w.png?source=d16d100b)



**后端Express主要采用 模型层（models）、控制器(controllers)、路由层(routers)**

- **模型层（models）：**

模型层通常负责定义和管理与数据存储相关的内容。

```js
const mongoose = require('mongoose')
let schema = new mongoose.Schema({
        title: {
            type: String,
            required: true,
            comment: '博文名称'
        },
        cover: {
            type: String,
            comment: '博文封面'
        },
        abstract: {
            type: String,
            comment: '博文摘要'
        },
        content: {
            type: String,
            required: true,
            comment: '博文内容'
        },
        userId: {
            type: mongoose.Schema.Types.ObjectId, // 使用 ObjectId 类型来引用角色表
            ref: 'users', // 关联的角色表名称
            comment: '作者'
        },
        remark: {
            type: String,
            comment: '备注',
        },
        category: {
            type: Number,
            comment: '分类 1 技术 2 生活 3 其他',
        },
        tags: {
            type: String,
            comment: '标签',
        },
    },
    {
        timestamps: true,
        versionKey: false, // 设置不需要version  _V:0
    });
    ...其他字段
module.exports = mongoose.model('blog_articles', schema);
```

- **控制器（controllers）：**

控制器是处理路由请求和业务逻辑的地方。

```js
/**
 * 创建博文管理
 * @param {Object} req - 请求对象，包含查询参数
 *    - params: 包含当前页码、页面大小和查询参数
 * @returns {object} 200 - 成功响应
 * @returns {object} 400 - 参数验证错误
 * @returns {Error} default - 未知错误
 * @security JWT - 需要提供有效的访问令牌
 */
exports.blog_articlesCreate = [
    tokenAuthentication,
    checkApiPermission('blog:blog_articles:create'),
    checkUserRole(),
    actionRecords({module: '博文管理/创建'}),
    [
        body("title").notEmpty().withMessage('博文名称不能为空.'),
        body("content").notEmpty().withMessage('博文内容不能为空.'),
    ],
    async (req, res, next) => {
        try {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                return apiResponse.validationErrorWithData(res, "参数错误.", errors.array()[0].msg);
            }
            const newBlog_articles = {
                ...req.body
            };
            newBlog_articles.userId = req.userId
            await Blog_articlesModel.create(newBlog_articles);
            return apiResponse.successResponseWithData(res, "添加博文管理成功.", newBlog_articles);
        } catch (err) {
            next(err);
        }
    }
];
```

- **路由层（routers）：**

实际的接口路径定义的地方。创建博文接口地址示例：POST /v1/blog/blog_articles/create 

```js
const express = require('express');
const router = express.Router();
const Blog_articlesController = require('@controllers/v1/blog/Blog_articlesController')

/****************************************************************************/

/**
 * 创建博文管理
 * @route POST /v1/blog/blog_articles/create
 * @group 博文管理管理 - 博文管理相关
 * @returns {object} 200 - 成功响应
 */
router.post('/create', Blog_articlesController.blog_articlesCreate);
module.exports = router;
```

### **全栈开发**

- PC前端请求数据axios的封装

```javascript
import axios from 'axios';
import dbUtils from "../libs/util.strotage";
import {ZyNotification} from "../libs/util.toast";
// 不能使用useRouter ,useRoute，他们需要在setup中调用执行后才能用
import router from '@/router'
// 创建 Axios 实例
const instance = axios.create({
    timeout: 5000,
});
// 请求拦截器
instance.interceptors.request.use(
    (config) => {
        // 在请求发送之前可以进行一些处理，例如设置 token、添加请求头等
        config.headers.authorization = dbUtils.get('token')

        return config;
    },
    (error) => {
        return Promise.reject();
    }
);


// 响应拦截器
instance.interceptors.response.use(
    (response) => {
        // 在接收到响应数据之前可以进行一些处理，例如解析响应数据、错误处理等
        // ...
        const contentType = response.headers['content-type'];
        if (contentType === 'application/octet-stream' ||
            contentType === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') {
            // 注意：Blob类型文件下载需要请求头参数添加 responseType:'blob'  下载 导出等功能需要
            downloadFile(response)
        } else {
            // 响应数据是有效的 JSON 格式，继续处理
            return Promise.resolve(response.data);
        }
    },
    (error) => {
        // 统一处理错误
        return handleRequestError(error)
    }
);

// 下载blob二进制文件
const downloadFile = (response) => {
    const url = window.URL.createObjectURL(new Blob([response.data]));
    const filename = response.headers['x-filename'];

    axios.get(url, {responseType: 'blob'}).then((res) => {
        const blob = new Blob([res.data]);
        if (window.navigator.msSaveBlob) {
            // 兼容 IE，使用 msSaveBlob 方法进行下载
            window.navigator.msSaveBlob(blob, decodeURIComponent(filename));
        } else {
            // 创建一个 <a> 元素
            const link = document.createElement('a');
            link.href = window.URL.createObjectURL(blob);
            link.setAttribute('download', decodeURIComponent(filename));
            // 模拟点击下载
            link.click();
            // 清理 URL 和 <a> 元素
            link.remove();
            window.URL.revokeObjectURL(url);
        }
    });
}

// 统一处理错误
const handleRequestError = (error) => {
    // 进行错误处理
    if (error.response) {
        // 服务器响应错误
        let status = error.response.status
        // 在这里可以进行错误处理逻辑，例如弹出错误提示、记录错误日志等
        switch (status) {
            case 400:
                console.error('参数校验失败:', error.response.data.message);
                ZyNotification.error(error.response.data.message || '参数校验失败')
                return Promise.reject(error.response.data.message ?? '参数json解析失败');
            case 401:
                console.error('未授权:', error.response.data.message);
                if (error.response.data.message === 'Token不存在或已过期') {
                    router.replace('/login')
                    ZyNotification.error('账号已过期,请重新登录')
                    return Promise.reject({error: 'Unauthorized', message: '账号已过期,请重新登录'});
                } else {
                    ZyNotification.error(error.response.data.message || '账号已过期,请重新登录')
                    return Promise.reject({error: '401', message: error.response.data.message});
                }

            case 404:
                console.error('404:', error.response.data.message);
                ZyNotification.error(error.response.data.message || '资源不存在')
                return Promise.reject({error: '接口不存在', message: error.response.data.message});
            case 500:
                console.error('服务器内部错误:', error.response.data.message);
                ZyNotification.error(error.response.data.message || '服务器内部错误')
                return Promise.reject({error: '服务器内部错误', message: error.response.data.message});
            default:
                ZyNotification.error('服务器响应错误')
                console.error('服务器响应错误:', error.response.data);
        }

    } else if (error.request) {
        // 请求未收到响应
        console.error('请求未收到响应:', error.request);
        ZyNotification.error('请求未收到响应')
        // 在这里可以进行错误处理逻辑，例如弹出错误提示、记录错误日志等
    } else {
        // 请求配置出错
        console.error('请求配置出错:', error.message);
        ZyNotification.error('请求配置出错')
        // 在这里可以进行错误处理逻辑，例如弹出错误提示、记录错误日志等
    }
}

// 封装请求方法
class AxiosService {
    constructor() {
        if (AxiosService.instance) {
            return AxiosService.instance;
        }
        AxiosService.instance = this;
    }

    // GET 请求
    get(url, params = null) {
        return instance.request({
            method: 'get',
            url,
            params,
        });
    }

    // POST 请求
    post(url, data = null, params = null, responseType) {
        return instance.request({
            method: 'post',
            url,
            data,
            params,
            responseType
        });
    }

    // PUT 请求
    put(url, data = null, params = null) {
        return instance.request({
            method: 'put',
            url,
            data,
            params,
        });
    }

    // DELETE 请求
    delete(url, params = null) {
        return instance.request({
            method: 'delete',
            url,
            params,
        });
    }

}
// 创建 AxiosService 实例
const axiosService = new AxiosService();

// 导出实例化后的 AxiosService 对象
export default axiosService;
```

- 小程序请求数据封装

```js
import config from "./config";
import log from "../utils/util.log";
import toast from "../utils/util.toast";

function getToken() {
    if (uni.getStorageSync('ZY-userInfo')) {
        return uni.getStorageSync('ZY-userInfo').token
    } else {
        return ''
    }
}

/**
 * @param method 请求方式
 * @param url 请求URL
 * @param data 请求数据
 * @returns {Promise}
 */
const request = function ({method, url, data = {}}) {
    return new Promise((resolve, reject) => {
        uni.request({
            url: config.SERVER_URL + url,
            data: data,
            header: {
                'Content-Type': 'application/json;charset=UTF-8',
                'authorization': getToken()
            },
            method: method,
            success: (res) => {
                switch (res.statusCode) {
                    case 200: {
                        let data = res.data.data || res.data;
                        resolve(data);
                        break;
                    }
                    default: {
                        let errData = res.data;
                        log.danger('********错误信息**********')
                        log.danger(`【URL】：${config.SERVER_URL + url}`)
                        log.danger(`【TYPE】：${method}`)
                        log.danger(`【CODE】：${res.statusCode}`)
                        log.danger(`【参数】：${JSON.stringify(data)}`)
                        log.danger(`【错误】：${errData.message || '服务器内部错误'}`)
                        log.warning(`【TIME】：${new Date().toLocaleTimeString()}`)
                        log.danger('*************************')
                        uni.showToast({
                            icon:'error',
                            title:errData.message|| '服务器内部错误'
                        })
                        reject(errData);
                        break;
                    }
                }
            },
            fail: (err) => {
                log.capsule('请求失败:', url, 'danger')
                toast.error('网络错误')
                reject(err);
            },
            complete: (res) => {
                // log.capsule('请求完成:',url,'primary')
            }
        });
    })
}

// 封装请求方法
class Service {
    // GET 请求
    get(url, params) {
        return request({
            method: 'get',
            url,
            params,
        });
    }

    // POST 请求
    post(url, data) {
        return request({
            method: 'post',
            url,
            data
        });
    }
}
export default new Service();
```

- 服务端处理前端请求在 控制器（controllers）的逻辑内

### **功能和特点**

- 用户注册和登录鉴权
- 支持Gitee和Github第三方登录
- 角色权限控制（基于角色）
- 按钮级权限控制
- 大屏图表地图数据展示
- 主题自定义、全局搜索
- 文章发布和编辑（支持上传md文件解析发布）
- 文章列表和详情展示
- 用户评论和留言点赞互动
- 邮件发送通知
- 定时任务
- 访客记录统计
- 操作日志
- 文章分类归档统计
- 代码生成器
- 文件上传预览
- 数据持久化存储
- 图片懒加载
- 前台一键配置页面动态展示
- 各种3D动画组件封装
- 社交分享和互动
- ...

### **未来展望**

1. **性能优化：** 进一步优化前端和后端性能，以确保系统快速响应，特别是在高访问量时。
2. **提高SEO:** 使用Vue的服务器端渲染框架，如Nuxt.js或者React的Next来进行重构前台。
3. **移动应用：** 考虑开发iOS和Android移动应用，以提供更多设备上的访问渠道。
4. **内容推荐和搜索：** 实现内容推荐算法和高级搜索功能，以助找到感兴趣的内容。
5. **多语言支持：** 考虑添加多语言支持，以吸引国际小伙伴使用。
6. **持续改进用户体验：** 不断收集用户反馈，改进用户界面和功能，以满足用户的期望和需求。
7. **博客主题：** 创建多个博客主题，能够选择不同的博客主题和自定义风格。
8. **教程和文档：** 提供使用系统的详细教程和文档，以帮助新用户快速上手。
9. **安全和隐私：** 持续关注安全漏洞，确保用户数据的隐私和安全。

### 总结

在构建这个个人博客全栈系统的过程中，积累了宝贵的经验，深入了解了Vue 3、Express、MongoDB和UniApp的构建应用的原理，认识到全栈开发的魅力和挑战，在全栈开发中能够站在上帝的视角去观察所有出现的BUG,可以很快速的定位问题的关键。

1. **技术多样性：** 这个项目涵盖了前端开发、后端开发、数据库管理、移动应用开发等多个领域，扩展了技术栈，提高了全栈开发和设计的能力。
2. **项目规划：** 在项目之前进行良好的规划和设计是关键。了解需求、选择合适的技术和工具，以及制定清晰的架构，可以大大减少后期的问题。
3. **前后端分离：** 将前端和后端分离并通过API进行通信是一种良好的实践。这提高了代码的模块化性和可维护性，前端和后端单独维护也是当前市场主要的形式。
4. **用户体验：** 关注用户体验是至关重要的。这个是最耗时间最耗脑细胞，写了一版然后又有新的想法，不停的修改。
5. **学习与改进：** 全栈开发是一个不断学习和改进的过程。技术领域不断演进，因此保持学习的态度非常重要。不断寻找新的工具和技术，以提高开发效率和性能。
6. **开源社区：** 从开源社区中获得支持和资源是非常有益的。许多开源项目和库可以加速开发，同时也为项目提供了一个展示和贡献的平台。
7. **源码阅读能力**：项目有用到不同的包库插件等会出现兼容vue3的语法，需要手动改造它们的源码进行兼容等等。
8. **组件封装设计能力：**不论是前端还是后端，需要具有抽象模块的思想，将公共的复杂的各种方法模块进行剥离开，有利于后期进行维护。
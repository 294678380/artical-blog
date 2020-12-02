
## 微前端大赏二-singlespa实践
[toc]
### 序

#### 介绍singleSpa

singleSpa是一个javascript库
它可以让很多小页面、小的组件、不同架构的前端组件在一个页面应用程序中共存。

这里有一个演示: (https://single-spa.surge.sh/)

这个库可以让你的应用可以使用多个不同的技术栈（vue、react、angular等等）,这样我们就可以做同步开发，最后再使用一个公用的路由即可实现路由完美切换。也可以使用一样的技术栈，分不同的团队进行开发，只需要最后使用这个库把它们整合在一起，设置不用的路由名称就可以了。

**优点:**
* 敏捷
独立开发和更快的部署周期： 开发团队可以选择自己的技术并及时更新技术栈。 一旦完成其中一项就可以部署，而不必等待所有事情完毕。
* 风险下降
降低错误和回归问题的风险，相互之间的依赖性急剧下降。

* 更小单元
更简单快捷的测试，每一个小的变化不必再触碰整个应用程序。

* 持续交付
更快交付客户价值，有助于持续集成、持续部署以及持续交付。

**缺点：**
* 配置复杂
singlespa相对来说配置复杂，当然我们还有更简单一点的qiankun，也可以基于singlespa封装一套更适合自己的框架。
* 一定的资源浪费
由于核心逻辑还是在于请求manifest，拿到js文件后执行渲染，这个过程不可避免会产生一些冗余，对于C端的应用来说，这个问题比较致命，当然，对于B端来说，这个是可以接受的，在可控制的范围之内

#### singleSpa核心逻辑
几张图可以解决singleSpa的核心逻辑
![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_2011301725479072df3196964d51abc0e7f250332962.jpg)
第一张图，很显然，第一步，在我们的webpack应用里生成一个manifest.json文件，这个文件内容差不多如下：
```
{
  "files": {
    "static/js/0.chunk.js": "/static/js/0.chunk.js",
    "static/js/0.chunk.js.map": "/static/js/0.chunk.js.map",
    "static/js/1.chunk.js": "/static/js/1.chunk.js",
    "static/js/1.chunk.js.map": "/static/js/1.chunk.js.map",
    "main.js": "/static/js/main.chunk.js",
    "main.js.map": "/static/js/main.chunk.js.map",
    "runtime-main.js": "/static/js/bundle.js",
    "runtime-main.js.map": "/static/js/bundle.js.map",
    "index.html": "/index.html",
    "static/media/logo.svg": "/static/media/logo.103b5fa1.svg"
  },
  "entrypoints": [
    "static/js/bundle.js",
    "static/js/0.chunk.js",
    "static/js/main.chunk.js"
  ]
}
```
关键点在 entrypoints 这个属性，我们可以通过manifest拿到项目的依赖表并可以使用script标签动态加载出来，这个时候我们就可以实现动态加载不同的微前端应用了。

![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_201130174007%E6%97%A0%E6%A0%87%E9%A2%98%E7%BB%98%E5%9B%BE.jpg)

第二张图，我画出了更加具体的，singlespa在渲染过程中的核心逻辑
1、 首先我们有 main（主app） child（子app），主app只有一个，子app可以有多个
2、 其次，主app上一般我们可以在index.html里面，写多几个空间，也就是多几个div
例如：
```
<div id=”react-app”></div>
<div id=”vue-app”></div>
```
3、然后，在我们的child上，我们要用webpack插件，生成一个带有所有需要加载的依赖文件的manifest.json
4、主应用去加载这个manifest.json，获取到具体的js，使用script标签把它放到主应用上，进行渲染

至此我们就可以完全搞清楚，为什么singlespa这么神奇了，接下来让我们搭建一个简易版的singlespa

### 搭建环境

#### vue main
由于我们需要使用webpack配置，而最新版本的vue-cli默认只有babel，我们用这个步骤来安装一个vue版本的main
1、装包
```
    npm install @vue/cli @vue/cli-init  -g
```
2、创建一个项目

```
vue init webpack demo-single
```

3、cd demo-single

4、装包
```
npm i single-spa single-spa-vue axios --save
```

5、在src目录创建一个singlespa配置文件 single-spa-config.js
```
    // single-spa-config.js
    import * as singleSpa from 'single-spa'; //导入single-spa
    import axios from 'axios'

    /*
        runScript：
        一个promise同步方法。可以代替创建一个script标签，然后加载服务
    */
    const runScript = async (url) => {
        return new Promise((resolve, reject) => {
            const script = document.createElement('script');
            script.src = url;
            script.onload = resolve;
            script.onerror = reject;
            const firstScript = document.getElementsByTagName('script')[0];
            firstScript.parentNode.insertBefore(script, firstScript);
        });
    };

    const getManifest = (url, bundle) => new Promise(async (resolve) => {
        const { data } = await axios.get(url);
        // eslint-disable-next-line no-console
        const { entrypoints } = data;

        for (let i = 0; i < entrypoints.length; i++) {
            await runScript('http://127.0.0.1:3000/' + entrypoints[i]).then(() => {
                if (i === entrypoints.length - 1) {
                    resolve()
                }
            })
        }
    });

    singleSpa.registerApplication( //注册微前端服务
        'singleDemoVue', async () => {
            let singleVue = null;
            await getManifest('http://127.0.0.1:3000/asset-manifest.json').then(() => {
                singleVue = window.singleReact;
            });
            return singleVue;    
        },
        location => location.pathname.startsWith('/react') // 配置前缀
    );

    singleSpa.start(); // 启动
```
**注： 可以看到，runScript就是个创建script标签的方法，getManifest是一个简单的获取manifest并创建script的方法**

6、在main.js里引入这个文件
```
import './single-spa-config'
```

7、运行 
npm run dev

最终得到这样一个工程

![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_201130181531fc277894-9970-4acc-9170-0aef2e457fd5.png)

这样我们就完成了一个入口的配置，当然它还很简单，更复杂的操作我们应该放在具体的工程上去做


#### react child

上面的代码可以看到，我们register了一个react应用 http://127.0.0.1:3000/asset-manifest.json 并且访问了它的manifest文件，现在我们需要创建一个react子应用，也是直接通过几个步骤来完成，我们使用create-react-app来快速搭建：

1、装包
```
npm install create-react-app -g
```
2、创建
```
npx create-react-app my-app
```
3、创建完成后，注意我们需要对webpack做一点修改，默认create-react-app会有一个git本地分支，让我们先提交到本地仓库一下
```
git status
git add ./
git commit -m ttt
```
4、拿到webpack配置文件，create-react-app默认隐藏了webpack配置文件
```
yarn eject 或 npm run eject
```
5、修改webpack文件
修改 /config/webpack.config.js 在output增加：
```
output: {
    ... 这里忽略了原有的
    library: 'singleReact',
    libraryTarget: 'window'
}
```
![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_2011301817110891255a-2940-44f5-a1b1-6bec5423c06d.png)


修改 /scripts/start.js文件，在const devServer = new ...这个地方，增加一个header的设置：
```
const devServer = new WebpackDevServer(compiler, {
      ...serverConfig,
      // 这里上增加的header设置
      headers: {
        'Access-Control-Allow-Origin': '*',
      }
  
    });
```
![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_20113018181813834fbd-2943-4980-aa4c-a0038b242b66.png)

6、修改src/index.js
一个是要把root改为动态渲染，一个是注册生命周期
```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import singleSpaReact, {SingleSpaContext} from 'single-spa-react';

const rootComponent = () => {
  ReactDOM.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>
    ,
    document.getElementById('react-root')
  );
}

// ReactDOM.render(
//   ,
//   document.getElementById('root')
// );


const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent,
  errorBoundary(err, info, props) {
    // https://reactjs.org/docs/error-boundaries.html
    console.error(err)
    return (
      <div>This renders when a catastrophic error occurs</div>
    );
  },
});
export const bootstrap = reactLifecycles.bootstrap;
export const mount = reactLifecycles.mount;
export const unmount = reactLifecycles.unmount;


// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

7、运行
npm run start

8、在main的vue那里，访问/react 你会看到下面有一个react渲染和vue的一起出现，大功告成


#### 生命周期
生命周期函数共有4个：bootstrap、mount、unmount、update。生命周期可以传入 返回Promise的函数也可以传入 返回Promise函数的数组。
引用一个大佬完整的说明, 非常的详细：
https://github.com/YataoZhang/my-single-spa/issues/4

### 结论
single spa可以给我们提供一整套方案，去搭建一套微前端集成框架，但它并不是一个开箱即用的封装，它有很多的坑等着我们去踩。
一般情况下，我们选择使用qiankun，它的封装程度更好，api更加友好一些。待积攒足够多的使用经验，可以考虑自研一套自己的微前端框架，增加整体的前端研发效率。
下节我将给大家带来qiankun对singlespa的封装，在具体应用中的实践。待完结框架篇后，我们可以再深入探究singlespa的实现原理以及各种概念。

### 参考文章
single-spa 文档: https://single-spa.js.org/docs/getting-started-overview/

微前端 single-spa: https://juejin.cn/post/6844903896884707342

这可能是你见过最完善的微前端解决方案！: https://www.infoq.cn/article/o6GxRD9iHQOplKICiDDU

single-spa微前端: http://www.soulapp.tech/2019/09/25/single-spa%E5%BE%AE%E5%89%8D%E7%AB%AF/

Single-Spa + Vue Cli 微前端落地指南 (项目隔离远程加载，自动引入) : https://juejin.cn/post/6844904025565954055


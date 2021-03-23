[toc]
# 序
 这才2月中旬，广州就已经渐渐地进入了夏季，——夏天总是让人焦虑的。过年闲暇时间写下了微前端这系列的终章，欢迎拍砖。
# qiankun原理和API介绍
qiankun是基于singlespa框架的一个上层应用，它提供了完整的生命周期，和一些钩子函数，通过路由匹配来动态加载注册微应用，同时提供了一系列api对微应用做管理和预加载等，它相对singlespa来说进步是比较大的。

所以---qiankun实质上是singlespa的一个封装，基于我们在上一节看到的，singlespa是通过输出一个manifest.json 通过标识入口信息动态构造script渲染实现的微前端应用，类似下面的图：
![avatar](https://images.cnblogs.com/cnblogs_com/ztfjs/841508/o_201130174007%E6%97%A0%E6%A0%87%E9%A2%98%E7%BB%98%E5%9B%BE.jpg)

回顾一下singlespa在渲染过程中的核心逻辑
1、 首先我们有 main（主app） child（子app），主app只有一个，子app可以有多个
2、 其次，主app上一般我们可以在index.html里面，写多几个空间，也就是多几个div
例如：
```html
<div id=”react-app”></div>
<div id=”vue-app”></div>
```
3、然后，在我们的child上，我们要用webpack插件，生成一个带有所有需要加载的依赖文件的manifest.json

4、主应用去加载这个manifest.json，获取到具体的js，使用script标签把它放到主应用上，进行渲染

在qiankun中对这套逻辑做了基本的封装, 让我们只需要经过简单的几个api就可以控制singlespa中比较复杂的配置和概念。

## 注册
```javascript
import { registerMicroApps, start } from 'qiankun';
registerMicroApps([
  {
    name: 'react app', // 应用名称
    entry: '//localhost:7100', // 应用入口，应用需要增加cors选项
    container: '#yourContainer', // 应用单独的appid的div
    activeRule: '/yourActiveRule', // 匹配路由
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    container: '#yourContainer2',
    activeRule: '/yourActiveRule2',
  },
]);
start();

```
## main
main是一个qiankun的主体部分，它也是不限制框架种类的，可以用react也可以用vue和angular，只需要在entry.js里面注册它就可以了。

一般情况下main的作用是存放公共代码，例如：
1、消息触发器
2、公共路由
3、权限触发器
4、存放例如全局管理、皮肤、用户管理等公共页面

你也可以把站点的首页写在这里，可以加快主体加载速度

## 生命周期

### bootstrap
boostrap相当于init，子应用在第一次加载的时候会调用这个方法， 一般可以在里面做一些项目的初始化操作，例如
### mount
每次在加载到子应用的时候都会调用它，就像是componentDidMount,一般情况下我们要把ReactDOM.render这样的初始化函数写在里面，每次mount时调用render
### unmount
这个跟mount正好相反，每一次注销/切换子应用的时候会调用它，一般我们在这里 ReactDOM.unmountComponentAtNode 注销这个应用，然后把整个项目的容器让出来
### update 
这是个可选的生命周期，子应用发生变化的时候会调用。


## 路由匹配
路由规则有两种，需要手动调用对应的子应用渲染就行了，通过一个叫loadMicroApp的方法挂载一个子应用组件，这样就可以在main中像配置一个正常的应用那样配置子应用的view了。
```javascript
import { loadMicroApp } from 'qiankun';
import React from 'react';
class App extends React.Component {
  containerRef = React.createRef();
  microApp = null;
  componentDidMount() {
    this.microApp = loadMicroApp(
      { name: 'app1', entry: '//localhost:1234', container: this.containerRef.current, props: { name: 'qiankun' } },
    );
  }
  componentWillUnmount() {
    this.microApp.unmount();
  }
  componentDidUpdate() {
    this.microApp.update({ name: 'kuitos' });
  }
  render() {
    return <div ref={this.containerRef}></div>;
  }
}
```
### 处理样式
## 沙箱
qiankun的沙箱模式是在start的api配置项里面开启的

sandbox 选项可选

```javascript
start({
  sandbox: true // true | false | { strictStyleIsolation?: boolean, experimentalStyleIsolation?: boolean }
})
```

默认情况下沙箱可以确保单实例场景子应用之间的样式隔离，但是无法确保主应用跟子应用、或者多实例场景的子应用样式隔离。当配置为 { strictStyleIsolation: true } 时表示开启严格的样式隔离模式。这种模式下 qiankun 会为每个微应用的容器包裹上一个 shadow dom 节点，从而确保微应用的样式不会对全局造成影响。
**shadow dom coco大神写过一篇文章介绍这个，我就不班门弄斧了**
https://www.cnblogs.com/coco1s/p/5711795.html 

## 样式冲突解决方案
qiankun 会自动隔离微应用之间的样式（开启沙箱的情况下），你可以通过手动的方式确保主应用与微应用之间的样式隔离。比如给主应用的所有样式添加一个前缀，或者假如你使用了 ant-design 这样的组件库，你可以通过这篇文档中的配置方式给主应用样式自动添加指定的前缀。

以 antd 为例：

配置 webpack 修改 less 变量
```javascript
{
  loader: 'less-loader',
+ options: {
+   modifyVars: {
+     '@ant-prefix': 'yourPrefix',
+   },
+   javascriptEnabled: true,
+ },
}
```
配置 antd ConfigProvider

```javascript
import { ConfigProvider } from 'antd';
export const MyApp = () => (
  <ConfigProvider prefixCls="yourPrefix">
    <App />
  </ConfigProvider>
);
```

## webpack配置的问题
微应用的打包工具还需要增加如下配置：
```javascript
  const packageName = require('./package.json').name;
  module.exports = {
    output: {
      library: `${packageName}-[name]`,
      libraryTarget: 'umd',
      jsonpFunction: `webpackJsonp_${packageName}`,
    },
  };
```

# qiankun实践-react微前端应用
## 起始，准备2个react应用
直接用create-react-app创建两个app应用
```
npx create-react-app main-app
npx create-react-app micro-app
```
得到一个文件夹里有两个项目
我们用main做主应用，micro做子应用，按照我们的api，子应用只需要配置一个register就可以引入子应用了
**其中子应用需要调出webpack配置，create-react-app默认是不允许手动配置的，使用命令就可以了**
进入micro-app的文件夹目录运行(create-react-app也有overload的办法更改配置，这里为了方便直接用命令调出来)：
```
npm run eject
```
这样项目的准备工作就做好了。
## 子应用配置
配置子应用两个步骤，一个是生命周期的配置。 我们把生命周期函数写好放到main.js中:

然后我们把reactDom.render放到mount生命周期里调用，让qiankun在准备好加载mount的时候再去初始化应用：

unmount的注销操作也不能忘记:

我们更改一下子应用的根节点id，在父应用中再去引用它(不要忘了html里也需要更改):

最后再把webpack中的配置修改一下：
1、修改devserver支持cors 修改端口
``
headers: {
      'Access-Control-Allow-Origin': '*',
    }
``
2、修改增加bundle的导出,在webpack.config.js增加配置：


## 父应用配置
然后我们就可以去在main应用中，注册了首先要 
```
npm install qiankun --save
```
然后在main文件index.js中注册子应用：
别忘了我们还需要在public/index.html中写一个div容器，id是我们子应用的那个id，用来承载子应用的渲染:

然后我们就可以开始运行看一看了:
运行成功，随便改一下micro的样式看看效果：
接下来我们需要处理一下路由跳转的问题。
## 路由的处理实践
前文有提到，在react中使用qiankun可以使用apiloadMicroApp,这里我们也用它来处理路由的跳转。
我们主要是在main-app中操作：
首先新建micro-app的view文件（每多一个子应用就新建一个）：

然后使用react-router直接配置：
由于create-react-app默认没有直接提供react-router，我们手动下一个
```
npm install react-router react-router-dom --save
```
改完index.js长这样：
再试一下：

## 结论和源码
相比较上一次我们看见的 singlespa的配置要简单了很多，而且更加直白，新增子应用更加无缝。
需要demo源码的同学企业微信 张泰峰 即可或许

# 应用场景和坑

## 坑-静态资源问题解决

微应用打包之后 css 中的字体文件和图片加载如果使用的加载路径是相对路径，会导致css 中的字体文件和图片加载 404。

而 css 文件一旦打包完成，就无法通过动态修改 publicPath 来修正其中的字体文件和背景图片的路径。

主要有三个解决方案：

* 所有图片等静态资源上传至 cdn，css 中直接引用 cdn 地址（推荐）
* 借助 webpack 的 url-loader 将字体文件和图片打包成 base64（适用于字体文件和图片体积小的项目）（推荐）
* 使用绝对地址，nginx中设置静态目录

# 结束语
qiankun整体的思路是比较ok的，它大大简化了singlespa的使用逻辑，让微前端的门槛变得更低，但它仍然有一些缺点，例如部分api总是会有莫名其妙的问题，例如api文档不是特别的直观等，这些都是待改进的地方。而对于微前端来说，做到能够技术栈无关、渐进升级旧项目、分离不同业务等功能就已经能发挥它的最大价值了。

## 快速上手
### 安装过程简介
* 首先安装lcd-cli (推荐使用yarn安装lcd-cli) 如果你还没有使用过yarn，请见 https://yarn.bootcss.com/
* 紧接着通过lcd-cli的命令行启动**快速开始服务**
* 选定项目后点击生成，在指定目录中生成脚手架基座
* 进入文件根目录，安装相关的依赖

### 安装流程
* 安装yarn 
https://yarn.bootcss.com/docs/getting-started/
* 设置内部nexus源
```
yarn config set registry http://nexus.sheincorp.cn:4873/
```
* 全局安装lcd-cli (注意，请不要使用sudo，会导致不可预料的问题)
```shell
    yarn global add lcd-cli 
```
* 通过命令启动**快速开始服务**
```shell
    lcd-cli ui
```
* 在自动打开的界面中创建
>选择项目=》选择一个合适的目录=》新建一个你的项目=》选择对应的信息后，完成看见右上角成功提示就可以关闭了(框架版本可不选)

* 进入生成的文件根目录

* 运行初始化,主要的目的是初始化webpack相关的信息
```shell
yarn bootstrap
```
* 安装依赖
可参考的package.json如下，记录了一个完整的lcd项目的依赖信息，具体可参考**依赖**, 
你可以直接复制到package.json中使用
```json
{
   "name": "lcd-scaffold",
   "version": "0.0.14",
   "description": "",
   "main": "index.js",
   "scripts": {
      "bootstrap": "lcd-cli --bootstrap",
      "start": "lcd-cli --start",
      "build": "NODE_ENV=production lcd-cli --build",
      "analysis": "ANALYSIS=true NODE_ENV=production lcd-cli --build",
      "lint": "eslint --ext .js --ext .jsx ."
   },
   "keywords": [],
   "author": "stoney",
   "license": "ISC",
   "dependencies": {
      "@shein-components/Aside": "^1.1.0",
      "@shein-components/Feedback": "^0.9.11",
      "@shein-components/Icon": "^2.0.4",
      "@shein-components/Layout": "^0.9.9-rc.3",
      "@shein-components/SearchArea": "^1.0.9",
      "@shein-components/TableSorter": "^1.0.7-rc.11",
      "@shein-components/UserGuide": "^0.9.1",
      "@shein/cloud-sdk": "^0.9.6",
      "classnames": "^2.2.5",
      "date-fns": "2.0.0-alpha.11",
      "history": "^4.9.0",
      "lodash.clonedeep": "^4.5.0",
      "query-string": "^5.1.1",
      "ramda": "^0.26.1",
      "react": "16.8.6",
      "react-dom": "16.8.6",
      "sheinq": "^1.1.38",
      "react-redux": "^7.2.2",
      "redux": "^4.0.5",
      "shineout": "1.6.3-rc.3"
   },
   "devDependencies": {
      "@babel/core": "^7.13.8",
      "@babel/plugin-proposal-class-properties": "^7.13.0",
      "@babel/preset-env": "^7.13.9",
      "@babel/preset-react": "^7.12.13",
      "@babel/preset-typescript": "^7.13.0",
      "@shein/webpack-external-map": "^0.6.14",
      "autoprefixer": "^9.8.5",
      "babel-loader": "^8.2.2",
      "css-loader": "3.6.0",
      "cssnano": "^4.1.10",
      "ejs-loader": "^0.3.3",
      "file-loader": "^6.2.0",
      "html-webpack-plugin": "4.5.0",
      "less": "^4.1.1",
      "less-loader": "4.1.0",
      "postcss-loader": "4.1.0",
      "react-redux-component-loader": "7.7.0",
      "rrc-loader-helper": "^7.8.1-beta.0",
      "style-loader": "^2.0.0",
      "vue-loader": "^16.0.0-beta.4",
      "webpack": "4.5.0",
      "webpack-cli": "3.3.12",
      "webpack-merge": "^4.2.2"
   },
   "peerDependencies": {
      "babel-polyfill": "*",
      "react-redux": "*",
      "react-router": "*",
      "react-router-dom": "*",
      "react-router-redux": "*",
      "redux": "*",
      "redux-saga": "*",
      "rrc-loader-helper": "*",
      "sheinq": "*",
      "vue": "*"
   }
}


```
* 运行install
```shell
yarn install
```
* 运行项目
```
yarn start
```
### 注意事项
* webpack webpack-dev-server webpack-cli 需要全局安装
```
yarn global add webpack@4.5.0 webpack-dev-server@3.11.2 webpack-cli@3.3.12
```
* 提示Error: Cannot find module 'vue-loader/lib/plugin'
在.webpack.base.config.js中去掉vue-loader的引用

## 版本与环境
## 目录定义
## 基本术语
## 接入TypeScript

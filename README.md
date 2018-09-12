# 原项目地址

> https://github.com/hql7/tree-transfer

这个包只是测试。不依赖element-ui的版本

# 安装

`npm i tree-transfer-ele -S`

# 需要覆盖其他版本element-ui的样式
```
<style>
#transfer .el-tree-node__expand-icon{
    border: 0; margin: 0; width: auto; height: auto;font-size: 12px;
}
#transfer .el-tree{
    border:0;
}
</style>
```
## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

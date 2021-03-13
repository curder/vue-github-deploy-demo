## 使用GitHub Actions服务部署Vue.js项目到GitHub Pages

![GitHub Pages Deploy](https://github.com/curder/vue-github-deploy-demo/workflows/Deploy/badge.svg)

- 创建一个vue配置文件
- 安装依赖
- 创建部署脚本
- 更新 `package.json` ，添加部署命令
- 创建 GitHub Actions 工作流程

### 创建Vue配置文件

部署 [Vue.js](http://vuejs.org) 应用程序时，需要指定公共路径，以便正确构建页面。

这通常是域名之后的 uri 部分（例如example.com/path），使用 GitHub 页面时，路径将是仓库的名称，比如这里的 **vue-github-deploy-demo**。

使用下面的示例在应用程序根文件夹中创建一个`vue.config.js`，并确保替换第3行中的路径。

```js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production'
  ? '/vue-github-deploy-demo'
  : '/'
}
```
> 修改第三行的 **vue-github-deploy-demo** 为你当前仓库。

### 安装依赖项

使用 execa 运行命令来构建和部署我们的应用程序。

如果使用 npm 运行：`npm install execa --save-dev`

或用 yarn ：`yarn add execa --dev`

### 创建部署脚本

现在，将创建一个部署脚本，以处理代码的实际构建和部署。

创建 `scripts` 目录并在下面添加 `gh-pages-deploy.js`部署脚本文件，当然可以将部署脚本命名成任意的名称，但是这个文件在下面的步骤将用上。

```js
/* eslint-disable no-console */
const execa = require("execa");
const fs = require("fs");
(async () => {
  try {
    await execa("git", ["checkout", "--orphan", "gh-pages"]);
    // eslint-disable-next-line no-console
    console.log("Building started...");
    // await execa("npm", ["run", "build"]);
    await execa("yarn", ["build"]);
    // Understand if it's dist or build folder
    const folderName = fs.existsSync("dist") ? "dist" : "build";
    await execa("git", ["--work-tree", folderName, "add", "--all"]);
    await execa("git", ["--work-tree", folderName, "commit", "-m", "gh-pages"]);
    console.log("Pushing to gh-pages...");
    await execa("git", ["push", "origin", "HEAD:gh-pages", "--force"]);
    // await execa("rm", ["-r", folderName]);
    // await execa("git", ["checkout", "-f", "master"]);
    // await execa("git", ["branch", "-D", "gh-pages"]);
    console.log("Successfully deployed, check your settings");
  } catch (e) {
    // eslint-disable-next-line no-console
    console.log(e.message);
    process.exit(1);
  }
})();
```

> 如果在 github 上默认的分支为 main，则需要修改脚本第18行的**master**为**main**即可。
> 如果项目使用的是 npm ，请主食掉第 10 行和打开第 9 行，以使用 npm。

### 更新package.json

最后，需要更新 package.json 包含我们的部署脚本命令，添加一下行

```json
"scripts": {
    "deploy": "node scripts/gh-pages-deploy.js"
}
```

至此，部署脚本已经准备就绪，执行 `npm run deploy` 或 `yarn deploy` 部署到 GitHub Pages 服务。

如果您也想自动化，让我们在下一步添加GitHub操作！


### 创建GitHub操作工作流程

GitHub Actions 是一种简单易行的方法，可以在您定义的触发器上运行代码上的脚本。


```yaml
name: Deploy

on: [push, pull_request]

jobs:
  gh-pages-deploy:
    runs-on: ubuntu-latest

    name: Deploying to Github Pages

    steps:·
      - name: Checkout Codes
        uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '15'

      - name: Install packages
        run: yarn install

      - name: Set Creds
        run: |
          git config user.name "curder"
          git config user.email "q.curder@gmail.com"

      - name: Deploy
        run: yarn deploy
```


将 GitHub 用户名和电子邮件地址设置为第25、26行。如果愿意，可以将第 21 行和第 29 行中的命令替换为相应的 npm 命令。

将 `gh-pages-deploy.yml` 文件添加到 `.github/workflows` 文件夹中。
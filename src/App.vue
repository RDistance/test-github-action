<template>
  <div class="blog">
    <article>
      <h1>2026年5月31号</h1>
      <h1>使用 GitHub Actions 自动将 Vue 3 纯前端项目部署到阿里云服务器</h1>
      <p>本文介绍如何通过 GitHub Actions 自动化流程，将 Vue 3 前端项目构建并部署到阿里云服务器。整个方案适用于纯前端项目，不依赖后端服务。</p>

      <h2>一、准备工作</h2>
      <ol>
        <li>在 GitHub 上创建仓库，并将 Vue 3 项目推送到仓库。</li>
        <li>确保项目中已有可运行的 Vue 3 前端应用，且可以通过 <code>npm run build</code> 生成静态文件。</li>
        <li>准备阿里云服务器（ECS），并确认已经配置好 SSH 访问权限。</li>
        <li>在服务器上安装 Nginx 或其他静态静态文件服务器，用于托管构建后生成的静态资源。</li>
      </ol>

      <h2>二、阿里云服务器配置</h2>
      <p>在阿里云 ECS 服务器上完成以下步骤：</p>
      <ul>
        <li>安装 Nginx：<code>sudo apt-get update && sudo apt-get install nginx -y</code></li>
        <li>配置 Nginx 站点，将部署目录指向项目的 <code>dist</code> 文件夹。</li>
        <li>确认服务器可通过 SSH 密钥登录。建议使用私钥进行自动化部署。</li>
      </ul>

      <h2>三、GitHub 仓库中添加 SSH 私钥</h2>
      <p>在 GitHub 仓库设置中，打开 <strong>Settings &gt; Secrets and variables &gt; Actions</strong>，添加以下 Secrets：</p>
      <ul>
        <li><code>SSH_PRIVATE_KEY</code>：部署服务器的私钥内容。</li>
        <li><code>SSH_HOST</code>：阿里云服务器公网 IP 或域名。</li>
        <li><code>SSH_USERNAME</code>：SSH 登录用户名，例如 <code>ubuntu</code> 或 <code>root</code>。</li>
        <li><code>DEPLOY_PATH</code>：服务器上存放静态文件的目标目录，例如 <code>/var/www/vue-app</code>。</li>
      </ul>

      <h2>四、GitHub Actions 工作流配置</h2>
      <p>在项目根目录下创建 <code>.github/workflows/deploy.yml</code>，配置如下：</p>
      <pre><code>name: Deploy Vue 3 to Alibaba Cloud ECS

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v5
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Build project
        run: npm run build

      - name: Setup SSH key
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H "$SSH_HOST" >> ~/.ssh/known_hosts
        env:
          SSH_PRIVATE_KEY: serets.SSH_PRIVATE_KEY
          SSH_HOST: secrets.SSH_HOST

      - name: Deploy to Alibaba Cloud ECS
        run: |
          rsync -avz --delete ./dist/ $SSH_USERNAME@$SSH_HOST:$DEPLOY_PATH/
        env:
          SSH_USERNAME: secrets.SSH_USERNAME
          SSH_HOST: secrets.SSH_HOST
          DEPLOY_PATH: secrets.DEPLOY_PATH
</code></pre>

      <h2>五、工作流说明</h2>
      <ul>
        <li><strong>actions/checkout</strong>：获取仓库代码。</li>
        <li><strong>actions/setup-node</strong>：配置 Node.js 运行环境。</li>
        <li><code>npm install</code>：安装依赖。</li>
        <li><code>npm run build</code>：构建 Vue 3 项目。</li>
        <li><strong>rsync</strong>：将构建后的 <code>dist</code> 文件夹同步到服务器指定目录。</li>
      </ul>

      <h2>六、阿里云服务器 Nginx 简单配置</h2>
      <p>在服务器上的 Nginx 配置文件中，添加类似如下站点配置：</p>
      <pre><code>server {
  listen 80;
  server_name your-domain.com;

  root /var/www/vue-app;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }
}
</code></pre>

      <h2>七、总结</h2>
      <p>通过上述配置，GitHub Actions 可以在每次推送到 main 分支时自动构建 Vue 3 前端项目，并将生成的静态文件部署到阿里云 ECS 服务器。该方案适合纯前端项目，部署过程简单、自动化程度高。</p>
    </article>
  </div>
</template>

<style scoped>
.blog {
  max-width: 860px;
  margin: 2rem auto;
  padding: 0 1rem;
  color: #333;
  font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

h1 {
  font-size: 2rem;
  margin-bottom: 1rem;
}

h2 {
  margin-top: 1.75rem;
  margin-bottom: 0.75rem;
  font-size: 1.25rem;
}

p,
ul,
ol,
pre {
  line-height: 1.8;
  margin: 0.75rem 0;
}

code {
  background: #f4f4f5;
  padding: 0.15rem 0.3rem;
  border-radius: 4px;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}

pre {
  background: #f7f7fb;
  padding: 1rem;
  overflow-x: auto;
  border-radius: 8px;
}
</style>

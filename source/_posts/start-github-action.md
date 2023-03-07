---
title: github action 初体验
date: 2023-03-07 10:13:11
tags: github action page
categories: 实战
thumbnail: https://img1.baidu.com/it/u=3091723246,1718758483&fm=253&fmt=auto&app=138&f=JPEG?w=600&h=328
---
# github action 初体验
之前一直用的很开心的Travis CI不免费了，我这种程序员贫农又得去到处乞讨了，这次找到了大户微软家的github，希望能稳定一段时间的饭碗
## 操作步骤
1. 创建一个名为.github的文件夹
2. 在.github下创建workflows文件夹
3. 在workflows文件夹下创建deploy.yml，用来部署我们的博客
4. 在deploy.yml里写下我们的部署代码
    ```shell
    name: Pages

on:
  push:
    branches:
      - main  # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
    ```
    代码是抄的hexo提供的部署脚本
5. 推送，坐等结果,正常工作，打完收工

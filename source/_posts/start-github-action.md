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
    name: Deploy

    on: [push]

    jobs:
    build:
        runs-on: ubuntu-latest
        name: A job to deploy blog.
        steps:
        - name: Checkout
        uses: actions/checkout@v1
        with:
            submodules: true # Checkout private submodules(themes or something else).
        
        # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
        - name: Cache node modules
        uses: actions/cache@v1
        id: cache
        with:
            path: node_modules
            key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
            restore-keys: |
            ${{ runner.os }}-node-
        - name: Install Dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
        
        # Deploy hexo blog website.
        - name: Deploy
        id: deploy
        uses: sma11black/hexo-action@v1.0.3
        with:
            deploy_key: ${{ secrets.DEPLOY_KEY }}
            user_name: xzjs  # (or delete this input setting to use bot account)
            user_email: xuezhijunshang@icloud.com  # (or delete this input setting to use bot account)
            commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
        # Use the output from the `deploy` step(use for test action)
        - name: Get the output
        run: |
            echo "${{ steps.deploy.outputs.notify }}"
    ```
    代码是抄的hexo提供的部署脚本
5. 推送，坐等结果
## 报错了
![Screen Shot 2023-03-07 at 11.10.51](https://image.baidu.com/search/down?url=http://tvax2.sinaimg.cn/large/9f8a45fbgy1hbr4gwzz4oj213407j0v5.jpg)
因为在themes下有个不用的主题文件夹，导致没有下载到对应的子模块，删除继续
![Screen Shot 2023-03-07 at 11.47.39](https://image.baidu.com/search/down?url=http://tvax3.sinaimg.cn/large/9f8a45fbgy1hbr5gxa78kj21fz07k78n.jpg)
又来一个提醒，还是没有部署成功
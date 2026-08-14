---
title: "批量删除bilibili收到的消息和点赞"
date: 2026-08-14T08:30:00+08:00
draft: false
categories: ["bilibili"]
summary: "批量删除哔哩哔哩收到的会话消息和赞。"
---


## 一、批量删除收到的消息

### 1. 打开消息分组

登录哔哩哔哩，在消息中心点击需要清理的消息分组，让会话列表显示在页面中。

### 2. 打开浏览器控制台

按 `F12` 打开开发者工具，切换到 `Console`（控制台）。

### 3. 执行删除脚本

将下面的代码复制到控制台，按回车执行：

```javascript
(async () => {
    const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

    while (true) {
        const button = document.querySelector(".list-container.ps .close");

        if (!button) {
            console.log("OK！你已删除了所有回复我的消息！");
            break;
        }

        const count = document.querySelectorAll(".list-container.ps .close").length;
        console.log(`当前剩余 ${count} 个消息`);

        button.click();

        await sleep(1200);
    }
})();
```


## 二、批量删除收到的赞

### 1. 打开收到的赞

在哔哩哔哩消息中心进入 `收到的赞` 页面。

### 2. 打开浏览器控制台

按 `F12` 打开开发者工具，切换到 `Console`（控制台）。

### 3. 执行删除脚本

将下面的代码复制到控制台，按回车执行：

```javascript
(async () => {
    const csrf = document.cookie.match(/bili_jct=([^;]+)/)?.[1];

    if (!csrf) {
        console.log("未获取到 csrf，请确认已经登录！");
        return;
    }

    const wait = ms => new Promise(r => setTimeout(r, ms));
    let count = 0;

    while (true) {
        const res = await fetch(
            "https://api.bilibili.com/x/msgfeed/like?platform=web&build=0&mobi_app=web",
            { credentials: "include" }
        ).then(r => r.json());

        const items = res.data?.total?.items || [];

        if (!items.length) {
            console.log(`OK！已删除 ${count} 条收到的赞！`);
            break;
        }

        console.log("本页消息", items.length, "个！");

        for (const item of items) {
            const body = new URLSearchParams({
                tp: "0",
                id: item.id,
                build: "0",
                mobi_app: "web",
                csrf_token: csrf,
                csrf: csrf
            });

            const result = await fetch(
                "https://api.bilibili.com/x/msgfeed/del",
                {
                    method: "POST",
                    credentials: "include",
                    headers: {
                        "Content-Type": "application/x-www-form-urlencoded"
                    },
                    body
                }
            ).then(r => r.json());

            if (result.code === 0) {
                count++;
                console.log(`已删除 ${count} 条`);
            } else {
                console.log("删除失败：", result);
            }

            await wait(1200);
        }
    }
})();
```

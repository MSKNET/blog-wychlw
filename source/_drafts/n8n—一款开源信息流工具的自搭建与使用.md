---
title: n8n—一款开源信息流工具的自搭建与使用
date: 2021-12-13 22:54:23
tags:
categories: note
---

# 什么是n8n
n8n是一款开源的自动工作流处理工具。
（至少它官网是这么写的）
但是在生活中捏，我们一般用它来同步博客/twitter/QQ空间/bilibili……上面发布的内容
（比如，你要是在bilibili或者twitter上面看到了这篇文章的链接，那就是n8n推送过去的
（懒是人类进步的阶梯嘛————

# 为什么用n8n

它免费。
你还需要第二个理由吗！！！（xxx

---

那让我们开始搭建吧~~
[这里是官方的doc](https://docs.n8n.io/getting-started/)

## before

### Docker
当然，你可以把n8n直接搭建在裸机上，或者……还是套层Docker吧，方便以后迁移

以下均以~~某盒装安装介质~~Debian系统为例

那，安装docker的第一步：卸载docker（旧版本啦
`apt-get remove docker docker-engine docker.io containerd runc`

然后安装一点点小工具来添加docker的apt源
`apt-get install ca-certificates curl gnupg lsb-release`

添加源
`curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`

设置stable源
`echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`

然后安装它！
```
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
```

然后就没有然后了。

## install

有了docker，那安装n8n就是一句话的事了
```
docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

这句话会在当前的用户文件夹下产生一个.n8n文件夹来存放数据，并会让应用运行在5678端口上
（然后你就会发现你的5678端口门户大开，但凡这玩意有点鉴权，也不至于一点点鉴权也没有啊x

## usage
这玩意就是将节点妥妥拽拽来完成工作的，应该不难上手
这里着重讲一下twitter的使用吧

### twitter
n8n连接twitter，本质上是你在twitter上创建了一个你自己的app，然后让n8n连接那个app
那么自然，我们要先：
#### 创建twitter app

首先你需要登录[twitter的开发者页面](https://developer.twitter.com/en/portal/projects-and-apps)，并创建一个app
![创建app](https://cdn.jsdelivr.net/gh/wychlw/img@main//img/%E6%89%B9%E6%B3%A8%202021-12-14%20000124.png)
然后选择Development（这里是因为我本身就有一个，所以选择不了）
![](https://cdn.jsdelivr.net/gh/wychlw/img@main//img/20211214000546.png)
取名，然后假如你是李华，你要…………（xxxx
一般来说都会过的啦~~
接下来把key和token填过去就好啦~~

### 一些有用的节点：

#### 当有新feed时：
```

```
（需要先运行一次才会是新的feed）

{
  "nodes": [
    {
      "parameters": {},
      "name": "Start",
      "type": "n8n-nodes-base.start",
      "typeVersion": 1,
      "position": [
        -600,
        340
      ]
    },
    {
      "parameters": {
        "functionCode": "const urls_new = items[0].json.urls;\nconst urls_old = items[1].json.urls;\nconst difference = urls_new.filter(x => !urls_old.includes(x));\n\nreturn [{ json: {\n  \"old\": urls_old,\n  \"new\": urls_new,\n  \"diff\": difference\n} }];"
      },
      "name": "Diff",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [
        140,
        160
      ]
    },
    {
      "parameters": {
        "filePath": "old.json"
      },
      "name": "Read Old",
      "type": "n8n-nodes-base.readBinaryFile",
      "typeVersion": 1,
      "position": [
        -380,
        340
      ],
      "continueOnFail": true
    },
    {
      "parameters": {
        "options": {
          "keepSource": false
        }
      },
      "name": "Data to JSON",
      "type": "n8n-nodes-base.moveBinaryData",
      "typeVersion": 1,
      "position": [
        -200,
        340
      ]
    },
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {}
          ]
        }
      },
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [
        -600,
        160
      ],
      "disabled": true
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$node[\"Diff\"].data[\"diff\"].length}}",
              "operation": "larger"
            }
          ]
        }
      },
      "name": "IF",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [
        700,
        140
      ]
    },
    {
      "parameters": {
        "fileName": "old.json"
      },
      "name": "Clear Old",
      "type": "n8n-nodes-base.writeBinaryFile",
      "typeVersion": 1,
      "position": [
        -220,
        520
      ]
    },
    {
      "parameters": {
        "functionCode": "items[0].json.urls = [];\nreturn items;"
      },
      "name": "Function",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [
        -600,
        520
      ]
    },
    {
      "parameters": {
        "mode": "jsonToBinary",
        "options": {}
      },
      "name": "JSON to Data1",
      "type": "n8n-nodes-base.moveBinaryData",
      "typeVersion": 1,
      "position": [
        -380,
        520
      ]
    },
    {
      "parameters": {
        "url": "https://blog.wcysite.com/atom.xml"
      },
      "name": "RSS Feed Read",
      "type": "n8n-nodes-base.rssFeedRead",
      "typeVersion": 1,
      "position": [
        -380,
        160
      ],
      "executeOnce": false,
      "alwaysOutputData": false
    },
    {
      "parameters": {
        "fileName": "old_blog_rss.json"
      },
      "name": "Write Binary File",
      "type": "n8n-nodes-base.writeBinaryFile",
      "typeVersion": 1,
      "position": [
        -20,
        0
      ]
    },
    {
      "parameters": {
        "mode": "jsonToBinary",
        "options": {}
      },
      "name": "JSON to Data",
      "type": "n8n-nodes-base.moveBinaryData",
      "typeVersion": 1,
      "position": [
        -200,
        0
      ]
    }
  ],
  "connections": {
    "Start": {
      "main": [
        [
          {
            "node": "RSS Feed Read",
            "type": "main",
            "index": 0
          },
          {
            "node": "Read Old",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Diff": {
      "main": [
        [
          {
            "node": "IF",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Read Old": {
      "main": [
        [
          {
            "node": "Data to JSON",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Cron": {
      "main": [
        [
          {
            "node": "RSS Feed Read",
            "type": "main",
            "index": 0
          },
          {
            "node": "Read Old",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Function": {
      "main": [
        [
          {
            "node": "JSON to Data1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "JSON to Data1": {
      "main": [
        [
          {
            "node": "Clear Old",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "RSS Feed Read": {
      "main": [
        [
          {
            "node": "JSON to Data",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "JSON to Data": {
      "main": [
        [
          {
            "node": "Write Binary File",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
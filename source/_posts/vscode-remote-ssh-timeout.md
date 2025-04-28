---
title: VSCode Remote SSH 超时 
date: 2025-04-28 09:36:29
tags: mac
---

VSCode Remote SSH首次连接服务器时可能会出现timeout的报错，原因是远端服务器无法将VSCode的server下载下来，可以通过配置本地配置，搜索`remote.SSH.useLocalServer`，并将其关闭来解决。
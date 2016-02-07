---
layout: post
title:  "Connecting to headless Xvfb session running selenium tests"
date:   2016-02-07
category: "tech gibberish"
issue_id: 4
---

```bash
xvfb-run --listen-tcp --server-num 44 --auth-file /tmp/xvfb.auth -s "-ac -screen 0 1920x1080x24" java -jar selenium.jar
```

```--server-num``` is the option to set what is the "display id" which is used in next step.


x11vnc -storepasswd shortest.vnc /tmp/vncpass
x11vnc -rfbport 4544 -rfbauth /tmp/vncpass -display :44 -forever -auth /tmp/xvfb.auth

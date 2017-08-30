---
layout: 工具
title: Python操作VPN
date: 2017-08-30 15:15:39
tags: python
---

**文/大大大大峰哥**

因为工作的需求需要不停的更换IP,但是介于IP池的价钱相对很高,所以买的是VPN,这里就要自己实现VPN登录与断开.

## 1. 环境

Win7,Pycharm

## 2. 步骤

1. 进入IE中Intent选项.
2. 进入连接.
3. 添加VPN.
4. Internet填VPN中的地址.
5. 目标名称需要记住,到时候通过Python操作的时候,需要填写进去.
6. 然后下一步,直接创建.
7. 然后就可以通过下面这个类来控制VPN的断开与连接,从而可以起到更换IP的功能.

## 3. 代码

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2017/8/30 13:49
# @Author  : TangZhiFeng
# @File    : VPN.py
# @Software: PyCharm Community Edition

import os


class VPNHelper(object):
    def __init__(self, _vpnIP, _userName, _passWord, WinDir=r"C:\windows\system32", RasDialFileName=r'\rasdial.exe'):
        self.IPToPing = _vpnIP
        self._VPNName = _vpnIP;
        self._UserName = _userName;
        self._PassWord = _passWord;
        self._WinDir = WinDir
        self._RasDialFileName = RasDialFileName
        self._VPNPROCESS = self._WinDir + self._RasDialFileName

    def connectVPN(self):
        try:
            command = self._VPNName + " " + self._UserName + " " + self._PassWord
            os.system(self._VPNPROCESS + " " + command)
        except:
            print("VPN连接失败!")

    def disConnectVPN(self):
        try:
            command = self._VPNName + " /d"
            os.system(self._VPNPROCESS + " " + command)
        except:
            print("VPN断开失败!")

    def Restart(self, waitingTime=0):
        import time
        self.connectVPN()
        time.sleep(waitingTime)
        self.disConnectVPN()


if __name__ == "__main__":
    vpn = VPNHelper("VPN设置的名称", "账号", "密码")
    vpn.Restart()
```

## 4. 可以改进的地方

1. 没有实现全自动,主要是不知道怎么通过代码的方式直接创建VPN,这个问题如果有大佬知道,可以发邮件给我,谢谢.
2. 这个程序可以封装成为一个小工具使用.
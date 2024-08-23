---
title: macOS 15.0开启信任任何来源的应用程序
date: 2024-08-23 09:36:29
tags: mac
---

在macOS 15.0中，我们已经不能像之前那样使用如下命令开启信任任何来源的应用程序。

```shell
sudo spctl --master-disable
```

需要自己编辑一个描述文件来进行开启，描述文件为如下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>PayloadContent</key>
    <array>
      <dict>
        <key>PayloadType</key>
        <string>com.apple.systempolicy.control</string>
        <key>PayloadUUID</key>
        <string>uuid-1</string>
        <key>PayloadIdentifier</key>
        <string>com.yourcompany.profile.systempolicy</string>
        <key>PayloadDisplayName</key>
        <string>System Policy Control</string>
        <key>PayloadDescription</key>
        <string>Configures System Policy Control settings</string>
        <key>PayloadVersion</key>
        <integer>1</integer>
        <key>EnableAssessment</key>
        <false />
      </dict>
    </array>
    <key>PayloadDisplayName</key>
    <string>Disable Gatekeeper</string>
    <key>PayloadIdentifier</key>
    <string>com.yourcompany.profile</string>
    <key>PayloadRemovalDisallowed</key>
    <false />
    <key>PayloadScope</key>
    <string>System</string>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadUUID</key>
    <string>uuid-2</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
  </dict>
</plist>

```

其中uui'd-1和uuid-2需要自行使用`uuidgen`命令生成，不一样即可。之后将如上文件保存为`disable.mobileconfig`文件，在Finder中双击运行安装该描述文件即可。



完成以上步骤后就可以在设置-隐私与安全性最下面的安全性中看到已经开启了“任何来源”的应用。

之后安装一些破解软件时还是会遇到已损坏无法打开的报错，可以使用如下命令添加签名。

```shell
sudo xattr -d com.apple.quarantine /Applications/xxxx.app
```



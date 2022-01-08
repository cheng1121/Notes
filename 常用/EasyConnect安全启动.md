# EasyConnect安全启动

关于 EasyConnect，懂的都懂，不用多说。本文介绍如何相对安全的在 macOS 上使用 EasyConnect。

EasyConnect 主要有以下行为：

1. 后台常驻进程 EasyMonitor 和 ECAgentProxy，且都以 root 权限运行；
2. 自动安装根证书，包括系统根证书与 FireFox 的根证书，且在删除后会重新安装。

以 root 权限常驻进程，意味着这些进程可以读取和写入任何东西；安装根证书，意味着可以直接进行中间人攻击。因此我们需要解决这些问题。

## 步骤

1. 使用 `sudo su`
2. 删除 /Library/LaunchDaemons/com.sangfor.EasyMonitor.plist
3. 删除 /Library/LaunchAgents/com.sangfor.ECAgentProxy.plist
4. 重新启动 Mac
5. 打开钥匙串，删除系统钥匙串-系统-证书下面的证书（关键词 sangfor）

注意一定要**先重启 Mac 再删除证书**，因为在后台的 EasyMonitor 进程会在证书被删除后自动导入。

这样就消除了 EasyConnect 的问题。但是，目前为止会导致 EasyConnect 不能连接，因此我们需要用安全的方式启动 EasyConnect。可以编写两个脚本实现启动和退出。

## 脚本

### `startEasyConnect`

```bash
#! /bin/bash

/Applications/EasyConnect.app/Contents/Resources/bin/EasyMonitor > /dev/null 2>&1 &
/Applications/EasyConnect.app/Contents/MacOS/EasyConnect > /dev/null 2>&1 &
```

### `killEasyMonitor`

```bash
#! /bin/bash

pkill EasyMonitor
pkill ECAgent
pkill ECAgentProxy
```

将上述两个脚本放到 `PATH` 路径下，并给予可执行权限：

```bash
chmod +x startEasyConnect
chmod +x killEasyMonitor
```

在需要使用 EasyConnect 时，使用普通用户身份执行 `startEasyConnect` 命令（如此，EasyMonitor 进程就没有权限导入根证书了）。在退出 EasyConnect 后，执行 `killEasyMonitor` 杀掉后台的 EasyMonitor 进程即可。
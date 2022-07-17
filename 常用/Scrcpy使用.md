

# Scrcpy的使用

scrcpy可以显示并控制通过USB（或TCP/IP）连接的安卓设备，且**不需要任何root权限**,支持GNU/Linux,Windows和macOS。

github地址：[scrcpy](https://github.com/Genymobile/scrcpy)

[中文文档](https://github.com/Genymobile/scrcpy/blob/master/README.zh-Hans.md)

 

必要条件：

- Android 系统最低版本为5.0，即Android API 21
- 需要开启USB 调试
- 个别设备需要开启`usb debugging(Security settings)`,用来使用键盘和鼠标等进行控制

## 一、下载

### 1. mac上下载`scrcpy`

- 终端执行`brew install scrcpy`命令安装

- `.bash_profile`文件中配置`adb`的环境变量

  - 安装有`Android SDK`的话，可以直接添加如下系统环境变量

    ```shell
    #aapt home
    export AAPT_HOME=${ANDROID_HOME}/build-tools/29.0.2
    export PATH=${PATH}:${AAPT_HOME}
    ```

  - 使用`homebrew`下载`android-platform-tools`

    ```shell
    # Homebrew >= 2.6.0
    brew install --cask android-platform-tools
    
    # Homebrew < 2.6.0
    brew cask install android-platform-tools
    ```

    

## 二、使用

终端执行命令`scrcpy` 即可连接`Android`设备;执行`scrcpy --help`即可查看其它命令

- 修改快捷键为左侧control并连接设备`scrcpy --shortcut-mod=lctrl`
- 使用`&`分配进程，让其不影响当前终端的使用 `scrcpy --shortcut-mod=lctrl &`


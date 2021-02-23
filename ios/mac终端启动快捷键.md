# 设置终端启动快捷键

## 使用系统中的自动操作App

- launcher中找到自动操作并打开

- 选择文稿类型-快捷操作(Quick Action)

- 双击或者拖动Run AppleScript（运行Apple脚本）到右侧

- 输入如下代码：

  ```shell
  on run {input, parameters}
      
      (* Your script goes here *)
      
      tell application "Terminal"
      reopen
      activate
      end tell
  end run
  ```

- `command + s`保存问`open terminal `

- 进入系统设置打开键盘

- 选择快捷键(Shortcuts) -> 服务(Services) -> 找到`open terminal`设置快捷键

  ps: 设置无效可能是和其他App的快捷键冲突导致

## 已设置的快捷键

- 打开终端command +` (按键1左侧的按键) 
- 打开Android Studio： `control+ 1` 
- 打开Vsual Studio Code ：`control + 2`
- 打开Xcode：`control + 3`


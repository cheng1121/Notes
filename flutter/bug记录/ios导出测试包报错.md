# ios导出测试包报错
```
The data couldn’t be read because it isn’t in the correct format.

```
- 点击show log进入IDEDistribution.standard.log，可以看到如下错误
```
/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- cfpropertylist (LoadError)
    from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    from /Applications/Xcode.app/Contents/Developer/usr/bin/ipatool:15:in `<main>'
2020-02-05 07:57:11 +0000  /Applications/Xcode.app/Contents/Developer/usr/bin/ipatool exited with 1
2020-02-05 07:57:11 +0000  ipatool JSON: (null)

```

## 解决办法
- 运行如下命令
```
gem install CFPropertyList
gem install sqlite3
```
然后重启系统

tips: 提示权限错误时使用sudo执行
```
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.

```

[点击此查看stackoverflow原文](https://stackoverflow.com/questions/60071471/export-ipa-file-fails)


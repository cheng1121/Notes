# M1 CocoaPods Error

## 1. 错误说明

换了M1版MBP后，运行旧项目刀iOS模拟器时报错：

```shell
Launching lib/main.dart on iPhone 13 in debug mode...
CocoaPods' output:
↳
      Preparing
    Analyzing dependencies
    Inspecting targets to integrate
      Using `ARCHS` setting to build architectures of target `Pods-Runner`: (``)
    Finding Podfile changes
      - Flutter
      - flutter_jim
    Fetching external sources
    -> Fetching podspec for `Flutter` from `Flutter`
    -> Fetching podspec for `flutter_jim` from `.symlinks/plugins/flutter_jim/ios`
    Resolving dependencies of `Podfile`
Error output from CocoaPods:
↳
    /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/universal-darwin21/rbconfig.rb:230: warning: Insecure world writable dir /opt/homebrew/bin in PATH, mode 040777
    [!] Automatically assigning platform `iOS` with version `9.0` on target `Runner` because no platform was specified. Please specify a platform for this target in your Podfile. See `https://guides.cocoapods.org/syntax/podfile.html#platform`.
    /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': dlopen(/Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle, 0x0009): tried: '/Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle' (mach-o file, but is an incompatible architecture (have 'arm64', need 'x86_64')), '/usr/lib/ffi_c.bundle' (no such file) - /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle (LoadError)
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:5:in `rescue in <top (required)>'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ethon-0.15.0/lib/ethon.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/typhoeus-1.4.0/lib/typhoeus.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:440:in `download_typhoeus_impl_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:372:in `download_and_save_with_retries_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:365:in `download_file_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:338:in `download_file'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:53:in `refresh_metadata'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source.rb:31:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:30:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `new'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `block in source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:322:in `source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `block in aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:26:in `aggregate'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:60:in `all'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:173:in `repo_information'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:77:in `stack'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:24:in `report'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:66:in `report_error'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:396:in `handle_exception'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:337:in `rescue in run'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:324:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/bin/pod:55:in `<top (required)>'
    	from /usr/local/bin/pod:23:in `load'
    	from /usr/local/bin/pod:23:in `<main>'
    /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- 2.6/ffi_c (LoadError)
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ethon-0.15.0/lib/ethon.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/typhoeus-1.4.0/lib/typhoeus.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:440:in `download_typhoeus_impl_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:372:in `download_and_save_with_retries_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:365:in `download_file_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:338:in `download_file'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:53:in `refresh_metadata'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source.rb:31:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:30:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `new'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `block in source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:322:in `source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `block in aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:26:in `aggregate'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:60:in `all'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:173:in `repo_information'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:77:in `stack'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface/error_report.rb:24:in `report'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:66:in `report_error'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:396:in `handle_exception'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:337:in `rescue in run'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:324:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/bin/pod:55:in `<top (required)>'
    	from /usr/local/bin/pod:23:in `load'
    	from /usr/local/bin/pod:23:in `<main>'
    /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': dlopen(/Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle, 0x0009): tried: '/Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle' (mach-o file, but is an incompatible architecture (have 'arm64', need 'x86_64')), '/usr/lib/ffi_c.bundle' (no such file) - /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi_c.bundle (LoadError)
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:5:in `rescue in <top (required)>'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ethon-0.15.0/lib/ethon.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/typhoeus-1.4.0/lib/typhoeus.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:440:in `download_typhoeus_impl_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:372:in `download_and_save_with_retries_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:365:in `download_file_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:338:in `download_file'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:53:in `refresh_metadata'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source.rb:31:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:30:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `new'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `block in source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:322:in `source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `block in aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:26:in `aggregate'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:60:in `all'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:393:in `source_with_url'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/sources_manager.rb:22:in `find_or_create_source_with_url'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:178:in `block in sources'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:177:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:177:in `sources'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:1077:in `block in resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface.rb:64:in `section'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:1076:in `resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:124:in `analyze'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:416:in `analyze'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:241:in `block in resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface.rb:64:in `section'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:240:in `resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:161:in `install!'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command/install.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:334:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/bin/pod:55:in `<top (required)>'
    	from /usr/local/bin/pod:23:in `load'
    	from /usr/local/bin/pod:23:in `<main>'
    /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': cannot load such file -- 2.6/ffi_c (LoadError)
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ffi-1.15.4/lib/ffi.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/ethon-0.15.0/lib/ethon.rb:3:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/typhoeus-1.4.0/lib/typhoeus.rb:2:in `<top (required)>'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:440:in `download_typhoeus_impl_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:372:in `download_and_save_with_retries_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:365:in `download_file_async'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:338:in `download_file'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:53:in `refresh_metadata'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source.rb:31:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/cdn_source.rb:30:in `initialize'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `new'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:315:in `block in source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:322:in `source_from_path'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `block in aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:331:in `aggregate_with_repos'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:26:in `aggregate'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:60:in `all'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-core-1.11.2/lib/cocoapods-core/source/manager.rb:393:in `source_with_url'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/sources_manager.rb:22:in `find_or_create_source_with_url'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:178:in `block in sources'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:177:in `map'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:177:in `sources'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:1077:in `block in resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface.rb:64:in `section'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:1076:in `resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer/analyzer.rb:124:in `analyze'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:416:in `analyze'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:241:in `block in resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/user_interface.rb:64:in `section'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:240:in `resolve_dependencies'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/installer.rb:161:in `install!'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command/install.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/claide-1.0.3/lib/claide/command.rb:334:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/lib/cocoapods/command.rb:52:in `run'
    	from /Library/Ruby/Gems/2.6.0/gems/cocoapods-1.11.2/bin/pod:55:in `<top (required)>'
    	from /usr/local/bin/pod:23:in `load'
    	from /usr/local/bin/pod:23:in `<main>'
Error running pod install
Error launching application on iPhone 13.
Exited (sigterm)

```



## 2. StackOverFlow上的解决办法

- 运行`gem list --local | grep cocoapods` 查看需要卸载的cocoapods组件

- 卸载以下几个：

  ```shell
  sudo gem uninstall cocoapods
  sudo gem uninstall cocoapods-core
  sudo gem uninstall cocoapods-downloader
  sudo gem uninstall cocoapods-deintegrate
  
  ```

- 执行`sudo arch -x86_64 gem install ffi`

- 执行`sudo arch -x86_64 gem install cocoapods`
# init进程启动
init进程做的工作比较多，主要有创建和挂载文件系统、初始化和启动属性服务和启动Zygote进程。
#### main.cpp
在8.1系统之后init进程启动会首先执行main.cpp中main函数
```
int main(int argc, char** argv) {

#if __has_feature(address_sanitizer)
    __asan_set_error_report_callback(AsanReportCallback);
#endif
     /**
     *  strcmpd对比两个字符串是否相同 相同返回0
     *  ueventd主要是负责设备节点的创建、权限设定等一些列工作
     *  basename是C库中的一个函数，得到特定的路径中的最后一个'/'后面的内容，
     *  比如/sdcard/miui_recovery/backup，得到的结果是backup
     */

    if (!strcmp(basename(argv[0]), "ueventd")) {
        return ueventd_main(argc, argv);
    }

    if (argc > 1) {
        if (!strcmp(argv[1], "subcontext")) {
            android::base::InitLogging(argv, &android::base::KernelLogger);
            const BuiltinFunctionMap function_map;

            return SubcontextMain(argc, argv, &function_map);
        }

        if (!strcmp(argv[1], "selinux_setup")) {
            return SetupSelinux(argv);
        }
        //执行第二部分
        if (!strcmp(argv[1], "second_stage")) {
            return SecondStageMain(argc, argv);
        }
    }
    //该main方法会执行两次，第一次执行第一部分
    //执行第一部分
    return FirstStageMain(argc, argv);
}
```

接下来看下第一阶段主要做了哪些事

#### 第一阶段代码在first_stage_init.cpp类中

```
int FirstStageMain(int argc, char** argv) {
    if (REBOOT_BOOTLOADER_ON_PANIC) {
        InstallRebootSignalHandlers();
    }

    boot_clock::time_point start_time = boot_clock::now();

    std::vector<std::pair<std::string, int>> errors;
#define CHECKCALL(x) \
    if (x != 0) errors.emplace_back(#x " failed", errno);

    // Clear the umask.
    //清理umask
    umask(0);
    //创建和挂载启动所需的文件目录
    CHECKCALL(clearenv());
    CHECKCALL(setenv("PATH", _PATH_DEFPATH, 1));
    //创建目录
    CHECKCALL(mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755"));
    CHECKCALL(mkdir("/dev/pts", 0755));
    CHECKCALL(mkdir("/dev/socket", 0755));
    CHECKCALL(mount("devpts", "/dev/pts", "devpts", 0, NULL));
#define MAKE_STR(x) __STRING(x)
    CHECKCALL(mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC)));
    // Don't expose the raw commandline to unprivileged processes.
    CHECKCALL(chmod("/proc/cmdline", 0440));
    gid_t groups[] = {AID_READPROC};
    CHECKCALL(setgroups(arraysize(groups), groups));
    CHECKCALL(mount("sysfs", "/sys", "sysfs", 0, NULL));
    CHECKCALL(mount("selinuxfs", "/sys/fs/selinux", "selinuxfs", 0, NULL));

    CHECKCALL(mknod("/dev/kmsg", S_IFCHR | 0600, makedev(1, 11)));

    if constexpr (WORLD_WRITABLE_KMSG) {
        CHECKCALL(mknod("/dev/kmsg_debug", S_IFCHR | 0622, makedev(1, 11)));
    }

    CHECKCALL(mknod("/dev/random", S_IFCHR | 0666, makedev(1, 8)));
    CHECKCALL(mknod("/dev/urandom", S_IFCHR | 0666, makedev(1, 9)));

    CHECKCALL(mknod("/dev/ptmx", S_IFCHR | 0666, makedev(5, 2)));
    CHECKCALL(mknod("/dev/null", S_IFCHR | 0666, makedev(1, 3)));
    CHECKCALL(mount("tmpfs", "/mnt", "tmpfs", MS_NOEXEC | MS_NOSUID | MS_NODEV,
                    "mode=0755,uid=0,gid=1000"));
    CHECKCALL(mkdir("/mnt/vendor", 0755));
    CHECKCALL(mkdir("/mnt/product", 0755));

    CHECKCALL(mount("tmpfs", "/apex", "tmpfs", MS_NOEXEC | MS_NOSUID | MS_NODEV,
                    "mode=0755,uid=0,gid=0"));

    CHECKCALL(mount("tmpfs", "/debug_ramdisk", "tmpfs", MS_NOEXEC | MS_NOSUID | MS_NODEV,
                    "mode=0755,uid=0,gid=0"));
#undef CHECKCALL

    // 我们需要为从第一阶段init派生的子进程设置stdin / stdout / stderr作为挂载过程的一部分。 如果内核先前已打开它，这将关闭/ dev / console。
    auto reboot_bootloader = [](const char*) { RebootSystem(ANDROID_RB_RESTART2, "bootloader"); };
    //初始化Kernel的Log，这样就可以从外界获取Kernel的日志
    InitKernelLogging(argv, reboot_bootloader);

    if (!errors.empty()) {
        for (const auto& [error_string, error_errno] : errors) {
            LOG(ERROR) << error_string << " " << strerror(error_errno);
        }
        LOG(FATAL) << "Init encountered errors starting first stage, aborting";
    }

    LOG(INFO) << "init first stage started!";

    auto old_root_dir = std::unique_ptr<DIR, decltype(&closedir)>{opendir("/"), closedir};
    if (!old_root_dir) {
        PLOG(ERROR) << "Could not opendir(\"/\"), not freeing ramdisk";
    }

    struct stat old_root_info;
    if (stat("/", &old_root_info) != 0) {
        PLOG(ERROR) << "Could not stat(\"/\"), not freeing ramdisk";
        old_root_dir.reset();
    }

    if (ForceNormalBoot()) {
        mkdir("/first_stage_ramdisk", 0755);
        // SwitchRoot() must be called with a mount point as the target, so we bind mount the
        // target directory to itself here.
        if (mount("/first_stage_ramdisk", "/first_stage_ramdisk", nullptr, MS_BIND, nullptr) != 0) {
            LOG(FATAL) << "Could not bind mount /first_stage_ramdisk to itself";
        }
        SwitchRoot("/first_stage_ramdisk");
    }

    // If this file is present, the second-stage init will use a userdebug sepolicy
    // and load adb_debug.prop to allow adb root, if the device is unlocked.
    if (access("/force_debuggable", F_OK) == 0) {
        std::error_code ec;  // to invoke the overloaded copy_file() that won't throw.
        if (!fs::copy_file("/adb_debug.prop", kDebugRamdiskProp, ec) ||
            !fs::copy_file("/userdebug_plat_sepolicy.cil", kDebugRamdiskSEPolicy, ec)) {
            LOG(ERROR) << "Failed to setup debug ramdisk";
        } else {
            // setenv for second-stage init to read above kDebugRamdisk* files.
            setenv("INIT_FORCE_DEBUGGABLE", "true", 1);
        }
    }

    if (!DoFirstStageMount()) {
        LOG(FATAL) << "Failed to mount required partitions early ...";
    }

    struct stat new_root_info;
    if (stat("/", &new_root_info) != 0) {
        PLOG(ERROR) << "Could not stat(\"/\"), not freeing ramdisk";
        old_root_dir.reset();
    }

    if (old_root_dir && old_root_info.st_dev != new_root_info.st_dev) {
        FreeRamdisk(old_root_dir.get(), old_root_info.st_dev);
    }

    SetInitAvbVersionInRecovery();

    setenv(kEnvFirstStageStartedAt, std::to_string(start_time.time_since_epoch().count()).c_str(),
           1);

    const char* path = "/system/bin/init";
    const char* args[] = {path, "selinux_setup", nullptr};
    execv(path, const_cast<char**>(args)); //重新执行main方法进入第二阶段

    // execv() only returns if an error happened, in which case we
    // panic and never fall through this conditional.
    PLOG(FATAL) << "execv(\"" << path << "\") failed";

    return 1;
}
```
init进程第一阶段做的主要工作是挂载分区,创建设备节点和一些关键目录,初始化日志输出系统,启用SELinux安全策略

可以看到第一阶段只是创建和挂载了文件系统，然后就通过执行execv()函数重新执行main函数进入第二阶段。

Android系统分别挂载了以下几种文件系统
- tmpfs
  - tmpfs是一种虚拟内存文件系统，他会将所有的文件存储在虚拟内存中，如果你将tmpfs文件系统卸载后，那么其下的所有的内容将不复存在
  - 既可以使用RAM，也可以使用交换分区，会根据实际需要而改变大小
  - 速度快，因为是驻留在RAM中的，即使使用了交换分区，性能也非常卓越
  - 由于驻留RAM中，所以内容是不持久的。断电后tmpfs的内容就消失了

- devpts
  - 为伪终端提供了一个标准接口，它的标准接入点是/dev/pts
  - 只要pty主复合设备/dev/ptmx被打开，就会在/dev/pts下动态创建一个新的pty设备文件

- proc
  - 是一个非常重要的虚拟文件系统，它可以看作是内核内部数据结构的接口，通过它我们可以获得系统的信息，同时也能够在运行时修改特定的内核参数

- sysfs
  - 与proc文件系统类似，sysfs文件系统也是一个不占有任何磁盘空间的虚拟文件系统
  - 被挂接在/sys目录下。sysfs文件系统是Linux2.6内核引入的，它把连接在系统上的设备和总线组织成为一个分级的文件，使得他们可以在用户空间存取

- selinuxfs
  - 也是虚拟文件系统，通常挂载在/sys/fs/selinux目录下，用来存放SELinux安全策略


#### 第二阶段在init.cpp中
```
//init进程执行第二阶段
int SecondStageMain(int argc, char** argv) {
     ...
     
    //初始化属性服务
    property_init(); //1

    ...
    
    //创建epoll句柄
    Epoll epoll;
    if (auto result = epoll.Open(); !result) {
        PLOG(FATAL) << result.error();
    }
     //用于设置子进程信号处理函数，如果子进程(Zygote进程)异常退出，init进程会调用该函数中设定的信号处理函数来进行处理
    InstallSignalFdHandler(&epoll); //2
    //导入默认的环境变量
    property_load_boot_defaults(load_debug_prop);
    UmountDebugRamdisk();
    fs_mgr_vendor_overlay_mount_all();
    export_oem_lock_status();
    //启动属性服务
    StartPropertyService(&epoll); //3
    MountHandler mount_handler(&epoll);
    set_usb_controller();

    const BuiltinFunctionMap function_map;
    Action::set_function_map(&function_map);

    if (!SetupMountNamespaces()) {
        PLOG(FATAL) << "SetupMountNamespaces failed";
    }

    subcontexts = InitializeSubcontexts();

    ActionManager& am = ActionManager::GetInstance();
    ServiceList& sm = ServiceList::GetInstance();
    //加载启动脚本(其中会解析init.rx配置文件)
    LoadBootScripts(am, sm); //4

    ...
    
    while (true) {
        ...
        
        if (!(waiting_for_prop || Service::is_exec_service_running())) {
           //内部遍历执行每个action中携带的command对应的执行函数
            am.ExecuteOneCommand();
        }
        if (!(waiting_for_prop || Service::is_exec_service_running())) {
            if (!shutting_down) {
                //重启死去的进程
                auto next_process_action_time = HandleProcessActions();//5

                if (next_process_action_time) {
                    epoll_timeout = std::chrono::ceil<std::chrono::milliseconds>(
                            *next_process_action_time - boot_clock::now());
                    if (*epoll_timeout < 0ms) epoll_timeout = 0ms;
                }
            }

            if (am.HasMoreCommands()) epoll_timeout = 0ms;
        }

        if (auto result = epoll.Wait(epoll_timeout); !result) {
            LOG(ERROR) << result.error();
        }
    }

    return 0;
}
```

注释1处调用了property_init函数来对属性进行初始化，并在注释3处调用StartPropertyService来启动属性服务。它们定义在property_service.cpp文件中

注释2处设置子进程信号处理函数，主要用于防止init进程的子进程成为僵尸进程，为了防止僵尸进程的出现，系统会在子进程暂停和终止的时候发出SIGCHLD信号

注释4处解析init.rx文件

注释5处重启Zygote进程

##### 僵尸进程与危害
在UNIX/Linux中，父进程使用fork创建子进程，在子进程终止之后，如果父进程并不知道子进程已经终止了，这时子进程虽然已经退出了，但是在系统进程表中还为它保留了一定的信息(比如进程号、退出状态、运行时间等)，这个子进程就被称作僵尸进程。系统进程表是一项有限资源，如果系统进程表被僵尸进程耗尽的话，系统就可能无法创建新的进程了

#### 解析init.rc
init.rc是一个非常重要的配置文件，它是由Android初始化语言(Android Init Language)编写的脚本，这种语言主要包含5中类型语句：Action、Command、Service、Option和Import

on init和on boot是Action类型语句，格式如下：
```
on <trigger> [&& <trigger>]* //设置触发器
   <command>
   <command>  //动作触发后要执行的命令
```

service类型的语句格式：
```
service <name> <pathname> [ <argument> ]* //<service的名字><执行程序路径><传递参数>
   <option> 
   <option> //option是service的修饰词，影响什么时候、如何启动Service

```

下面看Zygote启动脚本在init.zygote64.rc中定义。该文件存在于/system/core/rootdir

```
   service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    socket blastula_pool stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks

```
Service用于通知init进程创建名为zygote的进程，这个进程执行程序的路径为/system/bin/app_process64,后边为传递的参数-Xzygote /system/bin --zygote。

class main指的是zygote的classname为main

在init.rc中有如下配置代码：
```
on nonencrypted
    class_start main
    class_start late_start
```

class_start main 表示启动classname为main的Service，也就是用来启动Zygote的

class_start是一个COMMAND，对应的函数为do_class_start函数,位置在/system/core/init/builtins.cpp文件中
```
static Result<Success> class_start(const std::string& class_name, bool post_data_only) {
    // Do not start a class if it has a property persist.dont_start_class.CLASS set to 1.
    if (android::base::GetBoolProperty("persist.init.dont_start_class." + class_name, false))
        return Success();
     // Starting a class does not start services which are explicitly disabled.
     // They must  be started individually.
     //启动类不会启动明确禁用的服务。
     //必须单独启动它们。
     // 遍历service列表，找到classname为main的Zygote，并启动它
    for (const auto& service : ServiceList::GetInstance()) {
        if (service->classnames().count(class_name)) {
            if (post_data_only && !service->is_post_data()) {
                continue;
            }
            if (auto result = service->StartIfNotDisabled(); !result) {
                LOG(ERROR) << "Could not start service '" << service->name()
                           << "' as part of class '" << class_name << "': " << result.error();
            }
        }
    }
    return Success();
}

static Result<Success> do_class_start(const BuiltinArguments& args) {
    return class_start(args[1], false /* post_data_only */);
}

```
在Service列表中查找指定名字的zygote，如果找到则调用StartIfNotDisabled函数启动它。
StartIfNotDisabled函数位于/system/core/init/service.cpp

```
Result<Success> Service::StartIfNotDisabled() {
    if (!(flags_ & SVC_DISABLED)) {
        return Start();
    } else {
        flags_ |= SVC_DISABLED_START;
    }
    return Success();
}
```
很简单，如果未设置disabled选项，则会调用start函数
```
Result<Success> Service::Start() {
    if (is_updatable() && !ServiceList::GetInstance().IsServicesUpdated()) {
        ServiceList::GetInstance().DelayService(*this);
        return Error() << "Cannot start an updatable service '" << name_
                       << "' before configs from APEXes are all loaded. "
                       << "Queued for execution.";
    }

    bool disabled = (flags_ & (SVC_DISABLED | SVC_RESET));
    flags_ &= (~(SVC_DISABLED|SVC_RESTARTING|SVC_RESET|SVC_RESTART|SVC_DISABLED_START));

    //如果service已经运行，则不启动
    if (flags_ & SVC_RUNNING) {
        if ((flags_ & SVC_ONESHOT) && disabled) {
            flags_ |= SVC_RESTART;
        }
        
        return Success();
    }

    bool needs_console = (flags_ & SVC_CONSOLE);
    if (needs_console) {
        if (console_.empty()) {
            console_ = default_console;
        }


        int console_fd = open(console_.c_str(), O_RDWR | O_CLOEXEC);
        if (console_fd < 0) {
            flags_ |= SVC_DISABLED;
            return ErrnoError() << "Couldn't open console '" << console_ << "'";
        }
        close(console_fd);
    }
    //判断需要启动的Service的对应的执行文件是否存在，不存在则不启动该Service
    struct stat sb;
    if (stat(args_[0].c_str(), &sb) == -1) {
        flags_ |= SVC_DISABLED;
        return ErrnoError() << "Cannot find '" << args_[0] << "'";
    }

    std::string scon;
    if (!seclabel_.empty()) {
        scon = seclabel_;
    } else {
        auto result = ComputeContextFromExecutable(args_[0]);
        if (!result) {
            return result.error();
        }
        scon = *result;
    }

    if (!IsRuntimeApexReady() && !pre_apexd_) {
        pre_apexd_ = true;
    }

    post_data_ = ServiceList::GetInstance().IsPostData();

    LOG(INFO) << "starting service '" << name_ << "'...";
    //通过clone或者fork()来创建子进程
    pid_t pid = -1;
    if (namespace_flags_) {
        pid = clone(nullptr, nullptr, namespace_flags_ | SIGCHLD, nullptr);
    } else {
        pid = fork();
    }
    //以下代码逻辑运行在子进程中
    if (pid == 0) {
        umask(077);

        ...
        //内部调用Execv函数执行
        if (!ExpandArgsAndExecv(args_, sigstop_)) {
            PLOG(ERROR) << "cannot execve('" << args_[0] << "')";
        }

        _exit(127);
    }

    ...

    NotifyStateChange("running");
    return Success();
}
```
上方代码会clone或者fork出一个service，如果这个service是zygote，那么程序将会进入app_main.cpp的main函数,app_main.cpp源文件位于/frameworks/base/cmds/app_process/app_main.cpp




#### 属性服务
Windows平台上有一个注册表管理器，注册表的内容采用键值对的形式来记录用户、软件的一些使用信息。即使系统或者软件重启，其还是能够根据之前注册表中的记录，进行相应的初始化工作。Android也有类似的机制，即属性服务

init进程启动时的第一个阶段是创建和挂载文件系统；第二个阶段是初始化并启动属性服务、解析init.rc配置文件并启动zygote进程。

init进程通过以下两个函数来启动属性服务
```
//初始化属性服务
property_init();
//启动属性服务
StartPropertyService(&epoll);
```

这两个函数所在位置为/system/core/init/property_service.cpp，首先来看下property_init函数
```
void property_init() {
    selinux_callback cb;
    cb.func_audit = PropertyAuditCallback;
    selinux_set_callback(SELINUX_CB_AUDIT, cb);

    mkdir("/dev/__properties__", S_IRWXU | S_IXGRP | S_IXOTH);
    CreateSerializedPropertyInfo();
    if (__system_property_area_init()) {
        LOG(FATAL) << "Failed to initialize property area";
    }
    if (!property_info_area.LoadDefaultPath()) {
        LOG(FATAL) << "Failed to load serialized property info file";
    }
}
```

```
void StartPropertyService(Epoll* epoll) {
    property_set("ro.property_service.version", "2");

    property_set_fd = CreateSocket(PROP_SERVICE_NAME, SOCK_STREAM | SOCK_CLOEXEC | SOCK_NONBLOCK,
                                   false, 0666, 0, 0, nullptr);
    if (property_set_fd == -1) {
        PLOG(FATAL) << "start_property_service socket creation failed";
    }

    listen(property_set_fd, 8);

    if (auto result = epoll->RegisterHandler(property_set_fd, handle_property_set_fd); !result) {
        PLOG(FATAL) << result.error();
    }
}
```

1. 创建非阻塞的Socket
2. 对property_set_fd设置监听，Socket就变成了server，也就是属性服务，最多可同时为8个用户设置属性服务
3. 使用epoll监听property_set_fd是否新数据，有的话调用handle_property_set_fd来处理数据


```
static void handle_property_set_fd() {
    static constexpr uint32_t kDefaultSocketTimeout = 2000; /* ms */

   ...

    switch (cmd) {
    case PROP_MSG_SETPROP: {
        char prop_name[PROP_NAME_MAX];
        char prop_value[PROP_VALUE_MAX];

        if (!socket.RecvChars(prop_name, PROP_NAME_MAX, &timeout_ms) ||
            !socket.RecvChars(prop_value, PROP_VALUE_MAX, &timeout_ms)) {
          PLOG(ERROR) << "sys_prop(PROP_MSG_SETPROP): error while reading name/value from the socket";
          return;
        }

        prop_name[PROP_NAME_MAX-1] = 0;
        prop_value[PROP_VALUE_MAX-1] = 0;

        std::string source_context;
        //如果socket读取不到属性数据则返回
        if (!socket.GetSourceContext(&source_context)) {
            PLOG(ERROR) << "Unable to set property '" << prop_name << "': getpeercon() failed";
            return;
        }

        const auto& cr = socket.cred();
        std::string error;
        uint32_t result = HandlePropertySet(prop_name, prop_value, source_context, cr, &error);
        if (result != PROP_SUCCESS) {
            LOG(ERROR) << "Unable to set property '" << prop_name << "' from uid:" << cr.uid
                       << " gid:" << cr.gid << " pid:" << cr.pid << ": " << error;
        }

        break;
      }

  ...
}
```
对客户端的请求进行处理，使用HandlePropertySet函数来进一步封装处理

系统属性分为两种类型：一种是普通属性；还有一种是控制属性，控制属性用来执行一些命令，比如开机的动画就使用了这种属性。ctl.开头的属性名称，就是控制属性



### init进程启动总结
1. 创建和挂载启动所需的文件目录
2. 初始化和启动属性服务
3. 解析init.rc配置文件并启动Zygote进程


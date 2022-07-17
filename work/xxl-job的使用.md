# xxl-job的使用

## 1.部署jiar包

### 1. mac上使用 

运行jar包

  ```shell
  java -jar xxl-job.jar
  ```

### 2. 修改jar包中的配置文件

1. 使用vim打开jar

   ```shell
   vim xxl-job.jar
   ```

2. vim 搜索指定文件

   - 使用?application.yml查找指定文件
   - 使用/application.yml查找指定文件

3. 打开文件

   搜索到指定文件后，shell中的光标会自动移动到文件名起点位置；

   此时按回车即可打开文件

4. vim修改指定配置后`:wq` 保存并关闭文件,继续使用`:q`即可退出vim

5. 使用上面的方法修改`applycation.yml`文件中的数据库地址和`log`文件存储地址

## 2. 配置任务

1. 进入配置[网站](http://localhost:8080/xxl-job-admin) ，输入账号和密码

   -  账号：admin

   - 密码：123456

2. 配置执行器：

   新增执行器：输入名称，AppName,执行器地址，本地地址：http://127.0.0.1:9999,执行器默认端口号为9999

3. 配置任务

   1. 选择配置好的执行器，并输入名称等必填项
   2. 选择调度配置中的调度类型为`corn` ，并配置`corn`公式；公式决定了任务的执行周期等
   3. 任务配置：选择bean模式，`JobHandler`输入，项目中`@XxlJob`注解中写入的内容



163测试环境的Job定时任务：
http://192.168.2.206:8080/xxl-job-admin/joblog
admin / 123456

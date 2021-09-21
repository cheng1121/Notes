# TFS的使用

## 安装idea插件

1. 插件市场查找TFS插件并安装

## 配置TFS

1. 在Version Control中找到TFS

2. 取消勾选warn about not installed policies

3. 点击manage

4. 点击右侧的add

5. 输入提示内容：

   - address: 服务器地址
   - auth: 使用默认的NTLM
   - user name:  公司分配的用户名
   - domain: 公司配置的域
   - password: 用户名对应的密码

6. 配置本地映射目录

   - 点击Version Control在右侧点击加号添加本地目录

   - 弹出框中选中Directory并且 VCS选择TFS,

## 克隆项目代码

1. 打开idea，TFS配置完成后上方菜单栏的vcs会变成TFS，点击TFS
2. 点击get from version control
3. 左侧选中Repository URL
4. 右侧的Version control 下拉菜单中选中TFS
5. 点击右下角的clone
6. 弹窗中输入新的工作区或者选择已有工作区，并点击next



## 更新代码

有几个地方可以做同样的处理

### 工具栏更新代码 

工具栏右侧也就是运行项目的绿色三角按钮右侧有蓝色指向左下角的箭头

点击后会更新代码文件

### 项目上右键更新

1. 在项目名称上鼠标右键
2. 在弹出菜单中选择TFS
3. 在TFS菜单中选择Update Directory

### 菜单栏选择TFS更新

1. 菜单栏中选择VCS/TFS 
2. update project



## 签入代码











### TFS窗口更新本地修改的文件列表

1. 在idea编辑窗口的下方，有个TFS选项卡点击后会展示TFS窗口

2. 在TFS窗口中有个Local Changes 选项卡
3. Local Changes选项卡右侧有个刷新按钮，点击后刷新列表
# 【一】 Markdown语法
## 标题
### 标题

>这是引用

1.语文
2.数学
3.英语

- 语文
- 数学
- 英语



```c
int main(){
    return 0;
}
```

$$
\frac{\partial f}{\partial x} = 2\sqrt{a}x
$$

行内数学公式：
$\theta=x^2$

|姓名|学号|
|:---:|:---:|
|norain|1015|

姓名[^真]

---
[百度](baidu.com"good")

请参考[标题](#标题)

http://www.baidu.com

![baidu](![alt text](image.png))

*斜体*

**加粗**

<u>下划线</u>

H~2~O

X^2^

==这是一段高亮文字==

# 【二】 VSCode Markdown 文件上传 GitHub 极简操作手册
## 一、前置前提（仅第一次配置，后续永久不用改）

1. WSL 已安装 Git
2. SSH 密钥绑定 GitHub 账号，免密码免验证码
3. 已绑定远程仓库：`ros-_study-_notes`
4. 本地仓库文件夹路径：
`\\wsl$\Ubuntu-24.04\home\norain\Robotcup-from-NoRain`
## 二、每次上传完整分步流程（照着点就行）

### 步骤 1：在 VSCode 写完 / 编辑好你的 `.md` 笔记文件

在 VSCode 里正常编写 Markdown 笔记、插入图片，全部编辑保存完毕。

### 步骤 2：复制需要上传的全部文件

1. 在 VSCode 左侧资源管理器，选中你要备份的 md、图片、素材(或者是在文件夹中打开，复制)
2. 快捷键 `Ctrl + C` 复制

### 步骤 3：打开本地仓库文件夹，粘贴文件

1. 打开「此电脑」，点击顶部地址输入框
2. 全选原有内容，粘贴下面路径，按下回车

```
\\wsl$\Ubuntu-24.04\home\norain\Robotcup-from-NoRain
```

3. 在弹出的文件夹空白处，快捷键 `Ctrl + V` 粘贴刚才复制的所有文件

### 步骤 4：打开 WSL 黑色终端，进入仓库目录

1. 调出 WSL 终端窗口
2. 输入下面指令并回车，**必须执行这一步，否则 Git 会报错找不到仓库**

```
cd ~/Robotcup-from-NoRain
```

### 步骤 5：三条固定命令，一键上传云端

逐条复制粘贴，每一行输完按一次回车

1. 抓取本次所有新增 / 修改的文件

```
git add .
```

2. 本地记录本次修改，引号内填写本次更新说明（可自定义文字）

```
git commit -m "填写本次上传备注，例：新增ROS笔记test.md"
```

3. 推送到 GitHub 云端仓库

```
git push
```

### 步骤 6：确认上传成功

终端末尾出现 `main -> main`，无红色报错，代表上传完成。
浏览器打开仓库页面刷新，即可看见文件。
# 【三】# Ubuntu24.04 WSL2 安装 ROS2 Jazzy 完整排坑笔记
## 一、前置系统基础配置
### 1. 系统版本确认
```bash
lsb_release -a
```
目标：确认系统为 Ubuntu 24.04 (noble)
### 2. UTF-8 中文环境配置
```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```
### 3. 开启 Universe 软件源
```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
```

## 二、ROS2 核心环境部署
### 1. 安装基础依赖
```bash
sudo apt install curl gnupg lsb-release -y
```

### 2. 导入ROS官方GPG密钥（核心排坑点）
#### 原始指令问题
直接执行GitHub源`raw.githubusercontent.com`链接极易出现**域名解析失败、连接超时、404报错**。
#### 修复方案
删除旧失效密钥，使用稳定方式导入公钥：
```bash
# 清理原有错误密钥文件
sudo rm -f /usr/share/keyrings/ros-archive-keyring.gpg
# 方案1：官方原生地址（多执行1~2次即可成功）
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
# 方案2：备用公钥导入命令（curl完全无法联网时使用）
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F42ED6FBAB17C654
```

### 3. 写入ROS2软件源
⚠️ 必须**整行完整复制，不可自动换行截断**，否则源地址残缺，后续找不到安装包
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu noble main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### 4. 更新软件源索引
```bash
sudo apt update
```
#### 高频报错：NO_PUBKEY F42ED6FBAB17C654
- 原因：密钥文件损坏/未正确下载，系统无法校验ROS源合法性
- 解决：回到上一步删除密钥文件，重新执行密钥下载指令

### 5. 安装ROS2完整版桌面包
```bash
sudo apt install ros-jazzy-desktop-full -y
```
#### 报错：E: Unable to locate package ros-jazzy-desktop-full
排查顺序：
1. 软件源是否粘贴完整、无换行
2. `sudo apt update` 是否执行成功无公钥报错
3. 网络环境是否能访问`packages.ros.org`

## 三、配置永久环境变量（根治新开终端命令失效）
### 1. 编辑环境配置文件
```bash
nano ~/.bashrc
```
### 2. 问题点：重复写入source指令
若文件末尾出现多行`source /opt/ros/jazzy/setup.bash`，会冗余加载；且**未安装ROS前写入该行**，新开终端会直接弹窗报错`No such file or directory`。
### 标准操作
1. 删除所有旧的ros/jazzy相关source行
2. 在文件末尾仅添加**单独一行**
```bash
source /opt/ros/jazzy/setup.bash
```
3. `Ctrl+O` 回车保存，`Ctrl+X` 退出编辑器
4. 生效配置
```bash
source ~/.bashrc
```

### 3. 环境验证指令
❌ 错误写法（ROS2不支持该参数）
```bash
ros2 --version
```
✅ 正确验证方式
```bash
# 查看当前ROS发行版
echo $ROS_DISTRO
# 输出 jazzy 即代表环境变量永久生效
# 详细环境检测
ros2 doctor --report
```

## 四、经典示例通信测试（Talker & Listener）
### 终端1：启动发布方
```bash
ros2 run demo_nodes_cpp talker
```
持续循环打印递增`Hello World`消息，无手动终止则一直运行。
### 终端2：启动订阅方
```bash
ros2 run demo_nodes_cpp listener
```
#### 卡死无接收消息排坑
WSL Windows端开启代理工具，会生成`HTTP_PROXY`环境变量，干扰DDS节点互相发现，表现为listener挂起无输出。
#### 临时清空代理（当前终端立即生效）
```bash
unset HTTP_PROXY HTTPS_PROXY ALL_PROXY http_proxy https_proxy all_proxy
```
#### 永久根治：
打开`~/.bashrc`，删除所有代理相关export语句，保存刷新配置。

## 五、小海龟Turtlesim仿真验收
### 终端1 启动海龟画布窗口
```bash
ros2 run turtlesim turtlesim_node
```
### 终端2 键盘控制节点
```bash
ros2 run turtlesim turtle_teleop_key
```
⚠️ 操作前提：鼠标点击选中键盘控制终端窗口，上下左右方向键才可控制海龟移动；`q`退出程序。
此步骤可完全证明ROS2图形界面、话题通信、节点调度全部正常。

## 六、配套开发工具一键安装
### 1. ROS官方开发工具集
```bash
sudo apt update && sudo apt install ros-dev-tools -y
```
### 2. 命令行参数自动补全
```bash
sudo apt install python3-argcomplete
```
### 3. Python包管理 & Colcon编译工具
```bash
sudo apt install python3-pip -y
sudo apt install python3-colcon-common-extensions
```
> 终端提示`already the newest version`代表已预装，无需重复执行；过程中Python SyntaxWarning为源码提示，不影响功能，可忽略。

## 七、全流程完成确认清单
- [x] Ubuntu24.04系统基础环境
- [x] ROS2 GPG密钥 + 软件源配置无报错
- [x] `ros-jazzy-desktop-full` 完整安装
- [x] `.bashrc` 环境变量单条写入，新开终端自动加载ROS
- [x] Talker/Listener 收发消息正常
- [x] Turtlesim海龟仿真可正常操控
- [x] dev-tools、argcomplete、pip、colcon编译扩展全部部署

## 八、可选：文档归档推送至个人GitHub仓库
复用固定Git推送流程备份本次排坑文档：
```bash
cd ~/Robotcup-from-NoRain
git add .
git commit -m "WSL2 Ubuntu24.04 ROS2 Jazzy 安装全流程+排坑解决方案存档"
git push
```

## 九、后续入门进阶路线
1. 创建ROS标准工作空间`colcon_ws`
2. 在src目录新建C++/Python自定义功能包
3. 编写自定义发布订阅节点，使用`colcon build`编译工程
4. 学习服务、动作、参数、TF坐标等ROS2核心通信机制
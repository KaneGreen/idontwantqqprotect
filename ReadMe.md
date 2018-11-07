# 我不需要QQProtect.exe

用于暂时性解决QQProtect.exe的**流氓行为**的问题。

### 背景介绍

腾讯公司的即时聊天软件QQ程序(Windows版本)会创建一个`QPCore Service`的服务，开机启动一个名为`QQProtect.exe`的程序，据官方解释为“腾讯安全服务”程序。 如果该“安全服务”程序被关闭，则QQ进程本身也会自动关闭；如果QQ进程启动时该“安全服务”程序没有运行，则会启动`QQProtect.exe`，若启动失败，QQ本身也将启动失败。  
2017年11月初，该程序会开始在后台**自动启动QQ浏览器的安装程序**，企图强制推广QQ浏览器。这种行为已经超出了正常软件的行为范围了，但此**流氓行为**并未被多数杀毒软件或安全软件拦截。  

## 我应该如何防护？

#### 启用UAC

首先，对于Win7及更新操作系统的用户，应该在开始菜单的输入框中输入`UAC`并按回车执行，打开`用户账户控制`，将滑块调至至上而下的第二档(推荐)或第一档，再点击确认按钮。  
这样，当QQ浏览器的安装程序或其他软件的安装程序启动前，您将看到一个`用户账户控制`的对话框询问您`是否允许该程序对您的计算机的修改？`。此时，您应该点击查看详细信息并核对该程序是否是您需要安装的程序。对于本文所述的“强制推广的QQ浏览器”，您应该点击`否`。  
如果你的电脑上有安装具有HIPS功能的安全软件，建议开启该功能，用于阻止不明来源的程序在后台安装。

#### 挂起QQProtect.exe
由于`QQProtect.exe`在后台允许需要占用一定的系统资源，并且可能在一段时间之后再次触发“安装QQ浏览器”，但是我们不能将其关闭，因此需要将其挂起。  
**下面有两种方法可以使用，请自行选择其一即可。**

##### 自动操作
在[微软SysinternalsSuite官方网站](https://docs.microsoft.com/en-us/sysinternals/downloads/pssuspend)下载`PsTools`工具包，得到一个zip压缩包，解压的文件中含有大量exe可执行文件，但这里只需要其中的`PsSuspend`工具。  
【32位操作系统用户请使用`pssuspend.exe`，64位操作系统用户请使用`pssuspend64.exe`。】  
1. 如果您的PowerShell的脚本执行策略（使用`Get-ExecutionPolicy`命令可以查看）不是`RemoteSigned`或`Undefined`，请使用管理员身份运行PowerShell，再执行`Set-ExecutionPolicy RemoteSigned`命令来将其修改。
2. 在一个将上面得到的文件复制到一个目录下面，例如`C:\QQstartup\`
3. 然后将以下PowerShell脚本保存到这个目录下  
您可能需要根据您QQ的安装位置不同和实际需要，对以此进行适当修改。  
这个例子中，我假设您将这个脚本保存到了`C:\QQstartup\QQwithoutProtect.ps1`。
```PowerShell
Start-Process -FilePath "pssuspend64.exe" -ArgumentList "-r","QQProtect.exe" -Verb runas -Wait
Start-Process -FilePath "C:\Program Files (x86)\Tencent\QQ\Bin\QQ.exe" -WorkingDirectory "C:\Program Files (x86)\Tencent\QQ\Bin\"
Start-Sleep -Seconds 10
Start-Process -FilePath "pssuspend64.exe" -ArgumentList "QQProtect.exe" -Verb runas -Wait
```
4. 将QQ的快捷方式的`目标`修改为：
`%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe C:\QQstartup\QQwithoutProtect.ps1`，  
并将`起始位置`改为：
`C:\QQstartup\`。
5. 此后**每次**启动QQ，都会先遇到一次UAC提示，请点击`是`，  
然后是QQ主程序启动，  
大约10秒后再遇到一次UAC提示，请也点击`是`。  
（这里大家可能觉得10秒太短，不够输入密码：但是也不用着急输入密码，因为待第二次UAC提示后，QQ登陆窗口也是有效的）
（不建议为了少点一次UAC提示，而关闭UAC或以管理员身份运行QQ）


##### 手动操作
在[微软SysinternalsSuite官方网站](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer)下载`ProcessExplorer`工具，得到一个zip压缩包，解压的文件中会有2个exe可执行文件。  
【32位操作系统用户请使用`procexp.exe`，64位操作系统用户请使用`procexp64.exe`。】  
**每一次登陆QQ，都需要按照以下步骤：**  
1. 运行QQ之前，以管理员身份运行该工具，找到`QQProtect.exe`进程，鼠标右键点击它，在弹出菜单中点击`Resume`(恢复)。  
**如果没有找到`Resume`，而是有`Suspend`，请直接进行下一步。**
2. 正常登陆QQ。如果有登陆多个QQ的需求，这里你可以像一般使用一样，登陆多个。
3. 在QQ登陆完成后，同样，在该工具中找到`QQProtect.exe`进程，鼠标右键点击它，在弹出菜单中点击`Suspend`(挂起)，之后就可以关闭`ProcessExplorer`这个工具了。

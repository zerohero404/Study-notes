<img width="699" height="367" alt="Linux：虚拟化18" src="https://github.com/user-attachments/assets/9f46f544-9a5c-4d45-a2f1-a418ee4b6539" />- ESXi 安装
- VMware EXSI 是一款行业领先，专门构建的裸机 hypervisor
- ESXI 直接安装在物理服务器上，并将其划分为多个逻辑服务器，即虚拟机
- 安装版本
  - VMware-VMvisor-Installer-6.0.0-2494585.x86_64.iso
- 连接客户端
  - VMware-viclient-all-6.0.0.exe
- 安装步骤
  - 制作U盘启动盘
    - 使用2中下载的文件制作ESXi的U盘启动盘，与制作一般操作系统的U盘启动盘没有区别
  - 选择U盘启动
  - 进入安装界面后，首先会读取iso，然后进行初始化<br>
  <img width="943" height="677" alt="Linux：虚拟化6" src="https://github.com/user-attachments/assets/406b1fe6-8e44-46a9-a02a-07d03bbd1b63" /><br>
  - 初始化<br>
  <img width="700" height="504" alt="Linux：虚拟化7" src="https://github.com/user-attachments/assets/be404ef8-26f7-498e-b99d-b9bd83a28bd3" /><br>
  - 进入到真正的安装界面，点击”enter”继续<br>
  <img width="620" height="218" alt="Linux：虚拟化8" src="https://github.com/user-attachments/assets/0798daeb-e43b-4f8f-951b-c88ba5214224" />
  - 点击”F11″继续<br>
  <img width="699" height="365" alt="Linux：虚拟化9" src="https://github.com/user-attachments/assets/c1959e43-aa8b-43cd-ad27-ab5934bd5dd1" />
  - 选择安装到哪个硬盘上，注意不要错选成U盘。选择完成之后按”enter”键继续<br>
  <img width="921" height="386" alt="Linux：虚拟化10" src="https://github.com/user-attachments/assets/205d12a1-1ca3-44ad-858b-98f1ef62c0bd" /><br>
  - 确认选择的硬盘，确认之后按”enter”确认<br>
  <img width="703" height="296" alt="Linux：虚拟化12" src="https://github.com/user-attachments/assets/22e9f795-8a6f-4fec-952d-faf279369937" /><br>
  - 选择键盘布局，选择默认的就好，按”enter”继续<br>
  <img width="578" height="287" alt="Linux：虚拟化13" src="https://github.com/user-attachments/assets/80aaba47-260d-4512-a1d9-b6d66e38da9d" /><br>
  - 设置密码，因为ESXi是服务器级别的虚拟机，所以对密码的要求比较严格，密码格式有几点要求
    - 第一个为大写字母
    - 接下来为小写字母
    - 然后再加一串数字
    - 最后一位为一个符号
    - 密码长度不少于八位<br>
    <img width="596" height="210" alt="Linux：虚拟化15" src="https://github.com/user-attachments/assets/c0c80bbb-87cb-40f5-9832-15bd4c94e932" /><br>
  - 设置成功后按”enter”进入下一项
  - 在进入下一步之前可能会出现一个warning，原因是没有开启bios中的虚拟化技术，到bios中开启就好<br>
  <img width="701" height="350" alt="Linux：虚拟化16" src="https://github.com/user-attachments/assets/49986c25-d780-43a8-b5ac-c4496569b490" /><br>
  - 确认安装，按”F11″确认安装<br>
  <img width="703" height="183" alt="Linux：虚拟化17" src="https://github.com/user-attachments/assets/32c0f8c5-9346-4fa7-9ba6-0937bf3e8229" /><br>
  - 点击”enter”重启主板。这时还没有拔掉U盘的注意要把U盘拔掉<br>
  <img width="699" height="367" alt="Linux：虚拟化18" src="https://github.com/user-attachments/assets/0967d8e8-e8bd-414f-8ba2-a95c31ce588a" /><br>
  - 重新进入系统中是如下界面<br>
  <img width="692" height="448" alt="Linux：虚拟化19" src="https://github.com/user-attachments/assets/f626761a-99ac-45a7-a839-fab57f79e121" /><br>
  - 按”F2″进入参数配置界面。需要输入用户名和密码，用户名就是root，密码是你设置的密码。<br>
  <img width="690" height="464" alt="Linux：虚拟化20" src="https://github.com/user-attachments/assets/96036233-5043-414e-9a4c-c2354d45bc34" /><br>
  - 如果想要取消密码的话，选择最后一项”Reset System Configuration”，每次进入系统就无需输入密码。
  - 配置网络，选择第三项”Configure Management Network”进入<br>
  <img width="702" height="450" alt="Linux：虚拟化21" src="https://github.com/user-attachments/assets/1514b8f1-136a-422a-9977-808abcbff620" /><br>
  - 选择第三项”IPv4 Configuration”进入<br>
  <img width="696" height="278" alt="Linux：虚拟化22" src="https://github.com/user-attachments/assets/5a545ec3-c044-4d3b-a9d6-501baee8fc68" /><br>
  - 设置你的网段，掩码还有DNS即可
  - 个人电脑要和服务器在同一网段，然后在个人电脑上安装 VMware-viclient-all-6.0.0.exe，然后输入服务器的 IP 、账号和密码登录即可管理
















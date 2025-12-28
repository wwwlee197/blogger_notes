連線步驟(請使用手機熱點操作
PC端連上手機熱點後確認IPv4地址(172.20.10.4)
```
ipconfig
```
![](../專題/廠商預設架構/內容/PC端%20&%20機器人端%20熱點設定/file-20251228222108441.png)
## 虛擬機端
設定與PC端相同網卡，網路設定成橋接模式
![](../專題/廠商預設架構/內容/PC端%20&%20機器人端%20熱點設定/file-20251228222117226.png)

cmd中確認虛擬機ip
```
ifconfig
```
![](../專題/廠商預設架構/內容/PC端%20&%20機器人端%20熱點設定/file-20251228222134702.png)

修改 ROS_IP & ROS_MASTER_URI
```
sudo nano .bashrc
```
.bashrc 設定如下
```
export ROS_IP=172.20.10.4
export ROS_MASTER_URI=http://172.20.10.4:11311
```
ROS_IP=虛擬機ip
ROS_MASTER = 指向自己 ROS_IP (將自己設為master)


## 樹梅派-機器人端
Etho: 乙太網路 
Wifi : 無線網路
```
sudo nano /etc/netplan/50-cloud-init.yam
```
![](../專題/廠商預設架構/內容/PC端%20&%20機器人端%20熱點設定/file-20251228222139964.png)

連線至手機熱點 wifis 後輸入下面指令重新啟動網路連線
```
sudo netplan generate
sudo netplan apply
```

```
ifconfig
```
確認wifis 欄位是否ip更新為手機熱點ip

虛擬機端執行:確認是否能與機器人端依靠ip通信
```
ip 172.20.10.4      //手機熱點ip
```
![](../專題/廠商預設架構/內容/PC端%20&%20機器人端%20熱點設定/file-20251228222147409.png)

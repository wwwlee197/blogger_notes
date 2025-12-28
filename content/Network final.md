## 題目
![](assets/Network%20final/file-20251229020132285.png)
![](assets/Network%20final/file-20251229020137562.png)
一、設定基本資訊
1. 設定路由器/交換器名稱：R1、R2、R3、R4、R5、S1、S2、S3
2. 參考拓樸，設定路由器的IP位址，並啟動介面
3. 參考拓樸，設定PC的IP位址、子網路遮罩及預設閘道

二、設定路由

1. R1、R2、R3、R4、R5設定OSPF，process id號碼為1，使用萬用遮罩傳送指定子網路。
2. 在R1上設定預設路由(使用出口介面)，並將此預設路由傳送到OSPF


三、設定EthernetChannel

1. 將S1的FastEthernet0/11和FastEthernet0/12設定為LACP，群組號碼為1，模式為active。
2. 將S2的FastEthernet0/11和FastEthernet0/12設定為LCAP，群組號碼為1，模式為passive。

 
四、設定VTP
1.  S1、S2交換器之間設定為trunk
2. 將S1設定為VTP Server、S2設定為VTP Client，VTP Domain皆設定為ccna
3. 在S1上建立VLAN：vlan10：teacher、vlan20：student
4. 確認S2有收到上述VLAN


五、設定InterVLAN Routing
1. 在S2設定連接埠對應的VLAN：vlan10：F0/1-F0/2、vlan20：F0/6-F0/7
2. 參考拓樸設定交換器S1上F0/24的trunk port
3. PC1、PC2、PC3、PC4互ping都能成功


六、設定HSRP
1. R4、R5之間啟動HSRP，使用群組編號1，虛擬IP為192.168.30.254
2. 將R4設為活動路由器，設定priority為150，並啟用搶佔功能
3. 確認PC4可以ping到PC1、PC2、PC3、PC4

七、設定DHCP
1. 在R2上設定192.168.40.0/24的DHCP Server
   pool的名稱為 Lan40
   預設閘道為 192.168.40.254
   請排除.1到.9的IP位址
2. 確認PC6都有取得.10的IP位址，且可以ping到PC1、PC2、PC3、PC4、PC5

 
八、設定NAT
1. 在R1上設定PAT，使用ACL編號1號，分別設定192.168.10.0/24、192.168.20.0/24、192.168.30.0/24、192.168.40.0/24可以透過NAT上網、(有四條敘述)。
2. 使用出口介面的IP當成inside global
3. 設定inside / outside 介面
4. 測試PC1、PC2、PC3、PC4、PC5、PC6 ping Web Server，都能成功

九、設定ACL
1. 測試PC1、PC2可以透過瀏覽器連到Web Server
2. 在R2的G0/0.10介面，設定命名型延伸ACL，使用名稱pc2noweb，讓PC2無法 http到Web Server。
3. 測試PC1可以透過瀏覽器連到Web Server，PC2無法透過瀏覽器連到Web Server

---
# 指令一鍵複製貼上

##### Router R1
>功能： 基本 IP、OSPF (含預設路由發布)、NAT/PAT。

```
enable
conf t
hostname R1
!
interface S0/0/0
 ip address 200.200.10.1 255.255.255.252
 ip nat outside
 no shutdown
!
interface S0/0/1
 ip address 192.168.1.254 255.255.255.0
 ip nat inside
 no shutdown
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 S0/0/0
!
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.40.0 0.0.0.255
ip nat inside source list 1 interface S0/0/0 overload
!
end
write
```

##### Router R2
>功能： 基本 IP、Router-on-a-Stick (VLAN Gateway)、OSPF、DHCP Server、ACL (阻擋 PC2 上網)。

```
enable
conf t
hostname R2
!
interface S0/0/0
 ip address 192.168.1.253 255.255.255.0
 no shutdown
!
interface S0/0/1
 ip address 192.168.2.254 255.255.255.0
 no shutdown
!
interface G0/1
 ip address 192.168.40.254 255.255.255.0
 no shutdown
!
interface G0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
 ip access-group pc2noweb in
!
interface G0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
!
interface G0/0
 no shutdown
!
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
!
ip dhcp excluded-address 192.168.40.1 192.168.40.9
ip dhcp pool Lan40
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.254
!
ip access-list extended pc2noweb
 deny tcp host 192.168.10.2 host 120.96.82.30 eq 80
 permit ip any any
!
end
write
```

##### Router R3
>功能： 基本 IP、OSPF。
```
enable
conf t
hostname R3
!
interface S0/0/0
 ip address 192.168.2.253 255.255.255.0
 no shutdown
!
interface G0/0
 ip address 192.168.3.254 255.255.255.0
 no shutdown
!
interface G0/1
 ip address 192.168.4.254 255.255.255.0
 no shutdown
!
router ospf 1
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
!
end
write
```

##### Router R4
>功能： 基本 IP、OSPF、HSRP (Active)。
```
enable
conf t
hostname R4
!
interface G0/1
 ip address 192.168.3.253 255.255.255.0
 no shutdown
!
interface G0/0
 ip address 192.168.30.253 255.255.255.0
 standby 1 ip 192.168.30.254
 standby 1 priority 150
 standby 1 preempt
 no shutdown
!
router ospf 1
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
!
end
write
```

##### Router R5
>功能： 基本 IP、OSPF、HSRP (Standby)。
```
enable
conf t
hostname R5
!
interface G0/1
 ip address 192.168.4.253 255.255.255.0
 no shutdown
!
interface G0/0
 ip address 192.168.30.252 255.255.255.0
 standby 1 ip 192.168.30.254
 no shutdown
!
router ospf 1
 network 192.168.4.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
!
end
write
```

##### Switch S1
>功能： EtherChannel (Active)、VTP Server、建立 VLAN、Trunk。
```
enable
conf t
hostname S1
!
interface range f0/11-12
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
!
interface port-channel 1
 switchport mode trunk
!
vtp domain ccna
vtp mode server
vlan 10
 name teacher
vlan 20
 name student
!
interface f0/24
 switchport mode trunk
!
end
write
```

##### Switch S2
>功能： EtherChannel (Passive)、VTP Client、Trunk、分配 VLAN Port。
```
enable
conf t
hostname S2
!
interface range f0/11-12
 channel-protocol lacp
 channel-group 1 mode passive
 no shutdown
!
interface port-channel 1
 switchport mode trunk
!
vtp domain ccna
vtp mode client
!
interface range f0/1-2
 switchport mode access
 switchport access vlan 10
!
interface range f0/6-7
 switchport mode access
 switchport access vlan 20
!
end
write
```

Switch S3
功能： 僅需改名。
```
enable
conf t
hostname S3
end
write
```

##### PC 設定

PC1: 192.168.10.1 / 255.255.255.0, GW: 192.168.10.254

PC2: 192.168.10.2 / 255.255.255.0, GW: 192.168.10.254

PC3: 192.168.20.1 / 255.255.255.0, GW: 192.168.20.254

PC4: 192.168.20.2 / 255.255.255.0, GW: 192.168.20.254

PC5: 192.168.30.1 / 255.255.255.0, GW: 192.168.30.254 
(這是 HSRP 虛擬 IP)

PC6: 選擇 DHCP 自動取得 
(應為 .10)
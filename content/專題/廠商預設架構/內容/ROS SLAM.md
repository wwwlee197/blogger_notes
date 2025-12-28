遠端電腦 

啟動主控節點Master
```
roscore
```

ssh登入後,機器人端 啟動底盤控制 + 光學雷達
```
roslaunch robot_setup_tf MBR_robot_bringup.launch
```

啟動 SLAM（同步定位與地圖建立)相關節點
```
roslaunch robot_2nav gmapping.launch
```

rviz 可視覺化觀察 SLAM 運作的狀態
File -> Open config -> 接著選擇在 catkin_ws/src/robot_2dnav/mapping.rviz 設定檔
```
rviz
```

啟用搖桿功能 [[鍵盤操控]]
```
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

處存當前地圖,地圖會被儲存在/home下
```
rosrun map_server map_saver–f [地圖名稱]
```
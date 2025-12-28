
### ros1
  
```
catkin_ws/
├── build/
├── devel/
├── src/
│   └── <packages>
└── CMakeLists.txt
```
- build/：編譯時產生的中間檔案
- devel/：開發環境（執行檔、設定檔等）
- src/：原始碼與 package
- CMakeLists.txt：catkin workspace 的入口

ROS 1：主要用 catkin_make 或 catkin build
  
---
### ros2
  
```
ros2_ws/
├── build/
├── install/
├── log/
├── src/
│   └── <packages>
```
- build/：編譯時產生的中間檔案
- install/：安裝後的可執行檔與資源（執行時會 source 這裡）
- log/：編譯與執行的 log 檔
- src/：原始碼與 package

ROS 1：主要用 catkin_make 或 catkin build
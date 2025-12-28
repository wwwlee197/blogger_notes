1. bridge.cpp

- 橋接的主要實作檔案，負責 ROS1 與 ROS2 之間的訊息轉換與傳遞。

2. builtin_interfaces_factories.cpp

- 處理 ROS 內建訊息型別（如時間、空間座標等）的轉換工廠。

3. convert_builtin_interfaces.cpp

- 負責將 ROS1/ROS2 內建訊息型別互相轉換。

4. dynamic_bridge.cpp

- 動態橋接的主程式，會自動偵測 ROS1/ROS2 的 topic 並建立對應的橋接（你常用的 ros2 run ros1_bridge dynamic_bridge 就是這個）。

5. parameter_bridge.cpp

- 處理 ROS1/ROS2 之間參數（parameter）的同步與橋接。

6. simple_bridge.cpp

- 簡單橋接的實作，通常用於靜態、手動指定 topic 的橋接。

7. simple_bridge_1_to_2.cpp / simple_bridge_2_to_1.cpp

- 單向橋接，分別從 ROS1 → ROS2 或 ROS2 → ROS1。

8. static_bridge.cpp

- 靜態橋接，需手動指定要橋接的 topic，適合固定 topic 應用。
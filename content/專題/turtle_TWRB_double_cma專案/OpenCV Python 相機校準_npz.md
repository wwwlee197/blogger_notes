
### 1. 內部參數（Intrinsics）-> 成像特性

- 焦距 (focal length, f)

決定影像縮放比例，通常以像素為單位。

- 主點 (principal point, cx, cy)

影像的光軸中心點（通常接近影像中心）。

- 畸變參數 (distortion, k1, k2, p1, p2, k3)

用來描述鏡頭的徑向與切向畸變（鏡頭變形）。

---

### 2. 外部參數（Extrinsics）-> 拍攝角度

- 旋轉 (Rotation, R)

3x3 矩陣，描述相機座標系與世界座標系的旋轉關係。

- 平移 (Translation, T)

3x1 向量，描述相機座標系與世界座標系的平移關係。
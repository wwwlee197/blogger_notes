
```python
# 使用雙攝影機即時偵測 ArUco 標記、估算三維位置與姿態

  

import cv2

import numpy as np

import os

  

class DualCameraArucoDetector:

    def __init__(self, cam1_index=1, cam2_index=2, calibration_file="calibration.npz"):

        self.cam1_index = cam1_index

        self.cam2_index = cam2_index

        self.calibration_file = calibration_file

  

        # ArUco 檢測參數

        self.aruco_dict = self.get_aruco_dictionary()

        self.aruco_params = self.get_aruco_parameters()

        self.marker_size = 5.0  # 公分

  

        self.camera_matrix = None

        self.dist_coeffs = None

        self.cap1 = None

        self.cap2 = None

        self.cam_width = 845

        self.cam_height = 480

  

        print(f"📋 OpenCV 版本: {cv2.__version__}")

        self.load_calibration()

  

    def get_aruco_dictionary(self):

        try:

            return cv2.aruco.getPredefinedDictionary(cv2.aruco.DICT_6X6_1000)

        except AttributeError:

            try:

                return cv2.aruco.Dictionary_get(cv2.aruco.DICT_6X6_1000)

            except AttributeError:

                print("❌ 不支援的 OpenCV 版本")

                return None

  

    def get_aruco_parameters(self):

        try:

            return cv2.aruco.DetectorParameters()

        except AttributeError:

            try:

                return cv2.aruco.DetectorParameters_create()

            except AttributeError:

                print("❌ 無法創建 ArUco 參數")

                return None

  

    def detect_markers_compatible(self, frame):

        try:

            detector = cv2.aruco.ArucoDetector(self.aruco_dict, self.aruco_params)

            corners, ids, rejected = detector.detectMarkers(frame)

            return corners, ids, rejected

        except AttributeError:

            corners, ids, rejected = cv2.aruco.detectMarkers(frame, self.aruco_dict, parameters=self.aruco_params)

            return corners, ids, rejected

  

    def draw_axis_compatible(self, frame, camera_matrix, dist_coeffs, rvec, tvec, length):

        try:

            cv2.drawFrameAxes(frame, camera_matrix, dist_coeffs, rvec, tvec, length)

        except AttributeError:

            try:

                cv2.aruco.drawAxis(frame, camera_matrix, dist_coeffs, rvec, tvec, length)

            except AttributeError:

                pass

  

    def load_calibration(self):

        if not os.path.exists(self.calibration_file):

            print(f"❌ 找不到校正檔案: {self.calibration_file}")

            print("請先執行相機校正程式產生 calibration.npz 檔案")

            return False

        try:

            calib_data = np.load(self.calibration_file)

            self.camera_matrix = calib_data['camMatrix']

            self.dist_coeffs = calib_data['distCoeff']

            print("✅ 校正參數載入成功")

            print(f"Camera Matrix:\n{self.camera_matrix}")

            print(f"Distortion Coefficients: {self.dist_coeffs.flatten()}")

            return True

        except Exception as e:

            print(f"❌ 載入校正參數失敗: {e}")

            return False

  

    def init_cameras(self):

        print(f"📹 初始化攝影機 {self.cam1_index} 和 {self.cam2_index}")

        self.cap1 = cv2.VideoCapture(self.cam1_index, cv2.CAP_DSHOW)

        if not self.cap1.isOpened():

            print(f"❌ 無法開啟攝影機 {self.cam1_index}")

            return False

        self.cap2 = cv2.VideoCapture(self.cam2_index, cv2.CAP_DSHOW)

        if not self.cap2.isOpened():

            print(f"❌ 無法開啟攝影機 {self.cam2_index}")

            self.cap1.release()

            return False

        for cap in [self.cap1, self.cap2]:

            cap.set(cv2.CAP_PROP_FRAME_WIDTH, self.cam_width)

            cap.set(cv2.CAP_PROP_FRAME_HEIGHT, self.cam_height)

            cap.set(cv2.CAP_PROP_FPS, 30)

        print("✅ 兩台攝影機初始化成功")

        return True

  

    def detect_aruco_markers(self, frame, camera_matrix, dist_coeffs):

        corners, ids, _ = self.detect_markers_compatible(frame)

        marker_info = []

        if ids is not None and len(ids) > 0:

            valid_indices = []

            for i, marker_id in enumerate(ids.flatten()):

                if 1 <= marker_id <= 6:

                    valid_indices.append(i)

            if valid_indices:

                valid_corners = [corners[i] for i in valid_indices]

                valid_ids = [ids[i] for i in valid_indices]

                rvecs, tvecs, _ = cv2.aruco.estimatePoseSingleMarkers(

                    valid_corners, self.marker_size, camera_matrix, dist_coeffs)

                cv2.aruco.drawDetectedMarkers(frame, valid_corners, np.array(valid_ids))

                for i, (rvec, tvec, marker_id) in enumerate(zip(rvecs, tvecs, valid_ids)):

                    self.draw_axis_compatible(frame, camera_matrix, dist_coeffs,

                                             rvec, tvec, self.marker_size * 0.5)

                    center_3d = np.array([[0.0, 0.0, 0.0]], dtype=np.float32)

                    center_2d, _ = cv2.projectPoints(center_3d, rvec, tvec, camera_matrix, dist_coeffs)

                    center_2d = center_2d.reshape(-1, 2).astype(int)

                    marker_info.append({

                        'id': marker_id[0],

                        'position_3d': tvec[0],

                        'position_2d': center_2d[0],

                        'rotation': rvec[0]

                    })

                    text_pos = (center_2d[0][0] + 20, center_2d[0][1])

                    cv2.putText(frame, f"ID:{marker_id[0]}", text_pos,

                                cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)

        return frame, marker_info

  

    def print_marker_info(self, cam1_markers, cam2_markers, frame_count):

        if frame_count % 30 == 0 and (cam1_markers or cam2_markers):

            print("\n" + "="*80)

            print(f"🎯 第 {frame_count} 幀 - ArUco 標記檢測結果")

            print("="*80)

            print(f"\n📹 攝影機 {self.cam1_index}:")

            if cam1_markers:

                print(f"   檢測到 {len(cam1_markers)} 個標記")

                for marker in cam1_markers:

                    x, y, z = marker['position_3d']

                    px, py = marker['position_2d']

                    print(f"   ├─ ID {marker['id']:2d}: 3D座標({x:6.1f}, {y:6.1f}, {z:6.1f})cm  影像座標({px:3d}, {py:3d})px")

            else:

                print("   └─ 未檢測到標記")

            print(f"\n📹 攝影機 {self.cam2_index}:")

            if cam2_markers:

                print(f"   檢測到 {len(cam2_markers)} 個標記")

                for marker in cam2_markers:

                    x, y, z = marker['position_3d']

                    px, py = marker['position_2d']

                    print(f"   ├─ ID {marker['id']:2d}: 3D座標({x:6.1f}, {y:6.1f}, {z:6.1f})cm  影像座標({px:3d}, {py:3d})px")

            else:

                print("   └─ 未檢測到標記")

            print("-"*80)

  

    def run(self):

        if self.camera_matrix is None:

            print("❌ 校正參數未載入，無法執行")

            return

        if not self.init_cameras():

            print("❌ 攝影機初始化失敗")

            return

        print("🚀 雙攝影機 ArUco 檢測系統啟動")

        print("操作說明:")

        print("  ESC: 退出程式")

        print("  S: 儲存當前畫面")

        print("📊 座標資訊將顯示在此控制台中")

        print("-" * 50)

        cv2.namedWindow(f'Camera {self.cam1_index}', cv2.WINDOW_NORMAL)

        cv2.namedWindow(f'Camera {self.cam2_index}', cv2.WINDOW_NORMAL)

        frame_count = 0

        try:

            while True:

                ret1, frame1 = self.cap1.read()

                ret2, frame2 = self.cap2.read()

                if not ret1 or not ret2:

                    print("⚠️ 無法讀取攝影機畫面")

                    continue

                # ArUco 標記檢測

                frame1_with_markers, cam1_markers = self.detect_aruco_markers(

                    frame1, self.camera_matrix, self.dist_coeffs)

                frame2_with_markers, cam2_markers = self.detect_aruco_markers(

                    frame2, self.camera_matrix, self.dist_coeffs)

                # 顯示原始畫面

                cv2.imshow(f'Camera {self.cam1_index}', frame1_with_markers)

                cv2.imshow(f'Camera {self.cam2_index}', frame2_with_markers)

                # 控制台顯示標記資訊

                self.print_marker_info(cam1_markers, cam2_markers, frame_count)

                key = cv2.waitKey(1) & 0xFF

                if key == 27:  # ESC

                    print("🛑 用戶按下 ESC，程式退出")

                    break

                elif key == ord('s') or key == ord('S'):

                    filename1 = f"camera{self.cam1_index}_capture_{frame_count:04d}.jpg"

                    filename2 = f"camera{self.cam2_index}_capture_{frame_count:04d}.jpg"

                    cv2.imwrite(filename1, frame1_with_markers)

                    cv2.imwrite(filename2, frame2_with_markers)

                    print(f"📸 畫面已儲存: {filename1}, {filename2}")

                frame_count += 1

        except KeyboardInterrupt:

            print("🛑 程式被中斷")

        finally:

            self.cleanup()

  

    def cleanup(self):

        print("🧹 清理資源中...")

        if self.cap1:

            self.cap1.release()

        if self.cap2:

            self.cap2.release()

        cv2.destroyAllWindows()

        print("✅ 資源清理完成")

  

def main():

    print("🚀 雙攝影機 ArUco 標記檢測系統")

    print("=" * 50)

    detector = DualCameraArucoDetector(cam1_index=1, cam2_index=2)

    detector.run()

  

if __name__ == "__main__":

    main()
```
# Stopwatch - Lab A2

Ứng dụng **đồng hồ bấm giờ (Stopwatch)** được xây dựng cho Lab A2 của học phần **Lập trình trên các thiết bị di động (INT4211)**.

Project tập trung vào xử lý vòng đời `Activity`, cập nhật giao diện định kỳ bằng `Handler` và lưu, khôi phục trạng thái bằng `onSaveInstanceState`.

## 1. Mục tiêu dự án

- Hiểu và xử lý đúng các callback vòng đời của `Activity` như `onStart`, `onResume`, `onPause`, `onStop`, `onRestart` và `onDestroy`.
- Xây dựng đồng hồ bấm giờ bằng `SystemClock.elapsedRealtime()` để tính thời gian đã trôi qua.
- Sử dụng `Handler` và `Runnable` để cập nhật giao diện định kỳ mà không chặn main thread.
- Lưu và khôi phục trạng thái khi `Activity` bị tạo lại, đặc biệt trong trường hợp xoay màn hình.

## 3. Chức năng chính

### 3.1. Đồng hồ bấm giờ

Ứng dụng hỗ trợ ba thao tác chính:

- **Bắt đầu:** bắt đầu hoặc tiếp tục đếm thời gian.
- **Tạm dừng:** dừng việc đếm và giữ nguyên thời gian đã tích lũy.
- **Đặt lại:** đưa đồng hồ về `00:00.0`.

Thời gian được tính dựa trên mốc `SystemClock.elapsedRealtime()` thay vì cộng thêm một lượng thời gian cố định sau mỗi lần cập nhật. Cách này giúp kết quả không phụ thuộc hoàn toàn vào chu kỳ cập nhật giao diện.

### 3.2. Cập nhật giao diện bằng Handler

Ứng dụng sử dụng `Handler` gắn với main thread và một `Runnable` để cập nhật đồng hồ định kỳ.

Ticker được quản lý bằng hai hàm:

```java
startTicking();
stopTicking();
```

Trước khi đăng ký ticker mới, callback cũ được gỡ bằng `removeCallbacks()` để tránh tình trạng nhiều ticker chạy đồng thời. Khi `Activity` bị hủy, callback cũng được loại bỏ để hạn chế nguy cơ giữ tham chiếu tới `Activity` cũ.

### 3.3. Lưu và khôi phục trạng thái

Các trạng thái quan trọng của đồng hồ được lưu vào `Bundle` trong `onSaveInstanceState()` và khôi phục lại trong `onCreate()`.

Các giá trị chính gồm:

- Trạng thái đang chạy hay tạm dừng.
- Thời gian đã tích lũy.
- Mốc `startTime`.
- Số lần `Activity` được tạo lại.
- Danh sách Lap của phần nâng cao NC1.

Nhờ đó, khi xoay màn hình, dữ liệu của đồng hồ không bị mất.

## 4. Vòng đời Activity

Project có sử dụng các callback vòng đời để kiểm soát việc cập nhật giao diện:

- `onResume()`: nếu đồng hồ đang chạy thì khởi động lại ticker.
- `onPause()`: dừng cập nhật giao diện để tránh hoạt động không cần thiết khi Activity không còn ở trạng thái tương tác.
- `onDestroy()`: luôn gọi `stopTicking()` để dọn dẹp callback.
- `onSaveInstanceState()`: lưu trạng thái cần thiết để khôi phục sau khi Activity bị tạo lại.

Điểm quan trọng là **dừng ticker không đồng nghĩa với dừng đồng hồ**. Thời gian thực tế vẫn được suy ra từ mốc `elapsedRealtime()`.

## 5. Bài nâng cao đã thực hiện

Project hoàn thành hai bài nâng cao:

- **NC1 - Nút Vòng (Lap)**
- **NC3 - Đổi màu con số và rung khi Đặt lại**

---

# 6. Ý tưởng triển khai NC1 - Nút Vòng (Lap)

## 6.1. Mục tiêu

NC1 yêu cầu thêm một nút **Lap** để lưu mốc thời gian hiện tại vào danh sách, hiển thị danh sách phía dưới đồng hồ và bảo đảm danh sách không mất khi xoay màn hình.

## 6.2. Ý tưởng

Thay vì tạo một cơ chế tính thời gian riêng cho Lap, phần nâng cao tận dụng trực tiếp hàm:

```java
elapsed()
```

Mỗi khi người dùng nhấn **Lap**:

1. Lấy thời gian hiện tại của stopwatch.
2. Chuyển thời gian sang dạng `mm:ss.t`.
3. Tạo chuỗi có số thứ tự Lap.
4. Thêm chuỗi vào `ArrayList<String>`.
5. Cập nhật `TextView` hiển thị danh sách.

Ví dụ:

```text
Lap 1: 00:03.4
Lap 2: 00:07.8
Lap 3: 00:12.2
```

## 6.3. Lưu danh sách khi xoay màn hình

Danh sách Lap được lưu vào `Bundle` trong `onSaveInstanceState()` bằng:

```java
outState.putStringArrayList(KEY_LAPS, new ArrayList<>(laps));
```

Khi `Activity` được tạo lại, danh sách được lấy ra bằng:

```java
savedInstanceState.getStringArrayList(KEY_LAPS);
```

Nhờ đó, các Lap đã ghi trước khi xoay màn hình vẫn được giữ lại.

## 6.4. Hiển thị danh sách

Danh sách Lap được hiển thị trong một `TextView` đặt bên trong `ScrollView`. Khi số lượng Lap tăng lên, người dùng có thể cuộn để xem các mốc thời gian cũ.

---

# 7. Ý tưởng triển khai NC3 - Đổi màu con số và rung khi Đặt lại

## 7.1. Mục tiêu

NC3 gồm hai yêu cầu:

1. Đổi màu `tvTime` khi stopwatch vượt quá **60 giây**.
2. Tạo một nhịp rung nhẹ khi người dùng bấm **Đặt lại**.

## 7.2. Đổi màu khi vượt 60 giây

Phần đổi màu được tích hợp ngay trong hàm cập nhật thời gian.

Ý tưởng là sau khi tính được số mili-giây hiện tại:

```java
long ms = elapsed();
```

chương trình kiểm tra:

```java
if (ms > 60_000L) {
    tvTime.setTextColor(Color.RED);
} else {
    tvTime.setTextColor(normalTimeColor);
}
```

Do `updateTimeText()` được gọi liên tục bởi ticker, màu của đồng hồ tự động thay đổi ngay khi thời gian vượt ngưỡng 60 giây.

Màu ban đầu được lưu lại bằng:

```java
normalTimeColor = tvTime.getCurrentTextColor();
```

nhằm bảo đảm sau khi reset, màu có thể trở về đúng màu ban đầu của giao diện.

## 7.3. Rung khi Đặt lại

Chức năng rung được tách thành một hàm riêng, ví dụ:

```java
private void vibrateReset() {
    // khởi tạo Vibrator/VibratorManager
    // thực hiện một nhịp rung ngắn
}
```

Khi người dùng bấm **Đặt lại**, hàm này được gọi sau khi reset trạng thái stopwatch.

Cách tách chức năng rung thành một hàm riêng giúp phần xử lý `resetStopwatch()` vẫn rõ ràng và dễ đọc.

Để tương thích với nhiều phiên bản Android, code xử lý cả API cũ và API mới khi lấy đối tượng rung.

## 7.4. Kết quả người dùng nhìn thấy

Khi stopwatch chạy:

```text
00:59.x  -> màu bình thường
01:00.x  -> chuyển sang màu đỏ
```

Khi bấm **Đặt lại**:

```text
00:00.0
```

đồng thời thiết bị rung nhẹ và danh sách Lap được xóa theo thiết kế của project.

---

# 8. Cấu trúc chức năng chính

```text
MainActivity.java
│
├── Logic stopwatch
│   ├── elapsed()
│   ├── startStopwatch()
│   ├── pauseStopwatch()
│   └── resetStopwatch()
│
├── Handler / Runnable
│   ├── startTicking()
│   └── stopTicking()
│
├── Giao diện
│   ├── updateTimeText()
│   └── updateUi()
│
├── Vòng đời Activity
│   ├── onStart()
│   ├── onResume()
│   ├── onPause()
│   ├── onStop()
│   ├── onRestart()
│   └── onDestroy()
│
├── Lưu trạng thái
│   ├── onSaveInstanceState()
│   └── onRestoreInstanceState()
│
├── NC1 - Lap
│   ├── addLap()
│   └── updateLapsText()
│
└── NC3
    └── vibrateReset()
```

## 9. Kiểm thử đề xuất

Các kịch bản quan trọng:

1. Bắt đầu → Tạm dừng → tiếp tục → kiểm tra thời gian.
2. Đang chạy → xoay màn hình → kiểm tra thời gian không về 0.
3. Tạm dừng → xoay màn hình → kiểm tra trạng thái và thời gian vẫn giữ nguyên.
4. Bật **Không giữ hoạt động** → đưa app xuống nền → quay lại → kiểm tra trạng thái được khôi phục.
5. Nhấn Back để thoát hẳn → mở lại app → kiểm tra stopwatch trở về trạng thái ban đầu.
6. Bật stopwatch → tạo nhiều Lap → xoay màn hình → kiểm tra danh sách Lap vẫn còn.
7. Chạy vượt 60 giây → kiểm tra `tvTime` đổi màu.
8. Bấm Đặt lại → kiểm tra đồng hồ reset và thiết bị rung.

## 10. Thông tin sinh viên

- **Họ tên:** `<HỌ_TÊN>`
- **MSSV:** `<MSSV>`
- **Lớp:** `<LỚP>`
- **Học phần:** INT4211 - Lập trình trên các thiết bị di động
- **Bài lab:** Lab A2


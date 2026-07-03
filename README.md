# room-firmware

Kho phân phối firmware nhị phân cho thiết bị ESP32 qua OTA. Repository không chứa source firmware chính; nó lưu các bản `firmware.bin`, manifest và metadata phiên bản để dashboard hoặc thiết bị xác định bản đang hoạt động.

## Cấu trúc repository

```text
version.json                    # Metadata phiên bản ở root
releases/
  manifest.json                 # Danh sách/thông tin bản phát hành
  latest/firmware.bin           # Binary được xem là bản mới nhất
  v1.0.1/firmware.bin
  v1.0.2/firmware.bin
  v1.1.0/firmware.bin
  v1.1.1/firmware.bin
  v2.0.0/firmware.bin
  v2.0.1/firmware.bin
```

## Vai trò trong hệ thống

Repository này có thể được dùng cùng `DeviceManager`: quản trị viên upload firmware, cập nhật manifest và kích hoạt một version. ESP32 đọc metadata từ endpoint/raw file, so sánh với version hiện tại rồi tải binary tương ứng.

```text
Build firmware -> tạo version -> upload binary -> cập nhật manifest
       -> kích hoạt -> thiết bị kiểm tra -> tải -> xác minh -> cập nhật
```

## Quy ước version

Nên dùng Semantic Versioning `MAJOR.MINOR.PATCH`:

- `PATCH`: sửa lỗi tương thích, không đổi protocol.
- `MINOR`: thêm chức năng nhưng vẫn tương thích ngược.
- `MAJOR`: thay đổi protocol, cấu hình hoặc cấu trúc có thể không tương thích.

Tên thư mục cần có tiền tố `v`, ví dụ `v2.0.2`. `latest` chỉ là con trỏ/bản sao tiện dụng; thiết bị nên lưu version cụ thể để hỗ trợ rollback và audit.

## Quy trình phát hành đề xuất

1. Build firmware bằng đúng board profile và partition table của thiết bị.
2. Chạy thử trên ít nhất một thiết bị staging.
3. Tính SHA-256 và lưu checksum trong manifest.
4. Tạo `releases/vX.Y.Z/firmware.bin`.
5. Cập nhật `manifest.json` và `version.json` theo cùng version.
6. Chỉ cập nhật `latest/firmware.bin` sau khi binary đã được xác minh.
7. Commit với thông điệp rõ ràng và kích hoạt dần trên một nhóm thiết bị nhỏ.

## Yêu cầu phía thiết bị

Firmware OTA nên:

- Gửi version hiện tại và định danh board khi kiểm tra cập nhật.
- Chỉ tải bản tương thích với model/partition hiện tại.
- Dùng HTTPS và kiểm tra status/Content-Length.
- Xác minh checksum hoặc chữ ký trước khi flash.
- Không cập nhật khi nguồn yếu hoặc kết nối không ổn định.
- Ghi log kết quả và tự rollback nếu boot bản mới thất bại.

## Ví dụ manifest khuyến nghị

```json
{
  "version": "2.0.2",
  "file": "releases/v2.0.2/firmware.bin",
  "sha256": "<checksum>",
  "board": "esp32doit-devkit-v1",
  "minVersion": "1.1.0",
  "mandatory": false,
  "notes": "Mô tả thay đổi"
}
```

Đây là schema khuyến nghị; cần giữ tương thích với format mà firmware và `DeviceManager` thực sự đang đọc.

## Rollback

Giữ ít nhất một bản ổn định trước đó. Nếu tỷ lệ cập nhật lỗi tăng, ngừng kích hoạt bản mới, trỏ manifest về version ổn định và điều tra log. Với thiết bị hỗ trợ OTA hai partition, đánh dấu firmware mới hợp lệ chỉ sau khi hoàn thành self-check.

## Bảo mật

- Binary public có thể bị tải và phân tích; không nhúng secret dài hạn.
- Nên ký firmware và xác minh chữ ký trên thiết bị.
- Giới hạn quyền ghi repository/dashboard.
- Không thay thế nội dung một version cũ; phát hành version mới để audit được.

## Trạng thái hiện tại

Repository đã có nhiều binary version nhưng chưa công khai source build, checksum, board model hoặc changelog. Trước khi dùng production, cần bổ sung các thông tin trên và xác minh rằng mọi thiết bị đều có cơ chế rollback an toàn.

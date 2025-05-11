File mẫu Google Sheet: https://docs.google.com/spreadsheets/d/1yiI3ESBBseaVRODSVeNfix_VGyLbng6iUfU7OsM_58M/edit?usp=sharing

File mẫu Google Drive: https://drive.google.com/drive/folders/1pRZ0vRKVPp5CJfKe66-ZOzxve1Sw-yJp?dmr=1&ec=wgc-drive-hero-goto
Để giải quyết vấn đề này và cho phép script "tiếp tục" từ nơi nó dừng lại khi bạn chạy lại, chúng ta cần một cơ chế để "ghi nhớ" những file video nào trong Drive đã được xử lý.

Giải pháp:

Sử dụng PropertiesService: Google Apps Script cung cấp PropertiesService cho phép lưu trữ dữ liệu dạng key-value đơn giản giữa các lần chạy script. Chúng ta sẽ dùng UserProperties để lưu ID của các file video trên Drive đã được kiểm tra (dù cập nhật thành công hay không tìm thấy STT tương ứng trong Sheet).
Kiểm tra trước khi xử lý: Khi script chạy, trước khi xử lý một file video từ Drive, nó sẽ kiểm tra xem ID của file đó đã có trong danh sách đã xử lý chưa. Nếu có, nó sẽ bỏ qua file đó và tiếp tục với file tiếp theo.
Cập nhật danh sách đã xử lý: Sau khi kiểm tra một file, ID của file đó sẽ được thêm vào danh sách đã xử lý và lưu lại vào UserProperties.
Hàm xóa trạng thái (Tùy chọn): Tôi sẽ cung cấp thêm một hàm nhỏ để bạn có thể xóa danh sách các file đã xử lý nếu muốn chạy lại toàn bộ từ đầu.
overwriteExistingLinks = false: Vẫn giữ cài đặt này để đảm bảo script không ghi đè lên các ô đã có link đúng, giúp tiết kiệm thời gian và số lần ghi vào Sheet.
Dưới đây là phiên bản script đã được cập nhật với logic "resumable" này.

ID thư mục Google Drive của bạn: 1MyKHIpZyRjF90hOubpJ3nHab_QRujI9S

Hướng dẫn sử dụng:

Mở file Google Sheet của bạn.
Đi tới menu Tiện ích mở rộng (Extensions) -> Apps Script.
Xóa hết code cũ trong trình soạn thảo (Code.gs).
Sao chép (Copy) toàn bộ đoạn mã dưới đây và Dán (Paste) vào.
Nhấn Lưu dự án (Save project) (hình đĩa mềm).
Lần đầu tiên hoặc khi muốn quét lại toàn bộ:
Chọn hàm clearProcessedVideoIds từ menu thả xuống.
Nhấn Chạy (Run). Thao tác này sẽ xóa bộ nhớ về các video đã xử lý.
Để chạy/tiếp tục quá trình cập nhật:
Chọn hàm updateLinksFromDriveToSheet_Resumable từ menu thả xuống.
Nhấn nút Chạy (Run).
Cấp quyền cho script nếu được yêu cầu lần đầu.
Nếu script bị dừng do hết thời gian, bạn chỉ cần chạy lại hàm updateLinksFromDriveToSheet_Resumable. Nó sẽ tự động bỏ qua những video đã kiểm tra và tiếp tục từ những video chưa được xử lý.
Quan sát kết quả trong Sheet và hộp thoại thông báo. Xem log chi tiết qua menu Xem (View) -> Nhật ký (Logs).

Những thay đổi chính trong phiên bản này:

PROCESSED_IDS_KEY: Một hằng số được định nghĩa để làm key lưu trữ trong UserProperties.
Đọc và Ghi UserProperties:
Khi hàm updateLinksFromDriveToSheet_Resumable bắt đầu, nó đọc danh sách ID các file đã xử lý từ UserProperties.
Trước khi xử lý một file, nó kiểm tra xem ID file đó có trong danh sách đã xử lý không. Nếu có, bỏ qua.
Sau khi một file được kiểm tra (dù có cập nhật link hay không), ID của nó được thêm vào Set các ID đã xử lý.
Set này được lưu lại vào UserProperties định kỳ (ví dụ sau mỗi 20 file) và một lần nữa ở cuối script để đảm bảo trạng thái được cập nhật.
Hàm clearProcessedVideoIds():
Hàm này cho phép bạn xóa key PROCESSED_IDS_KEY khỏi UserProperties.
Bạn nên chạy hàm này nếu muốn script bắt đầu quét lại toàn bộ thư mục Drive từ đầu, bỏ qua những gì nó đã "nhớ" từ các lần chạy trước.
Log và Thông báo: Các thông báo và log đã được cập nhật để phản ánh logic mới, bao gồm số file được bỏ qua vì đã xử lý, tổng số ID đã xử lý qua tất cả các lần chạy.
Kiểm tra thời gian: Thời gian dừng được điều chỉnh thành 5.5 phút (330 giây) để có thêm một chút "buffer" trước giới hạn 6 phút.
Cách chạy hiệu quả với số lượng lớn video:

Chạy clearProcessedVideoIds() MỘT LẦN (nếu bạn muốn bắt đầu lại từ đầu hoặc đây là lần đầu tiên chạy phiên bản này).
Chạy updateLinksFromDriveToSheet_Resumable().
Nếu script hiển thị thông báo "Script tạm dừng do sắp hết thời gian thực thi...", bạn chỉ cần chờ vài giây rồi chạy lại hàm updateLinksFromDriveToSheet_Resumable(). Không cần chạy lại hàm clear... nữa.
Lặp lại bước 3 cho đến khi thông báo cuối cùng cho biết "Hoàn thành lần chạy!" và bạn thấy rằng tổng số ID đã xử lý khớp với số lượng video bạn có (hoặc không còn video nào được cập nhật mới nữa).

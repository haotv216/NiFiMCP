---
description: Quy trình chuẩn (Workflow) khi tương tác, xây dựng và cập nhật luồng trên Apache NiFi
---

# NiFi Development Workflow

Workflow này định hình các bước tiêu chuẩn để hệ thống Agent làm việc với Apache NiFi thông qua MCP server một cách an toàn, chính xác và luôn bám sát convention hiện có của hệ thống người dùng. Workflow này có thể được kích hoạt tự động nếu người dùng có nhu cầu xử lý các luồng NiFi.

## Bước 1: Khảo sát và Đọc hiểu luồng (Discovery & Understand)
- Dùng công cụ `search_nifi_flow` hoặc `get_flow_outline` để định vị chính xác **Process Group** mà người dùng yêu cầu dựa trên Tên hoặc UUID.
- Sử dụng `document_nifi_flow` để đọc toàn bộ kiến trúc bên trong (bao gồm Processors, Connections, Ports, Controller Services, **tiến hành đối chiếu cả các Variables được định nghĩa ở cấp độ Process Group**).
- Phân tích và tóm tắt lại **luồng nghiệp vụ (business logic)**, cấu hình property, **danh sách các Variables** và các biểu thức (Expression Language - EL) đang sử dụng để hiểu rõ mục đích của luồng hiện tại.

## Bước 2: Phân tích Convention & Môi trường (Convention Analysis)
- Trích xuất các khuôn mẫu (convention) định danh đang được dùng trong Process Group (ví dụ: nguyên tắc đặt tên Processor, cấu hình Auto-Terminate đối với Connection dư thừa, cách sử dụng Funnel, cách truyền Payload, Auth Context, cấu hình Retries...).
- Trích xuất môi trường (URL Pattern, cấu trúc đường dẫn API được định nghĩa sẵn, **các Variables dùng chung được khai báo từ trước**).
- Đối chiếu những convention và Variables này làm cơ sở thiết kế cho phần xây dựng/cập nhật luồng tiếp theo.

## Bước 3: Đề xuất & Xây dựng luồng (Implementation)
- Lên kế hoạch chi tiết (Implementation Plan) trước khi gọi API tạo luồng. Chú ý lên danh sách **các Variables** bắt buộc phải được gán/kế thừa cho Process Group đích, đảm bảo các hàm Expression Language không bị lỗi khi thực thi.
- Cân nhắc **lưu snapshot dự phòng (Backup)** kiến trúc gốc nếu thực hiện các update/delete diện rộng.
- Chạy các công cụ tạo luồng (`create_nifi_process_group`, `create_nifi_processors`, `create_nifi_connections`...) để xây dựng luồng mới hoặc cập nhật luồng cũ **tuân thủ tuyệt đối** các convention đã phân tích.
- Việc tạo mới / sửa đổi chỉ được áp dụng **cách thức hoàn toàn mới (custom approach)** khi và chỉ khi hệ thống hiện tại chưa từng có convention nào giải quyết được case/yêu cầu đó.

## Bước 4: Tự động sắp xếp giao diện (Auto-Layout)
// turbo
- Chạy công cụ `layout_nifi_process_group` ngay sau khi tạo / copy hoặc kết nối thành công các components. Bước này vô cùng quan trọng để hệ thống lưới (Canvas) NiFi không bị đè chéo component lên nhau, đảm bảo UI sạch đẹp và dễ maintain cho kỹ sư vận hành.

## Bước 5: Kiểm tra và Gỡ lỗi (Validation & Debugging)
- Sử dụng `get_process_group_status` hoặc `analyze_nifi_processor_errors` để kiểm tra trạng thái Health (sức khỏe) của luồng và hàng đợi ngay sau khi đã tạo/sửa.
- Đảm bảo tất cả các processors ở trạng thái `VALID`. Đặc biệt **kiểm tra chéo lại các Process Group Variables** xem có biến Expression nào gọi ra mà chưa được định nghĩa (hiện cảnh báo thiếu biến).
- Nếu phát báo lỗi `INVALID` (ví dụ: Relationship chưa kết nối, thiếu properties bắt buộc, undefined variable), hãy tự động đọc lỗi, tiến hành debug và tự động gỡ lỗi (fix).
- Gửi báo cáo nghiệm thu dạng Markdown ngắn gọn cho người dùng, liệt kê thay đổi hoặc các cấu hình Variables đã can thiệp, cùng danh sách các lỗi đã được sửa triệt để (nếu có).

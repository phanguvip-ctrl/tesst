graph TD

%% Actors
C((Khách hàng))
S((Hệ thống Baygolf))
B((Hệ thống Ngân hàng))
I((Hệ thống Hóa đơn điện tử))
NV((NV Sale Baygolf))
GC((Sân Golf đối tác))

%% Phase 1: Booking
C -->|Thực hiện đặt lịch| S
S -->|Tạo mới| B1[Tạo Booking Number duy nhất]

%% Modification & Cancellation
C -.->|Yêu cầu đổi lịch/Hủy| S
S -.->|Cập nhật| M1[Đổi Status Booking cũ = Cancelled]
M1 -.-> M2[Tạo Booking Number mới]
M2 -.-> M3[Map 'Original Booking Number' & Tính 'Chênh lệch giá']
M3 -.-> P_Choice

%% Phase 2: Payment Options
B1 --> P_Choice{Chọn PTTT}

%% Option A: Auto Payment
P_Choice -->|QR Code / Cổng thanh toán| P_Auto[Hiển thị QR Code cú pháp chuẩn]
C -->|Quét & Thanh toán| B
B -->|Webhook / API| S
S -->|Khớp 100% Cú pháp & Số tiền| Pay_Success[Cập nhật Status: Paid]

%% Option B: Manual Transfer
P_Choice -->|Chuyển khoản thủ công| P_Manual[Hiển thị thông tin STK Cty]
C -->|Chuyển khoản & Submit Form| S
S -->|Upload Chứng từ| OCR[Hệ thống quét OCR]
OCR --> Match{Khớp Data OCR vs Bank vs System?}

Match -->|Khớp Mã + Tiền + Thời gian| Pay_Success
Match -->|Sai lệch/Không rõ| Pay_Fail[Báo Push Notification cho NV Sale]

Pay_Fail --> NV
NV -->|Kiểm tra thủ công SMS/Bank| NV_Action[Manual Update Status: Paid]
NV_Action --> AuditLog[(Ghi Log: Ai, Khi nào, Làm gì)]
AuditLog --> Pay_Success

%% Phase 3: Invoicing
Pay_Success --> Inv_Check{Tùy chọn Hóa đơn?}
Inv_Check -->|Có Check| Inv_Corp[Lấy Data: MST, Tên, Đ/C]
Inv_Check -->|Không Check| Inv_Retail[Gắn nhãn: KHÁCH LẺ KHÔNG LẤY HĐ]

Inv_Corp --> API_Inv[Gọi API xuất hóa đơn GTGT]
Inv_Retail --> API_Inv
API_Inv --> I
I -->|Trả về Mã HĐ| S

%% Phase 4: Fulfillment
Pay_Success --> F1[Hệ thống báo đơn thành công]
F1 --> NV
NV -->|Liên hệ Sân Golf| GC
GC -->|Xác nhận| S_GC[Hệ thống Sân Golf cấp Mã Booking]
NV -->|Nhập liệu| F2[Map Booking Baygolf vs Sân]
F2 --> S
S -->|Gửi xác nhận| C

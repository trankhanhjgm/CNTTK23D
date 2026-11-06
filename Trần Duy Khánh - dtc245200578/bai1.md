# Bài 1 – Tổng quan hệ thống và MEP

Họ tên: Trần Duy Khánh  
MSSV: DTC245200718  
Lớp: CNTT K23D

> Bản nhật ký cần hoàn thiện minh chứng: kiểm tra lại họ tên, lớp; thay các chỗ `[CẦN ĐIỀN]` bằng kết quả thực tế. Các câu trả lời dưới đây dựa trên đặc tả bài thực hành và code hướng dẫn. Chỉ ghi hoàn thành sau khi chạy thành công trên máy.

## 1. Cài đặt

Môi trường yêu cầu gồm .NET 10 SDK, Git, Visual Studio Code và các extension C# Dev Kit, REST Client. Máy không có ổ D nên sử dụng thư mục `C:\Code_demo`.

Phiên bản .NET thực tế: [CẦN ĐIỀN từ dotnet --version].  
Phiên bản Git thực tế: [CẦN ĐIỀN từ git --version].  
Kết quả chạy HelloPtudpt: [CẦN ĐIỀN sau khi chạy].

Ảnh minh chứng: lưu ảnh kiểm tra tại `docs/images/01-cai-dat.png`.

<!-- Bỏ dấu chú thích ở dòng dưới khi đã lưu ảnh đúng tên. -->
<!-- ![Kiểm tra môi trường](images/01-cai-dat.png) -->

## 2. MepDemo (Bước 7)

### Mỗi bài nộp sub-x được mấy worker xử lý?

Trong mô phỏng MEP 3, mỗi bài nộp chỉ do một worker xử lý. Các worker cùng lấy công việc từ hàng đợi và chia nhau xử lý, không phải mỗi worker đều xử lý lại toàn bộ bài nộp. Đây là kết quả mong đợi của mô phỏng; không đồng nghĩa mọi hệ thống hàng đợi thực tế luôn bảo đảm không giao trùng thông điệp.

### Một sự kiện được gửi tới mấy dịch vụ?

Trong mô phỏng MEP 4, một sự kiện được gửi tới cả 3 dịch vụ đã đăng ký nhận sự kiện.

### Mô tả 5 mô hình trao đổi thông điệp

| Mô hình | Mô tả |
|---|---|
| Request–response | Bên gửi gửi yêu cầu và nhận phản hồi từ bên xử lý. |
| One-way | Bên gửi gửi thông điệp mà không chờ phản hồi ở mức ứng dụng. |
| Queue-based | Thông điệp được đưa vào hàng đợi để các worker chia nhau xử lý. |
| Publish–subscribe | Bên phát công bố sự kiện để các bên đăng ký phù hợp cùng nhận. |
| Streaming | Dữ liệu được truyền thành luồng để bên nhận xử lý dần. |

Quan sát thực tế: [CẦN ĐIỀN sau khi chạy MepDemo].

<!-- ![MEP 3](images/02-mep3.png) -->
<!-- ![MEP 4](images/03-mep4.png) -->

## 3. BasicWebApi (Bước 8)

### Kết quả kiểm thử

| Request | Kết quả mong đợi | Kết quả thực tế |
|---|---|---|
| GET / | 200; application là BasicWebApi | [CẦN ĐIỀN] |
| GET /hello/An | 200; lời chào mặc định | [CẦN ĐIỀN] |
| GET /hello/An?language=en | 200; Hello, An! | [CẦN ĐIỀN] |
| POST /greetings, tên Binh | 201; có header Location | [CẦN ĐIỀN] |
| POST /greetings, tên rỗng | 400; Name must not be empty. | [CẦN ĐIỀN] |
| GET /hello/An?language=ja | 200; Konnichiwa, An! sau khi bổ sung code | [CẦN ĐIỀN] |

<!-- ![BasicWebApi 200](images/04-basic-200.png) -->
<!-- ![BasicWebApi 201](images/05-basic-201.png) -->
<!-- ![BasicWebApi 400](images/06-basic-400.png) -->

### Trả lời câu hỏi mục 8.4

**1. AddSingleton hay Build phải đứng trước?**

`builder.Services.AddSingleton(...)` phải đứng trước `builder.Build()`. Cần đăng ký dịch vụ trước khi xây dựng ứng dụng và bộ cung cấp dịch vụ để endpoint có thể nhận dịch vụ qua cơ chế dependency injection.

**2. name và language lấy từ đâu?**

Trong `/hello/An?language=en`, `name` lấy từ phần route, có giá trị `An`; `language` lấy từ query string, có giá trị `en`.

**3. Vì sao POST hợp lệ trả 201?**

Vì yêu cầu tạo ra một tài nguyên mới. Mã 201 Created thể hiện rõ việc tạo thành công, còn header Location chỉ địa chỉ tài nguyên vừa tạo. Mã 200 là thành công nói chung.

### Bổ sung tiếng Nhật và debug

Kết quả bổ sung `language=ja`: [CẦN ĐIỀN].

Đặt breakpoint tại dòng `var greeting = service.Create(...)`, gửi lại POST; quan sát biến request. F10 chạy từng dòng, F5 chạy tiếp, Shift + F5 dừng debug.

Giá trị request quan sát được khi debug: [CẦN ĐIỀN].

<!-- ![Debug tại breakpoint](images/07-debug.png) -->

## 4. MiniDistributedSystem (Bước 9)

API và Client chạy trong hai tiến trình riêng, giao tiếp bằng HTTP.

<!-- ![Hai terminal API và Client](images/08-mini-api-client.png) -->

**1. Client gọi endpoint nào?**

| Method | URL mặc định | Phản hồi khi hợp lệ |
|---|---|---|
| GET | http://localhost:5000/api/assignments | 200, danh sách bài tập |
| POST | http://localhost:5000/api/submissions | 201, mã lượt nộp và trạng thái received |

Mỗi lần gọi gồm một yêu cầu của Client và một phản hồi của API.

**2. Hai bên có dùng chung code không?**

Hai project không tham chiếu nhau. Mỗi bên khai báo record riêng và thống nhất qua hợp đồng API: method, đường dẫn, cấu trúc JSON, kiểu dữ liệu và mã trạng thái.

**3. Vì sao truyền địa chỉ API qua tham số?**

Để thay máy chủ hoặc cổng mà không sửa và biên dịch lại code. Ví dụ có thể truyền `http://localhost:5050` khi API đổi cổng. Giá trị mặc định chỉ được dùng khi không truyền tham số.

Kết quả thực tế và ID lượt nộp: [CẦN ĐIỀN].

## 5. Thí nghiệm lỗi (Bước 10)

Các dự đoán dưới đây là gợi ý trước khi chạy. Cột thực tế và đúng/sai phải được điền sau khi thực hiện trên MiniDistributedSystem. Sau mỗi thí nghiệm, trả code về trạng thái ban đầu.

| # | Dự đoán | Thực tế | Đúng/Sai | Giải thích |
|---|---|---|---|---|
| a. Thêm bài tập thứ ba | Client in thêm bài tập thứ ba mà không cần sửa code Client | [CẦN ĐIỀN] | [CẦN ĐIỀN] | Client nhận danh sách và duyệt từng phần tử; số lượng thay đổi nhưng cấu trúc dữ liệu vẫn như cũ. |
| b. Tắt API rồi chạy Client | Client gặp HttpRequestException do không kết nối được máy chủ | [CẦN ĐIỀN] | [CẦN ĐIỀN] | API là tiến trình độc lập và đã dừng. Gọi HTTP phụ thuộc máy chủ và kết nối. Gọi hàm nội bộ không có lỗi kết nối mạng này, nhưng vẫn có thể lỗi logic hoặc dữ liệu. |
| c. Để StudentCode rỗng | API trả 400; EnsureSuccessStatusCode() phát sinh exception | [CẦN ĐIỀN] | [CẦN ĐIỀN] | Máy chủ phải tự kiểm tra dữ liệu, vì client có thể gửi dữ liệu sai. EnsureSuccessStatusCode chỉ chấp nhận mã thành công 2xx. |
| d. API ở 5050, Client gọi 5000 | Khi không có dịch vụ ở 5000, Client không kết nối được; đổi tham số sang 5050 thì chạy được | [CẦN ĐIỀN] | [CẦN ĐIỀN] | Chạy Client với -- http://localhost:5050 để khớp địa chỉ API, không sửa code. |

<!-- ![Thí nghiệm a](images/09-loi-a.png) -->
<!-- ![Thí nghiệm b](images/10-loi-b.png) -->
<!-- ![Thí nghiệm c](images/11-loi-c.png) -->
<!-- ![Thí nghiệm d](images/12-loi-d.png) -->

## 6. Dự án EduSubmit (Bước 11–12)

Tên dự án: `EduSubmit-DTC245200718`.

Theo ảnh terminal đã kiểm tra, hai project Submission.Api và Submission.Client đã build thành công trên .NET 10. Các kết quả chạy chức năng dưới đây cần bổ sung từ lần thực hành.

| Chức năng | Kết quả mong đợi | Kết quả thực tế |
|---|---|---|
| GET /api/assignments | 200, hai bài tập | [CẦN ĐIỀN] |
| POST hợp lệ | 201, có GUID, received, ReceivedAt và Location | [CẦN ĐIỀN] |
| POST thiếu dữ liệu hoặc AssignmentId ≤ 0 | 400 | [CẦN ĐIỀN] |
| GET /api/submissions/{id} có thật | 200, đúng lượt nộp đã tạo | [CẦN ĐIỀN] |
| GET /api/submissions/{id} không tồn tại | 404 | [CẦN ĐIỀN] |
| Client đọc lại lượt nộp | In lại cùng ID vừa tạo | [CẦN ĐIỀN] |

Luồng thông điệp:

1. Client gửi GET /api/assignments; API trả danh sách bài tập — request–response.
2. Client gửi POST /api/submissions kèm JSON; API kiểm tra, tạo lượt nộp và trả kết quả — request–response.
3. Client gửi GET /api/submissions/{id}; API tìm và trả lượt nộp hoặc 404 — request–response.

Bài mở rộng lưu dữ liệu trong List ở RAM. Khi khởi động lại API, các lượt nộp cũ mất; cần nộp lại để có ID mới.

Phần đã tham khảo hướng dẫn/code mẫu: [CẦN ĐIỀN trung thực phần đã tham khảo; nếu dùng code do ChatGPT hỗ trợ thì ghi rõ].

<!-- ![EduSubmit API và Client](images/13-edusubmit.png) -->
<!-- ![Đọc lượt nộp 200](images/14-submission-200.png) -->
<!-- ![Không tìm thấy 404](images/15-submission-404.png) -->

## 7. Khó khăn gặp phải và cách giải quyết

Máy không có ổ D nên lệnh chuyển tới D:\Code_demo báo lỗi. Đã chuyển sang sử dụng C:\Code_demo.

Khi dùng Read-Host, đã nhập nhầm lệnh mkdir vào chỗ nhập MSSV khiến tên thư mục chứa ký tự không hợp lệ. Đã sửa bằng cách tạo trực tiếp thư mục EduSubmit-DTC245200718.

Hiện chỉ nhận được MepDemo từ giảng viên nên chưa có đủ bộ mẫu BasicWebApi và MiniDistributedSystem như tài liệu mô tả. Cách bổ sung và phần đã thực hành: [CẦN ĐIỀN đúng tình trạng thực tế].

Khó khăn khác và cách xử lý: [CẦN ĐIỀN nếu có].

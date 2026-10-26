# Ghi chú về Alarm, Trip, Failure và Repose

Nguồn phân tích: `Modbus List.pdf`, tài liệu AQA10036557B của Daihatsu Diesel.

## Quy ước trạng thái Modbus

Các tín hiệu số trong tài liệu sử dụng:

- `0` (`OFF`): trạng thái không kích hoạt, thường là `Normal`.
- `1` (`ON`): trạng thái được kích hoạt, ví dụ `Alarm`, `Trip`, `Failure` hoặc `Repose`, tùy từng địa chỉ.

## Ý nghĩa các trạng thái

### Alarm

`Alarm` là cảnh báo một thông số vận hành hoặc thiết bị đang bất thường. Alarm yêu cầu người vận hành kiểm tra nhưng thông thường không tự động dừng động cơ.

Ví dụ:

- `10003`: áp suất dầu nhiên liệu thấp.
- `10008`: nhiệt độ nước làm mát đầu ra cao.
- `10015`: cảnh báo sơ bộ của bộ dò sương dầu.

### Trip

`Trip` là điều kiện bảo vệ nghiêm trọng, thường dẫn đến lệnh dừng động cơ để hạn chế hư hỏng hoặc nguy hiểm.

Ví dụ:

- `10019`: quá tốc độ.
- `10020`: nhiệt độ nước làm mát đầu ra quá cao.
- `10021`: áp suất dầu bôi trơn đầu vào thấp — Stop.
- `10022`: Oil mist detector high alarm — Stop.
- `10023`: Emergency stop.
- `10024`: khởi động thất bại.

### Failure

`Failure` báo rằng cảm biến, dây dẫn, công tắc, module hoặc đường truyền đang hỏng hay không còn đáng tin cậy. Nó không nhất thiết có nghĩa là thông số vật lý của động cơ thực sự cao hoặc thấp.

Khi bit Failure bằng `1`:

- Hệ thống thường phát cảnh báo lỗi thiết bị hoặc lỗi tín hiệu.
- Giá trị analog liên quan có thể về 0, lên toàn thang, đứng yên hoặc trở nên không hợp lệ.
- HMI/PLC không nên coi giá trị đo liên quan là đáng tin cậy.
- Failure không mặc nhiên đồng nghĩa với Trip; hành động cuối cùng phụ thuộc logic DECCS.

### Repose

`Repose` là điều kiện tạm khóa hoặc ức chế một cảnh báo khi điều kiện vận hành hiện tại khiến cảnh báo đó chưa có ý nghĩa.

Ví dụ, khi động cơ dừng thì áp suất dầu bằng 0 là bình thường. Repose ngăn hệ thống phát cảnh báo áp suất dầu thấp giả.

Các tín hiệu Repose trong tài liệu:

- `10041` — #14: tốc độ động cơ ≤ 300 rpm hoặc động cơ không hoạt động.
- `10042` — #14T: tương tự #14 nhưng có thêm điều kiện timer ST4.
- `10043` — Exhaust: khóa cảnh báo nhiệt độ khí xả trong điều kiện không phù hợp để đánh giá.
- `10044` — Priming: khóa cảnh báo trong các điều kiện liên quan đến bơm mồi.
- `10045` — Starting: khóa cảnh báo khi van điện từ khởi động đang tác động và thêm 5 giây sau khi kết thúc.
- `10046` — Filter differential pressure: khóa cảnh báo chênh áp lọc khi nhiệt độ dầu đầu vào ≤ 45°C.

## Trường hợp cảm biến `10089` bị đứt dây khi động cơ đang chạy

### Các địa chỉ liên quan

| Địa chỉ | Chức năng |
|---|---|
| `10089` | `Lub. oil engine inlet pressure sensor failure` — lỗi cảm biến áp suất dầu bôi trơn đầu vào |
| `30025` | Giá trị analog áp suất dầu bôi trơn |
| `10021` | `Lub. oil inlet pressure (stop)` — Trip do áp suất dầu bôi trơn thấp |
| `10062` | `Lub. oil pressure switch (Low press) – Disconnection` — giám sát đứt dây công tắc áp suất thấp |

### Diễn biến dự kiến

Nếu chỉ dây của cảm biến analog bị đứt:

1. `10089` chuyển từ `0 — Normal` sang `1 — Failure`.
2. Giá trị tại `30025` không còn đáng tin cậy.
3. Hệ thống phát cảnh báo lỗi cảm biến.
4. Nếu công tắc áp suất thấp độc lập và mạch bảo vệ vẫn bình thường, `10021` có thể tiếp tục ở `0 — Normal`.
5. Động cơ nhiều khả năng tiếp tục chạy; `10089 = Failure` không trực tiếp chứng minh rằng áp suất dầu thực tế đang thấp.

### Cấu trúc tín hiệu được suy luận từ Modbus List

```text
Cảm biến/transmitter analog
    ├── Giá trị áp suất: 30025
    └── Hỏng hoặc đứt dây: 10089

Công tắc áp suất thấp
    ├── Áp suất thấp, yêu cầu dừng: 10021
    └── Đứt dây công tắc: 10062
```

Việc tài liệu có riêng `10062` cho lỗi đứt dây công tắc áp suất thấp là dấu hiệu cho thấy `10021` có thể được tạo từ một công tắc bảo vệ độc lập, thay vì trực tiếp từ cảm biến analog `10089/30025`.

### Khi nào động cơ vẫn có thể dừng?

- Áp suất dầu thực tế thấp và công tắc bảo vệ kích hoạt `10021`.
- Logic DECCS được cấu hình fail-safe, coi mất `10089` là điều kiện dừng.
- Tín hiệu bị lỗi nhưng không được nhận dạng đúng là Failure, khiến giá trị thấp giả tác động vào logic Trip.
- Hai kênh dùng chung nguồn hoặc chung một phần mạch điện và sự cố làm mất cả hai.
- Người vận hành chủ động dừng động cơ vì không còn khả năng giám sát áp suất dầu.

## Kết luận về quan hệ `10089` và `10021`

Theo cấu trúc của Modbus List, nếu chỉ riêng cảm biến analog `10089` bị đứt dây, còn công tắc áp suất thấp và mạch `10021/10062` vẫn bình thường, thì hệ thống dự kiến báo `10089 = Failure` nhưng không nhất thiết kích hoạt `10021 = Trip`.

Đây là kết luận suy luận từ danh sách Modbus, chưa phải xác nhận tuyệt đối về logic nội bộ. Cần kiểm tra thêm tài liệu Cause & Effect, Alarm/Trip List, sơ đồ đấu dây hoặc logic DECCS để xác nhận nguồn tạo tín hiệu `10021` và hành động khi `10089` bị lỗi.

## Nguyên nhân làm động cơ dừng khi đang chạy

Nguồn bổ sung: `Start Engine.pdf`, mục 2.5 và 2.6.

### Automatic Trip được tài liệu xác nhận

| Nguyên nhân | Phạm vi giám sát | Modbus liên quan |
|---|---|---|
| Quá tốc độ động cơ | Luôn luôn | `10019` |
| Nhiệt độ nước làm mát HT quá cao | Từ `#14T` đến quá trình dừng | `10020` |
| Áp suất dầu bôi trơn thấp bất thường | Từ `#14T` đến quá trình dừng | `10021` |
| Oil Mist Detector High Alarm | Luôn luôn | `10022` |

`#14T` là trạng thái động cơ đạt khoảng 300 rpm và đã hết thời gian trễ cài đặt. Khoảng trễ giúp tránh Trip giả trong quá trình khởi động, khi áp suất dầu và các tín hiệu chưa ổn định.

### Dừng khẩn cấp bằng tay hoặc từ xa

Động cơ cũng dừng khẩn cấp khi:

- Nhấn nút Emergency Stop trên bảng điều khiển.
- Có tín hiệu Remote Emergency Stop từ thiết bị bên ngoài.

Tín hiệu Modbus liên quan là `10023`.

### Trình tự khi xảy ra Trip

1. Emergency stop solenoid valve được kích hoạt.
2. Shutdown solenoid valve cũng được kích hoạt.
3. Cơ cấu fuel cut cắt nhiên liệu đến các xy-lanh.
4. Động cơ dừng.
5. Hệ thống xuất serious-failure signal.
6. Đèn báo màu đỏ sáng và HMI hiển thị nguyên nhân.
7. Trạng thái Trip được giữ lại và không thể khởi động lại ngay.

Để reset Trip, động cơ phải dừng hoàn toàn, nguyên nhân phải được xử lý và Handle Switch phải được đưa về `OFF`.

### Dừng máy nhưng không phải Automatic Trip khi đang chạy

- Nhấn nút Stop.
- Có Remote Stop.
- Mất nguồn điện; tài liệu cho biết động cơ sẽ dừng khi mất nguồn.
- Start Failure làm máy dừng trong quá trình khởi động Remote, không phải Trip sau khi máy đã chạy ổn định.

## System Failure

`10033 — System failure` là cảnh báo tổng hợp của hệ thống DECSS:

- `0`: Normal.
- `1`: Alarm.

Nó được phân loại là Alarm, không phải Trip. Vì vậy `10033 = 1` không mặc nhiên có nghĩa rằng động cơ sẽ dừng hoặc không thể khởi động.

Các lỗi hệ thống chi tiết liên quan gồm:

- `10034`: Control module failure.
- `10035`: Safety module failure.
- `10036`: lỗi liên kết với màn hình trạng thái động cơ.
- `10037–10038`: lỗi liên kết Remote I/O.
- `10039`: lỗi liên kết LCD gauge board.

Máy có thể vẫn chạy nếu lỗi chỉ ảnh hưởng hiển thị, truyền thông hoặc một kênh không quan trọng. Máy có thể không khởi động hoặc phải dừng nếu lỗi làm mất interlock, nguồn, điều khiển nhiên liệu hoặc chức năng bảo vệ quan trọng. Cần kiểm tra các alarm chi tiết; không nên tiếp tục vận hành chỉ vì máy chưa Trip.

## Điều kiện cho phép khởi động

Nguồn: `Start Engine.pdf`, mục 2.3.1 — Startup Interlock.

Máy chỉ được phép khởi động khi tất cả điều kiện interlock sau được thỏa mãn:

1. Turning safety switch đang ON:
   - Thanh an toàn quay máy được đặt đúng vị trí.
   - Turning motor không ăn khớp với bánh răng máy.
2. Handle switch ở vị trí `RUN` khi vận hành ở chế độ Remote. Ở chế độ Local, vị trí handle không thuộc điều kiện interlock này, dù quy trình vận hành vẫn yêu cầu đưa cần về `RUN`.
3. Động cơ đang dừng hoàn toàn và không ở trạng thái vận hành.
4. Không có startup command nào đang tồn tại.
5. Không có Local Stop hoặc Remote Stop.
6. Không có Start Failure chưa được reset (`10024` phải ở trạng thái Normal).
7. Không có Emergency Stop hoặc Trip chưa được reset (`10019–10023` phải bình thường).
8. Hai hệ thống cảm biến tốc độ đều hoạt động để phát hiện tốc độ. Nếu tất cả speed pickup sensor bị lỗi (`10090`), máy có thể không đạt Ready to Start.
9. Áp suất dầu bôi trơn mồi đạt ít nhất `0,01 MPa`.

Ngoại lệ đối với điều kiện áp suất dầu mồi:

- Nếu chưa quá 24 giờ kể từ lúc áp suất giảm dưới `0,01 MPa`, hệ thống vẫn có thể cho phép khởi động.
- Nếu không priming từ 24 giờ trở lên, phải chạy priming pump trước khi khởi động.
- Nếu cảm biến áp suất dầu bôi trơn bị lỗi, tài liệu cho biết điều kiện kiểm tra áp suất này được loại khỏi startup interlock.

Khi tất cả điều kiện được thỏa mãn:

- Đèn `READY TO START` sáng.
- LCD gauge board hiển thị `READY`, nếu được trang bị.
- `10018 = 1` nghĩa là Ready; `10018 = 0` nghĩa là Not ready.

### Chuẩn bị trước khi nhấn Start

1. Thực hiện priming dầu bôi trơn.
2. Mở van khí khởi động và khí điều khiển.
3. Đưa operation lever về `RUN`.
4. Chọn đúng chế độ Local hoặc Remote.
5. Kiểm tra không có Trip hoặc alarm nghiêm trọng.
6. Xác nhận trạng thái `READY TO START` hoặc `10018 = 1`.
7. Sau đó mới nhấn Local Start hoặc đóng tín hiệu Remote Start.

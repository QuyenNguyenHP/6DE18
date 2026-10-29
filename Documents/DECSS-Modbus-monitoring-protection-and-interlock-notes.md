# Ghi chú giám sát, bảo vệ và liên động DECSS Modbus

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

Sơ đồ `Wiring Diagram.pdf` xác nhận có hai thiết bị vật lý riêng: transmitter analog `PTQ` và công tắc Trip áp suất thấp `63Q2`. Điều này củng cố rằng `10021` được tạo từ kênh công tắc bảo vệ độc lập, thay vì trực tiếp từ cảm biến analog liên quan đến `10089/30025`.

### Khi nào động cơ vẫn có thể dừng?

- Áp suất dầu thực tế thấp và công tắc bảo vệ kích hoạt `10021`.
- Logic DECCS được cấu hình fail-safe, coi mất `10089` là điều kiện dừng.
- Tín hiệu bị lỗi nhưng không được nhận dạng đúng là Failure, khiến giá trị thấp giả tác động vào logic Trip.
- Hai kênh dùng chung nguồn hoặc chung một phần mạch điện và sự cố làm mất cả hai.
- Người vận hành chủ động dừng động cơ vì không còn khả năng giám sát áp suất dầu.

## Kết luận về quan hệ `10089` và `10021`

Theo cấu trúc của Modbus List, nếu chỉ riêng cảm biến analog `10089` bị đứt dây, còn công tắc áp suất thấp và mạch `10021/10062` vẫn bình thường, thì hệ thống dự kiến báo `10089 = Failure` nhưng không nhất thiết kích hoạt `10021 = Trip`.

Sơ đồ đấu dây đã xác nhận sự tách biệt phần cứng giữa `PTQ` và `63Q2`; tuy nhiên vẫn cần Cause & Effect hoặc logic DECCS để xác nhận toàn bộ xử lý phần mềm khi một trong hai kênh bị lỗi.

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

### Giải thích thao tác reset trạng thái Emergency Stop

Nguồn: đoạn hướng dẫn tiếng Anh do người dùng cung cấp, bắt đầu bằng “To reset this status…”.

**Dịch nghĩa:** Để xóa trạng thái này, tốc độ quay của động cơ phải bằng không hoàn toàn. Đưa cần vận hành (`HANDLE SWITCH`) về vị trí `OFF`. Khi đó, trạng thái dừng khẩn cấp được hủy, van điện từ dừng khẩn cấp ngừng kích hoạt và hệ thống trở về trạng thái bình thường. Không thể khởi động lại động cơ cho đến khi hoàn thành các thao tác trên.

Trình tự và ý nghĩa:

1. **Chờ động cơ dừng hẳn (`0 rpm`).** “Naught” nghĩa là số không; động cơ còn quay theo quán tính thì chưa đáp ứng điều kiện này. Không được hiểu là chỉ cần tốc độ xuống dưới 300 rpm.
2. **Đưa `HANDLE SWITCH` về `OFF`.** Đây là thao tác reset được đoạn hướng dẫn yêu cầu. Chỉ chờ động cơ dừng mà chưa thực hiện thao tác này thì chưa hoàn tất quy trình reset.
3. **Trạng thái Emergency Stop được xóa; emergency stop solenoid valve chuyển OFF.** “Turns off” nghĩa là ngừng kích hoạt điện của solenoid; riêng câu này không xác định van cơ khí đang mở hay đóng.
4. **Hệ thống thoát trạng thái dừng khẩn cấp đang được giữ.** “Normal status is recovered” không có nghĩa là động cơ tự chạy lại hoặc mọi nguyên nhân sự cố đã được khắc phục.
5. **Chỉ thực hiện khởi động lại khi hoàn tất reset và đáp ứng các điều kiện khởi động.** Reset là điều kiện cần; vẫn phải xử lý nguyên nhân dừng và kiểm tra các interlock. Vị trí `OFF` dùng cho bước reset; vị trí cần khi khởi động thực hiện theo chế độ Local/Remote và quy trình tương ứng ở phần dưới.

```text
Emergency Stop → động cơ dừng hoàn toàn (0 rpm)
               → đưa HANDLE SWITCH về OFF
               → xóa trạng thái Emergency Stop, solenoid dừng khẩn cấp OFF
               → kiểm tra điều kiện khởi động trước khi khởi động lại
```

Đoạn trích không mô tả thao tác nhả nút Emergency Stop, xóa lệnh Emergency Stop từ xa hay ngưỡng phát hiện zero-speed của bộ điều khiển; không suy ra các chi tiết đó chỉ từ đoạn này.

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

### Hiểu đơn giản về `READY TO START`

`READY TO START` tự động bật khi máy đã an toàn và sẵn sàng khởi động. Người vận hành không bật trực tiếp trạng thái này.

Kiểm tra nhanh:

1. Động cơ đã dừng hoàn toàn.
2. Turning bar/turning gear đã tháo hoặc nhả và turning safety switch đang ON.
3. Không còn lệnh Start, Local Stop hoặc Remote Stop.
4. Start Failure, Emergency Stop và các Trip đã được xử lý và reset.
5. Cảm biến tốc độ hoạt động bình thường.
6. Khi chạy Remote, Handle Switch ở vị trí `RUN`.
7. Áp suất dầu bôi trơn mồi đạt ít nhất `0,01 MPa`, trừ trường hợp ngoại lệ 24 giờ được DECSS cho phép.

Khi tất cả điều kiện trên đạt:

- Đèn `READY TO START` sáng.
- Màn hình hiển thị `READY`.
- Modbus `10018 = 1` (`Ready`).

Nếu còn bất kỳ điều kiện nào chưa đạt, `10018 = 0` (`Not ready`) và máy không được phép khởi động.

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

## Thông tin quan trọng từ Wiring Diagram

Nguồn: `Wiring Diagram.pdf` — Wiring Diagram 6DE-18, Parts No. `AQA10036419`, revision B, ngày 2022-08-01. Sơ đồ là một sheet tổng hợp và ghi rõ trạng thái tiếp điểm được vẽ là trạng thái động cơ chạy bình thường.

### Hai cấp giám sát áp suất dầu bôi trơn

Sơ đồ xác nhận hai thiết bị độc lập:

| Mark | Thiết bị | Chức năng | Ngưỡng |
|---|---|---|---|
| `PTQ` | Lub. oil engine inlet pressure transmitter | Hiển thị và Alarm analog | Dưới `0,35 MPa` |
| `63Q2` | Lub. oil engine inlet pressure switch | Emergency Stop | Dưới `0,30 MPa` |

`PTQ` là transmitter hai dây riêng, còn `63Q2` là pressure switch có tiếp điểm riêng. Vì vậy:

- Hỏng hoặc đứt dây `PTQ` dự kiến tác động kênh sensor failure `10089` và làm giá trị `30025` không đáng tin cậy.
- Trip áp suất dầu thấp `10021` nhiều khả năng đến từ `63Q2`.
- Giám sát đứt dây pressure switch `10062` liên quan đến mạch `63Q2`.
- Chỉ hỏng `PTQ` không trực tiếp đóng tiếp điểm Trip của `63Q2`; động cơ chỉ Trip nếu logic DECCS có xử lý fail-safe bổ sung hoặc kênh `63Q2` cũng tác động.

### Hai cấp giám sát nhiệt độ nước làm mát HT

| Mark | Thiết bị | Chức năng | Ngưỡng |
|---|---|---|---|
| `TRWHO` | HTCW engine outlet temperature sensor | Hiển thị và Alarm | Trên `90°C` |
| `26WH2` | HTCW engine outlet temperature switch | Emergency Stop | Trên `95°C` |

Đây cũng là cấu trúc hai kênh: cảm biến analog cho hiển thị/cảnh báo và công tắc riêng cho Emergency Stop. `26WH2` là tiếp điểm thường mở; khi nhiệt độ trên 95°C thì tiếp điểm `1–3` ON.

### Bảng tính trạng thái cho các điểm analog đang sử dụng

Bảng này cũng được tách thành tài liệu riêng: [`DECSS-Modbus-analog-condition-table.md`](DECSS-Modbus-analog-condition-table.md).

Mục tiêu của bảng này là đưa mỗi điểm analog về ba trạng thái hiển thị:

- `Normal`: giá trị nằm trong vùng bình thường, hoặc máy chưa ở trạng thái cần đánh giá điểm đó.
- `Warning`: tương đương `Alarm` trong tài liệu Daihatsu; cần kiểm tra nhưng chưa phải lệnh dừng máy.
- `Critical`: điều kiện đã tạo `Trip/Emergency Stop` và có thể làm dừng động cơ.

#### Thứ tự ưu tiên khi tính

```text
1. Sensor Failure = 1  → INVALID/FAILURE, không dùng giá trị để tính Normal/Warning/Critical
2. Trip bit = 1        → Critical
3. Alarm bit = 1       → Warning
4. Không có bit Alarm  → so sánh giá trị analog với ngưỡng Warning đã xác nhận
5. Các trường hợp khác → Normal
```

`INVALID/FAILURE` là chất lượng dữ liệu, không phải trạng thái vận hành thứ tư. Khi sensor failure bằng `1`, HMI nên hiển thị `Failure/Bad data` thay vì gán giá trị analog vào một trong ba trạng thái.

Giá trị kỹ thuật được tính trước khi so sánh:

```text
Value = Raw Modbus × Reg
```

Ví dụ, các kênh áp suất có `Reg = 0,01`: raw `35` tương ứng `0,35 MPa`. Cần xác nhận thư viện Modbus chưa scale sẵn để tránh nhân hệ số hai lần.

#### Các tín hiệu dùng làm điều kiện khi không thu thập Repose

| Tên ngắn | Điều kiện |
|---|---|
| `RUN` | `10017 = 1` — động cơ đang chạy |
| `READY` | `10018 = 1` — đủ điều kiện khởi động |
| `ENGINE_ACTIVE` | `RUN = 1` AND `30026 > 300 rpm` |
| `EXH_ACTIVE` | `ENGINE_ACTIVE = 1` AND average cylinder exhaust temperature `> 200°C` |
| `PRESTART` | `READY = 1` AND `RUN = 0` |
| `OVERSPEED_TRIP` | `10019 = 1` — Trip quá tốc; `10019 = 0` — tốc độ chưa kích hoạt Trip quá tốc |
| `HTCW_HIGH_ALARM` | `10008 = 1` — Alarm nhiệt độ nước làm mát HT đầu ra cao |
| `HTCW_HIGH_TRIP` | `10020 = 1` — Trip/Stop do nhiệt độ nước làm mát HT đầu ra quá cao |

Quy ước tên các bit còn lại trong bảng:

| Address | Tên dễ đọc |
|---:|---|
| `10003` | `FO_PRESS_LOW_ALARM` — Alarm áp suất dầu nhiên liệu đầu vào thấp |
| `10004` | `LO_PRESS_LOW_ALARM` — Alarm áp suất dầu bôi trơn đầu vào thấp |
| `10005` | `HTCW_PRESS_LOW_ALARM` — Alarm áp suất nước làm mát HT đầu vào thấp |
| `10006` | `LTCW_PRESS_LOW_ALARM` — Alarm áp suất nước làm mát LT đầu vào thấp |
| `10011` | `START_AIR_PRESS_LOW_ALARM` — Alarm áp suất khí khởi động thấp |
| `10021` | `LO_PRESS_LOW_TRIP` — Trip/Stop do áp suất dầu bôi trơn đầu vào thấp |
| `10022` | `OIL_MIST_HIGH_TRIP` — Trip/Stop do Oil Mist Detector High Alarm |
| `10023` | `EMERGENCY_STOP_TRIP` — Emergency Stop tại chỗ hoặc từ xa |
| `10049–10054` | `CYL1–CYL6_EXH_DEV_ALARM` — Alarm sai lệch nhiệt độ khí xả xy-lanh 1–6 |
| `10067` | `LO_IN_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ dầu bôi trơn đầu vào |
| `10068` | `HTCW_OUT_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ nước HT đầu ra |
| `10070–10075` | `CYL1–CYL6_EXH_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ khí xả xy-lanh 1–6 |
| `10078` | `TC_IN1_EXH_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ khí xả vào T/C 1 |
| `10079` | `TC_IN2_EXH_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ khí xả vào T/C 2 |
| `10080` | `TC_OUT_EXH_TEMP_SENSOR_FAILURE` — lỗi cảm biến nhiệt độ khí xả ra T/C |
| `10081` | `HTCW_IN_PRESS_SENSOR_FAILURE` — lỗi cảm biến áp suất nước HT đầu vào |
| `10083` | `LTCW_IN_PRESS_SENSOR_FAILURE` — lỗi cảm biến áp suất nước LT đầu vào |
| `10084` | `START_AIR_PRESS_SENSOR_FAILURE` — lỗi cảm biến áp suất khí khởi động |
| `10085` | `FO_IN_PRESS_SENSOR_FAILURE` — lỗi cảm biến áp suất dầu nhiên liệu đầu vào |
| `10089` | `LO_IN_PRESS_SENSOR_FAILURE` — lỗi cảm biến áp suất dầu bôi trơn đầu vào |
| `10090` | `ALL_SPEED_PICKUP_FAILURE` — tất cả cảm biến tốc độ bị lỗi |
| `10091` | `LOAD_INPUT_FAILURE` — lỗi tín hiệu tải đầu vào |

Hệ thống thu thập hiện tại không có `10041–10046`, vì vậy bảng dưới đây không dùng bit Repose. Logic kết hợp `RUN` với tốc độ động cơ để chỉ đánh giá các thông số vận hành khi máy đã vượt `300 rpm`. Không dùng bit speed-pickup failure làm điều kiện trong phép tính này.

```text
ENGINE_ACTIVE = (10017 = 1) AND (30026 > 300 rpm)
EXH_AVG    = trung bình các giá trị hợp lệ 30006…30011
EXH_ACTIVE = ENGINE_ACTIVE AND (EXH_AVG > 200°C)
PRESTART   = (10018 = 1) AND (10017 = 0)
```

Nếu Alarm bit được thu thập thì vẫn ưu tiên bit đó vì DECSS đã áp dụng timer, hysteresis và Repose nội bộ trước khi xuất Alarm. Logic dựa trên analog chỉ là phép tính dự phòng. `READY` chỉ phù hợp để đánh giá điều kiện trước khi khởi động; không dùng `READY` thay cho `RUN` khi máy đang hoạt động.

#### Failure / Warning / Critical condition table

Only abnormal conditions are listed. Calculate the status in this order:

```text
IF Failure condition = true
    Status = Failure
ELSE IF Critical condition = true
    Status = Critical
ELSE IF Warning condition = true
    Status = Warning
ELSE
    Status = Normal
```

A dash (`—`) means that the available documentation does not define that condition for the data point.

| Analog | Name | Failure condition | Warning condition | Critical condition |
|---:|---|---|---|---|
| `30003` | Lubricating-oil inlet temperature | `10067 = 1` (`LO_IN_TEMP_SENSOR_FAILURE`) | `ENGINE_ACTIVE = 1 AND 30003 > 65°C` | — |
| `30004` | HT cooling-water outlet temperature | `10068 = 1` (`HTCW_OUT_TEMP_SENSOR_FAILURE`) | `10008 = 1` (`HTCW_HIGH_ALARM`) OR (`ENGINE_ACTIVE = 1 AND 30004 > 90°C`) | `10020 = 1` (`HTCW_HIGH_TRIP`) |
| `30006` | Cylinder 1 exhaust-gas temperature | `10070 = 1` (`CYL1_EXH_TEMP_SENSOR_FAILURE`) | `10049 = 1` (`CYL1_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30006 > 500°C`) | — |
| `30007` | Cylinder 2 exhaust-gas temperature | `10071 = 1` (`CYL2_EXH_TEMP_SENSOR_FAILURE`) | `10050 = 1` (`CYL2_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30007 > 500°C`) | — |
| `30008` | Cylinder 3 exhaust-gas temperature | `10072 = 1` (`CYL3_EXH_TEMP_SENSOR_FAILURE`) | `10051 = 1` (`CYL3_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30008 > 500°C`) | — |
| `30009` | Cylinder 4 exhaust-gas temperature | `10073 = 1` (`CYL4_EXH_TEMP_SENSOR_FAILURE`) | `10052 = 1` (`CYL4_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30009 > 500°C`) | — |
| `30010` | Cylinder 5 exhaust-gas temperature | `10074 = 1` (`CYL5_EXH_TEMP_SENSOR_FAILURE`) | `10053 = 1` (`CYL5_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30010 > 500°C`) | — |
| `30011` | Cylinder 6 exhaust-gas temperature | `10075 = 1` (`CYL6_EXH_TEMP_SENSOR_FAILURE`) | `10054 = 1` (`CYL6_EXH_DEV_ALARM`) OR (`EXH_ACTIVE = 1 AND 30011 > 500°C`) | — |
| `30014` | T/C inlet 1 exhaust-gas temperature | `10078 = 1` (`TC_IN1_EXH_TEMP_SENSOR_FAILURE`) | `EXH_ACTIVE = 1 AND 30014 > 580°C` | — |
| `30015` | T/C inlet 2 exhaust-gas temperature | `10079 = 1` (`TC_IN2_EXH_TEMP_SENSOR_FAILURE`) | `EXH_ACTIVE = 1 AND 30015 > 580°C` | — |
| `30016` | T/C outlet exhaust-gas temperature | `10080 = 1` (`TC_OUT_EXH_TEMP_SENSOR_FAILURE`) | `EXH_ACTIVE = 1 AND 30016 > 500°C` | — |
| `30017` | HT cooling-water inlet pressure | `10081 = 1` (`HTCW_IN_PRESS_SENSOR_FAILURE`) | `10005 = 1` (`HTCW_PRESS_LOW_ALARM`) OR (`ENGINE_ACTIVE = 1 AND 30017 < 0.15 MPa`) | — |
| `30019` | LT cooling-water inlet pressure | `10083 = 1` (`LTCW_IN_PRESS_SENSOR_FAILURE`) | `10006 = 1` (`LTCW_PRESS_LOW_ALARM`) OR (`ENGINE_ACTIVE = 1 AND 30019 < 0.15 MPa`) | — |
| `30020` | Starting-air pressure | `10084 = 1` (`START_AIR_PRESS_SENSOR_FAILURE`) | `10011 = 1` (`START_AIR_PRESS_LOW_ALARM`) OR (`PRESTART = 1 AND 30020 < 1.5 MPa`) | — |
| `30021` | Fuel-oil inlet pressure | `10085 = 1` (`FO_IN_PRESS_SENSOR_FAILURE`) | `10003 = 1` (`FO_PRESS_LOW_ALARM`) OR (`ENGINE_ACTIVE = 1 AND 30021 < 0.35 MPa`) | — |
| `30025` | Lubricating-oil inlet pressure | `10089 = 1` (`LO_IN_PRESS_SENSOR_FAILURE`) | `10004 = 1` (`LO_PRESS_LOW_ALARM`) OR (`ENGINE_ACTIVE = 1 AND 30025 < 0.35 MPa`) | `10021 = 1` (`LO_PRESS_LOW_TRIP`) |
| `30026` | Engine speed | `10090 = 1` (`ALL_SPEED_PICKUP_FAILURE`) | — | `10019 = 1` (`OVERSPEED_TRIP`); expected setting range: `1008–1035 rpm` |
| `30028` | Generator load | `10091 = 1` (`LOAD_INPUT_FAILURE`) | — | — |
| `30029` | Running hours | — | — | — |

#### Điểm analog không có trong danh sách 19 điểm `Use`

Các địa chỉ `30001`, `30002`, `30005`, `30012`, `30013`, `30018`, `30022`, `30023`, `30024`, `30033`, `30034`, `30035`, `30049` và các kênh option vẫn xuất hiện trong Modbus List nhưng không được đánh dấu `Use` cho cấu hình này. Không đưa chúng vào logic ba trạng thái nếu chưa xác nhận chúng được đấu dây và sử dụng thực tế.

Riêng các ngưỡng đã thấy trên wiring nhưng kênh analog không nằm trong danh sách `Use` gồm:

- `30023` áp suất dầu T/C: Warning dưới `0,06 MPa`, liên quan switch `63QT`/digital `10009`; nếu tự tính không có Repose thì chỉ đánh giá khi `ENGINE_ACTIVE`.
- `30024` chênh áp lọc dầu: Warning trên `0,09 MPa`, liên quan switch `63QDM`/digital `10001`; nếu không có `10046`, chỉ đánh giá gần đúng khi `RUN = 1` AND `30003 > 45°C`.

#### Lưu ý quan trọng về Critical

- Không tính Critical của `30004` chỉ bằng `30004 > 95°C`; trạng thái chính thức phải lấy từ `10020`, vì `26WH2` là switch Trip vật lý độc lập với transmitter `TRWHO`.
- Không tính Critical của `30025` chỉ bằng `30025 < 0,30 MPa`; trạng thái chính thức phải lấy từ `10021`, vì `63Q2` là switch Trip vật lý độc lập với transmitter `PTQ`.
- Với `30026`, tốc độ định mức là `900 rpm`. Tài liệu ghi Overspeed automatic stop được đặt trong khoảng `112–115%` tốc độ định mức:

  ```text
  900 × 112% = 1008 rpm
  900 × 115% = 1035 rpm
  ```

  Vì tài liệu chỉ cho một khoảng setting, chưa cho biết máy này đang đặt chính xác `112%`, `115%` hay một giá trị ở giữa, trạng thái Critical chính thức vẫn phải lấy từ `10019`. Không nên tự dùng `30026 ≥ 1008 rpm` làm Trip chính thức nếu chưa kiểm tra setting thực tế của Speed Switch Unit/DECSS.
- Nếu cần một trạng thái tổng cho toàn động cơ, `Critical` khi có bất kỳ bit sau bằng `1`:

  - `10019` — `OVERSPEED_TRIP`.
  - `10020` — `HTCW_HIGH_TRIP`.
  - `10021` — `LO_PRESS_LOW_TRIP`.
  - `10022` — `OIL_MIST_HIGH_TRIP`.
  - `10023` — `EMERGENCY_STOP_TRIP`.

  Nếu không Critical thì `Warning` khi có bất kỳ Alarm bit bằng `1` hoặc điều kiện Warning dự phòng trong bảng đúng; còn lại là `Normal`.

### Các ngưỡng hiển thị và Alarm

| Mark | Đại lượng | Ngưỡng Alarm |
|---|---|---|
| `TRC1–TRC6` | Nhiệt độ khí xả từng xy-lanh | Trên `500°C` |
| `TRTI(1)` | Nhiệt độ khí xả vào turbo, xy-lanh 1–3 | Trên `580°C` |
| `TRTI(2)` | Nhiệt độ khí xả vào turbo, xy-lanh 4–6 | Trên `580°C` |
| `TRTO` | Nhiệt độ khí xả ra turbo | Trên `500°C` |
| `TRQ` | Nhiệt độ dầu bôi trơn đầu vào động cơ | Trên `65°C` |
| `TRWHO` | Nhiệt độ nước làm mát HT đầu ra động cơ | Trên `90°C` |
| `PTQ` | Áp suất dầu bôi trơn đầu vào động cơ | Dưới `0,35 MPa` |
| `PTF` | Áp suất dầu nhiên liệu đầu vào động cơ | Dưới `0,35 MPa` |
| `PTAS` | Áp suất khí khởi động đầu vào động cơ | Dưới `1,5 MPa` |
| `PTWH` | Áp suất nước làm mát HT đầu vào động cơ | Dưới `0,15 MPa` |
| `PTWL` | Áp suất nước làm mát LT đầu vào động cơ | Dưới `0,15 MPa` |

Các transmitter, pressure switch và temperature sensor trong bảng được ghi là do Daihatsu cung cấp.

### Các công tắc Alarm và Emergency Stop

| Mark | Chức năng | Tác động | Trạng thái tiếp điểm quan trọng |
|---|---|---|---|
| `26WH2` | Nhiệt độ nước HT đầu ra cao | Emergency Stop trên `95°C` | Normally open; `1–3` ON khi tác động |
| `63Q2` | Áp suất dầu đầu vào thấp | Emergency Stop dưới `0,30 MPa` | Normally open; `1–2` ON khi tác động |
| `33QL` | Mức dầu sump tank thấp | Alarm | `A–B` OFF khi mức thấp |
| `33F` | Rò rỉ/vỡ ống dầu nhiên liệu cao áp | Alarm | Normally closed; `1–2` OFF khi tác động |
| `63QDM` | Chênh áp lọc dầu backwash cao | Alarm trên `0,09 MPa` | Normally closed; `1–3` OFF khi tác động |
| `63QT` | Áp suất dầu bôi trơn đầu vào turbo thấp | Alarm dưới `0,06 MPa` | Normally closed; `1–3` OFF khi tác động |

Khi kiểm tra bằng đồng hồ hoặc mô phỏng tín hiệu phải chú ý logic tiếp điểm: một số Alarm tác động bằng cách mở tiếp điểm đang đóng, không phải đóng tiếp điểm.

### Tín hiệu tốc độ, interlock và governor

- `SSE1` và `SSE2`: hai speed sensor dạng pulse, phù hợp yêu cầu phải có hai hệ thống phát hiện tốc độ để Ready to Start.
- `HS`: Handle Switch dùng cho trạng thái standby/RUN; ở vị trí RUN, tiếp điểm `2–4` ON và `1–2` OFF.
- `TC`: Turning Safety Switch; khi sử dụng turning bar, tiếp điểm `1–3` OFF. Đây là interlock chống khởi động khi dụng cụ quay máy còn gắn.
- `GM`: governor motor điều khiển tốc độ, dạng electric-control hydraulic governor, nguồn `DC24V` và yêu cầu nguồn dự phòng battery.

### Các van điện từ Start và Shutdown

| Mark | Chức năng | Hành động khi cuộn dây được cấp điện |
|---|---|---|
| `88V` | Starting solenoid valve | Thực hiện khởi động động cơ |
| `88L` | Fuel oil control solenoid valve | Khống chế nhiên liệu trong trình tự khởi động |
| `5V` | Shutdown solenoid valve — Control | Cắt nhiên liệu |
| `5S` | Shutdown solenoid valve — Safety & Control | Cắt nhiên liệu |

Các solenoid trên sử dụng `DC24V`, dòng tiêu thụ khoảng `0,7 A`. `5V` và `5S` đều được ghi rõ là **fuel shutdown when solenoid magnetized**. Việc có hai van shutdown cho thấy đường dừng có cả kênh điều khiển và kênh safety/control.

### Thiết bị phụ trợ

- `SVB`: van điện từ bypass bộ làm mát khí LT; nguồn `DC24V, 0,7 A`, phục vụ điều khiển nhiệt độ khí nạp ở tải thấp và được kích khi công suất máy phát dưới `150 kWe`.
- `BO`: blow-off valve điều khiển SCR; van mở khi cuộn dây được kích, nguồn `AC230V`, một pha, `60 Hz`, `48 W`, khoảng `300 mA`.
- `FR`: bộ phát vị trí fuel rack, tín hiệu `4–20 mA`, toàn thang `0–36 mm`.

### Lưu ý đấu dây và bảo trì

- Sơ đồ biểu diễn trạng thái chạy bình thường; không được suy luận trạng thái nghỉ chỉ từ vị trí nét vẽ mà không đọc cột Contact/Remark.
- Dây giữa thiết bị và junction box do Daihatsu thực hiện.
- Tiết diện dây tiêu chuẩn là `0,75 mm²`, trừ nơi có chỉ định khác.
- Cuộn van điện từ có diode dập xung; cần giữ đúng cực tính khi đo hoặc thay dây.
- Governor motor dùng surge killer `ERZV10D390` của Panasonic.
- Trip switch phải có điện trở giám sát `3,9 kΩ, 1/4 W`. Không được bỏ điện trở này hoặc thay sai trị số vì DECCS có thể báo disconnection/failure hoặc mất khả năng giám sát đứt dây.
- Khi đo cách điện hoặc cấp nguồn thử, phải tách các module/cảm biến điện tử và tuân thủ hướng dẫn nhà sản xuất để tránh làm hỏng transmitter, diode và surge suppressor.

## Quy tắc đọc và giải mã Tag

Hệ thống tag trong bộ tài liệu là sự kết hợp giữa quy ước đo lường/điện phổ biến và mã dịch vụ riêng của Daihatsu. Không được giả định toàn bộ tag tuân thủ tuyệt đối ISA-5.1.

Cấu trúc thường gặp:

```text
[Đại lượng hoặc loại thiết bị] + [Môi chất/hệ thống] + [Vị trí hoặc số thứ tự]
```

Khi có xung đột giữa cách giải thích theo tiêu chuẩn và nội dung bản vẽ, phải ưu tiên các cột `Name`, `Use`, `Remark` và `Contact` của Daihatsu.

### Các chữ cái thường gặp

| Ký hiệu | Ý nghĩa trong bộ tài liệu |
|---|---|
| `P` | Pressure — áp suất |
| `T` ở đầu tag | Temperature — nhiệt độ |
| `T` sau `P` | Transmitter — bộ phát tín hiệu |
| `TR` | Temperature sensor theo quy ước Daihatsu, nhiều khả năng là cảm biến điện trở |
| `S` | Switch hoặc Safety, tùy vị trí và ngữ cảnh |
| `V` | Valve — van |
| `M` | Motor |
| `F` | Fuel/fuel oil khi là hậu tố hệ thống |
| `Q` | Mã Daihatsu cho hệ thống dầu bôi trơn trong bản vẽ này |
| `W` | Water — nước làm mát |
| `H` | High-temperature circuit — vòng nhiệt độ cao |
| `L` | Low-temperature circuit — vòng nhiệt độ thấp |
| `I` | Inlet — đầu vào |
| `O` | Outlet — đầu ra |
| `AS` | Starting Air — khí khởi động |
| `TC` hoặc `T/C` trong tên hệ thống | Turbocharger; cần phân biệt với tag `TC` của Turning Safety Switch |

Cùng một chữ có thể mang nghĩa khác nhau tùy vị trí. Ví dụ `T` đầu tag thường là Temperature, nhưng `T` trong `PT` là Transmitter.

### Nhóm pressure transmitter

Cấu trúc chung:

```text
PT + hậu tố dịch vụ
P = Pressure
T = Transmitter
```

| Tag | Ý nghĩa |
|---|---|
| `PTQ` | Transmitter áp suất dầu bôi trơn đầu vào động cơ |
| `PTF` | Transmitter áp suất dầu nhiên liệu đầu vào động cơ |
| `PTAS` | Transmitter áp suất khí khởi động đầu vào động cơ |
| `PTWH` | Transmitter áp suất nước HT đầu vào động cơ |
| `PTWL` | Transmitter áp suất nước LT đầu vào động cơ |

Phần `PT` phù hợp với quy ước Pressure Transmitter phổ biến. Các hậu tố `Q`, `F`, `AS`, `WH` và `WL` là mã dịch vụ của Daihatsu. Đây là các kênh analog phục vụ hiển thị và Alarm; chúng không mặc nhiên là kênh Trip.

### Nhóm cảm biến nhiệt độ

Trong tài liệu này, `TR` được dùng cho temperature sensor. Nếu áp dụng ISA-5.1 thuần túy, `TR` có thể được hiểu là Temperature Recorder, nhưng cách hiểu đó không phù hợp với tên thiết bị trên bản vẽ. Phải sử dụng định nghĩa của Daihatsu.

| Tag | Cách giải thích |
|---|---|
| `TRC1–TRC6` | Cảm biến nhiệt độ khí xả của từng xy-lanh 1–6 |
| `TRTI(1)` | Cảm biến nhiệt độ khí xả đầu vào turbo, nhóm xy-lanh 1–3 |
| `TRTI(2)` | Cảm biến nhiệt độ khí xả đầu vào turbo, nhóm xy-lanh 4–6 |
| `TRTO` | Cảm biến nhiệt độ khí xả đầu ra turbo |
| `TRQ` | Cảm biến nhiệt độ dầu bôi trơn đầu vào động cơ |
| `TRWHO` | Cảm biến nhiệt độ nước HT đầu ra động cơ |

Ví dụ giải mã:

```text
TRWHO
TR = temperature sensor theo quy ước Daihatsu
W  = water
H  = high-temperature circuit
O  = outlet
```

```text
TRTI / TRTO
TR = temperature sensor
T  = turbocharger
I  = inlet
O  = outlet
```

### Nhóm switch và mã chức năng dạng số

| Tag | Thiết bị/chức năng |
|---|---|
| `63Q2` | Pressure switch dầu bôi trơn đầu vào; Emergency Stop dưới 0,30 MPa |
| `63QDM` | Differential-pressure switch của lọc dầu backwash |
| `63QT` | Pressure switch dầu bôi trơn đầu vào turbo |
| `26WH2` | Temperature switch nước HT đầu ra; Emergency Stop trên 95°C |
| `33QL` | Level switch báo mức dầu sump tank thấp |
| `33F` | Level/leak switch phát hiện rò rỉ hoặc vỡ ống dầu cao áp |

Ý nghĩa sử dụng của các số đầu trong sơ đồ:

| Số | Nhóm chức năng |
|---:|---|
| `26` | Thiết bị/công tắc nhiệt |
| `33` | Switch liên quan vị trí hoặc mức |
| `63` | Pressure switch |
| `88` | Thiết bị phụ trợ/van điện từ trong trình tự khởi động của hệ thống này |
| `5` | Chức năng Stop/Shutdown trong hệ thống này |

Phần chữ và số phía sau là mã dịch vụ, vị trí hoặc kênh của Daihatsu. Không được kết luận chức năng chỉ từ số đầu; phải đối chiếu `Name` và `Use`.

### Tag Start, Stop và điều khiển

| Tag | Ý nghĩa |
|---|---|
| `88V` | Starting solenoid valve |
| `88L` | Fuel-oil control solenoid valve trong trình tự khởi động |
| `5V` | Shutdown solenoid valve — Control |
| `5S` | Shutdown solenoid valve — Safety & Control |
| `SVB` | Solenoid valve của bypass bộ làm mát khí LT |
| `BO` | Blow-off valve |
| `GM` | Governor motor |
| `FR` | Fuel-rack position transmitter/indicator |

`5V` và `5S` cắt nhiên liệu khi cuộn dây được cấp điện. Việc có hai đường thể hiện kênh Control và kênh Safety & Control riêng.

### Tag tốc độ và interlock

| Tag | Ý nghĩa/nguyên tắc |
|---|---|
| `SSE1`, `SSE2` | Hai speed sensor dạng pulse, dùng để phát hiện tốc độ và đáp ứng điều kiện Ready to Start |
| `HS` | Handle Switch; ở RUN thì `2–4` ON và `1–2` OFF |
| `TC` | Turning Safety Switch; khi turning bar đang dùng thì `1–3` OFF, ngăn khởi động |
| `GM` | Governor Motor điều khiển tốc độ, nguồn DC24V có battery backup |

### Cách đọc trạng thái tiếp điểm

Sơ đồ ghi rõ các tiếp điểm được vẽ ở trạng thái động cơ chạy bình thường. Không được chỉ nhìn hình tiếp điểm; phải đọc thêm cột `Contact` và điều kiện trong `Remark`.

| Ghi chú | Ý nghĩa |
|---|---|
| `Normal open` | Bình thường hở |
| `Normal close` | Bình thường đóng |
| `(1–2) ON` | Chân 1 và 2 thông nhau khi điều kiện tác động |
| `(1–3) OFF` | Chân 1 và 3 mở ra khi điều kiện tác động |
| `(A–B) OFF` | Chân A và B mở ra khi điều kiện tác động |

Ví dụ:

- `63Q2`: normally open; áp suất dưới 0,30 MPa làm `1–2` ON và tạo Emergency Stop.
- `63QDM`: normally closed; chênh áp trên 0,09 MPa làm `1–3` OFF và tạo Alarm.
- `33F`: normally closed; phát hiện rò rỉ làm `1–2` OFF và tạo Alarm.

Một số mạch tác động bằng cách mở tiếp điểm đang đóng để hỗ trợ logic fail-safe và giám sát đứt dây.

### Nguyên tắc tách kênh analog và Trip

Không được đồng nhất sensor/transmitter analog với safety switch:

```text
PTQ
├── Đo analog và hiển thị: 30025
├── Alarm áp suất dưới 0,35 MPa
└── Sensor failure: 10089

63Q2
├── Pressure switch vật lý độc lập
├── Emergency Stop dưới 0,30 MPa: 10021
└── Giám sát đứt dây switch: 10062
```

Tương tự:

```text
TRWHO
├── Đo analog và Alarm trên 90°C
└── Sensor failure riêng

26WH2
└── Temperature switch độc lập, Emergency Stop trên 95°C
```

Vì vậy `Sensor Failure` không mặc nhiên bằng `Trip`. Phải xác định thiết bị nào báo Failure và switch vật lý nào thực sự tạo Trip.

### Trình tự giải thích một tag mới

1. Tìm tag trong cột `Mark`.
2. Đọc tên đầy đủ tại cột `Name`.
3. Xác định `Use`: Indication, Alarm, Emergency Stop, Control hay Safety.
4. Đọc ngưỡng hoặc điều kiện trong `Remark`.
5. Đọc trạng thái chân trong `Contact`.
6. Xác định thiết bị là transmitter analog, temperature sensor hay switch số.
7. Đối chiếu chân đấu tại junction box.
8. Sau cùng mới liên hệ tag với địa chỉ Modbus.

Khi đánh giá Failure/Trip cần trả lời đủ:

- Sensor hoặc transmitter nào đang báo Failure?
- Switch nào thực sự tạo Trip?
- Hai kênh có độc lập về phần cứng không?
- Có dùng chung nguồn hoặc dây không?
- Logic DECCS có xử lý fail-safe bổ sung không?

### Kết luận về hệ thống tag

```text
Quy ước ISA/điện phổ biến
        +
Mã môi chất và vị trí riêng của Daihatsu
        +
Số kênh hoặc số thiết bị
```

Ví dụ:

- `PTQ`: Pressure Transmitter — dầu bôi trơn.
- `TRWHO`: Temperature sensor — Water — High-temperature — Outlet.
- `63Q2`: Pressure switch — dầu bôi trơn — kênh/thiết bị số 2.
- `26WH2`: Thermal switch — Water High-temperature — kênh/thiết bị số 2.
- `88V`: Starting solenoid valve.
- `5S`: Shutdown solenoid thuộc kênh Safety & Control.

Ý nghĩa chính xác cuối cùng phải lấy từ bảng `Mark/Name/Use/Remark/Contact` của Daihatsu, không chỉ từ cách giải mã theo ISA.

## Bổ sung: van 88L, Air Run và Timing Chart

Ngày ghi chú: 2026-10-27.

Nguồn đối chiếu:

- `Start Engine.pdf`: mục 2.3.2–2.3.3, Fig.38–39, trang in 45–46; mục 2.3.5 và 2.6 về Start Failure và Emergency Stop.
- `FO & Exh piping diagram.pdf`: sơ đồ AQA10036415, đường nhiên liệu và khí xả.
- `SA piping diagram.pdf`: sơ đồ AQA10036416, đường khí khởi động và khí điều khiển.
- `Timing chart.pdf`: mục 5.1, Fig.46–48, trang in 59–60, tài liệu INB000095D.

### 88L nằm ở đâu và điều khiển môi chất gì?

Trên sơ đồ SA, cụm bốn van nằm phía trên, gần giữa bản vẽ. Từ trái sang phải:

| Tag | Tên ghi trên bản vẽ | Cổng ra |
|---|---|---|
| `88L` | F.O. control magnetic valve | A1 |
| `5V` | F.O. shut down magnetic valve | A2 |
| `88V` | Starting magnetic valve | A3 |
| `5S` | F.O. shut down magnetic valve | A4 |

Đường A1 của 88L nối đến khối `FO control piston & fuel shutdown device`. Đường A2 của 5V nối đến một cổng khác của cùng khối. Bản vẽ không thể hiện chi tiết cơ khí bên trong khối này.

**88L là van điện từ khí nén điều khiển piston FO; dầu FO không đi qua van 88L.** Tên van nói đến chức năng/đối tượng được điều khiển, không phải môi chất đi qua van. Vì vậy, van xuất hiện trên bản vẽ SA, vốn thể hiện cả Starting Air và Control Air.

Theo ký hiệu van 3 cửa, 2 vị trí, hồi bằng lò xo trên sơ đồ SA:

| Trạng thái | Đường khí | Ý nghĩa |
|---|---|---|
| 88L ON | Cửa 1 nối cửa 2; cửa R bị chặn | Cấp khí qua A1 đến piston FO |
| 88L OFF | Cửa 1 bị chặn; cửa 2 nối R | Ngừng cấp khí và xả khí từ nhánh piston FO |

Chuỗi tác động: điện điều khiển 88L → van chuyển đường khí → khí tác động piston FO → piston tác động cơ cấu điều khiển nhiên liệu.

Trên sơ đồ FO & Exh, đường dầu đi từ `From FO line` qua `F.O. filter` đến các `FO injection pump`, rồi đến vòi phun. Có đường `Return FO` và `Relief valve`. Không thấy 88L được vẽ như một van đóng/mở nối tiếp trên đường dầu cấp này.

### Piston FO khác piston động cơ và chưa xác nhận liên kết governor

- Nhánh `88V` điều khiển hệ thống khí khởi động; khí khởi động vào xy-lanh để đẩy piston động cơ và quay trục khuỷu.
- Nhánh `88L` cấp khí cho piston của cơ cấu điều khiển nhiên liệu; piston này không trực tiếp tạo mô-men quay trục khuỷu.
- Xả khí khi 88L OFF là giải phóng áp lực tác động ở nhánh piston FO, không phải cấp lực đẩy tạo đà quay.
- Governor điều chỉnh lượng phun để điều khiển tốc độ. Piston FO có thể liên kết với hệ thanh đòn/thanh răng mà governor cũng tác động, nhưng sơ đồ hiện có chưa xác nhận liên kết đó.
- Chưa đủ căn cứ để nói piston FO nằm trong governor, gắn trực tiếp với governor, đẩy thanh răng theo chiều nào, hoặc hồi về bằng cơ cấu nào.

Cần bản vẽ cơ khí/hình cắt `FO control piston & fuel shutdown device`, `fuel control mechanism` hoặc `governor linkage` để xác định các chi tiết trên.

### Ý nghĩa ON/OFF của 88L và chức năng khống chế nhiên liệu

Timing chart ghi nhãn Nhật `F.O. 抑制電磁弁(88L)`, trong đó `抑制` mang nghĩa ức chế/hạn chế/khống chế. Nhãn này củng cố cách hiểu 88L phục vụ chức năng khống chế nhiên liệu trong giai đoạn khởi động.

**Đã xác nhận:** 88L cấp/xả khí đến piston FO và chỉ được kích hoạt trong một giai đoạn của trình tự. **Suy luận phù hợp nhưng chưa xác nhận cơ khí:** piston có thể hạn chế lượng nhiên liệu khởi động. Chưa được khẳng định nó chặn thanh răng ở vị trí nào hoặc giới hạn lượng phun bao nhiêu.

Không đồng nhất ba trạng thái: solenoid ON/OFF, có dầu cấp đến bơm cao áp, và có phun dầu vào xy-lanh. 88L OFF không có nghĩa cắt nguồn dầu FO. 88L ON cũng không bảo đảm có phun dầu, như trường hợp Air Run.

Không diễn giải `88L OFF → governor điều khiển` thành governor chỉ bắt đầu làm việc khi 88L OFF. Tài liệu xác nhận tác động khí qua 88L kết thúc và máy tiếp tục chạy dưới sự điều chỉnh của governor, không xác định thời điểm governor bắt đầu tác động.

### Khởi động LOCAL so với Air Run

| Nội dung | Khởi động LOCAL | Air Run |
|---|---|---|
| ENGINE MODE | LOCAL | LOCAL |
| Cần vận hành | RUN | STOP theo mục 2.3.3; Fig.48 cũng dùng từ OFF trong chú thích |
| Mục đích | Máy nổ và chạy bằng nhiên liệu | Quay máy bằng khí, thổi sạch khí chưa cháy |
| Nhiên liệu phun | Có | Không |
| START | Nhấn giữ để khởi động theo hướng dẫn | Nhấn giữ 3–4 giây rồi thả theo mục 2.3.3 |
| 88V | OFF khi đạt ngưỡng khởi động | OFF ngay khi thả START |
| 88L | OFF sau khoảng duy trì quy định | Theo mục 2.3.3/Fig.39, giữ đến khi máy dừng hoàn toàn rồi OFF |

Air Run vẫn có tốc độ quay do khí nén. Câu “rotational speed ... does not increase” trong hướng dẫn được hiểu theo ngữ cảnh là máy không nổ và tăng tốc lên trạng thái chạy bằng nhiên liệu, không phải trục khuỷu đứng yên.

Cả hai chế độ đều có thể kích hoạt piston FO nhưng kết quả phun nhiên liệu khác nhau do trạng thái RUN/STOP và cơ cấu điều khiển liên quan. Không suy ra cơ cấu bên trong chỉ từ biểu đồ ON/OFF.

### Fig.46: khởi động và dừng bình thường

Các giá trị dưới đây là giá trị ghi trên timing chart; cần đối chiếu cài đặt máy thực tế khi sử dụng.

| Mốc | Hoạt động |
|---|---|
| Nhận START | 88V và 88L ON |
| Đạt `S#14 = 300 rpm` | 88V OFF; nhận trạng thái ENGINE RUN; bắt đầu các khoảng ST2/ST4 |
| Sau #14 thêm `ST2 = 15 giây` | 88L OFF |
| Sau #14 thêm `ST4 = 30 giây` | Hết repose nhóm #14T |
| Có STOP bình thường | 5V ON; không thể hiện 5S được kích hoạt |
| Giảm tốc và xác nhận dừng | Biểu đồ dùng ngưỡng `Send = 30 rpm` cùng `ST3 = 20 giây` |
| Hoàn thành trình tự dừng | 5V OFF; trở lại READY nếu đủ các interlock |

Ví dụ: nếu đạt #14 tại giây thứ 4 kể từ START, 88L OFF khoảng giây thứ 19 và #14T hết repose khoảng giây thứ 34. Mốc 4 giây chỉ là ví dụ, không phải giá trị đặt.

Mục 2.3.2 còn nói ngưỡng ngắt 88V có thể là 150 rpm tùy cài đặt; không áp dụng 300 rpm như giá trị bất biến cho mọi cấu hình.

### Fig.47: dừng khẩn cấp

- Khi có Emergency Stop: cả 5V và 5S ON.
- 5V kết thúc tác động theo trình tự xác nhận dừng có ST3.
- 5S giữ ON đến khi thực hiện HANDLE SW RESET; không tự kết thúc chỉ vì tốc độ đã giảm về không.
- Sau xác nhận dừng vẫn chưa READY nếu chưa reset. Hướng dẫn yêu cầu động cơ dừng hoàn toàn và đưa handle về OFF.
- `Send = 30 rpm` là ngưỡng dùng trong logic, không phải cho phép coi máy còn quay là đã đứng yên. Không bỏ qua yêu cầu dừng hoàn toàn của hướng dẫn vận hành.

### Fig.48: Start Failure và Air Run

Start Failure áp dụng cho REMOTE, handle RUN. Timer thất bại là 20 giây tính từ START. Nếu không đạt ngưỡng 300 rpm trong thời gian này, hệ thống phát hiện Start Failure, cho 88V/88L OFF và 5V/5S ON để dừng. Sau đó cần reset handle trước khi được phép khởi động lại.

Air Run áp dụng cho LOCAL, handle STOP/OFF. Thả START thì 88V OFF; hệ thống trở lại READY sau trình tự xác nhận dừng.

**Điểm thiếu cần giữ lại khi đối chiếu:** phần Air Run của Fig.48 không vẽ rõ khoảng ON của 88L, trong khi mục 2.3.3 và Fig.39 thể hiện 88L hoạt động và chỉ OFF sau khi máy dừng. Không dùng phần bỏ trống ở Fig.48 để kết luận Air Run không kích hoạt 88L; cũng không áp dụng ST2 của khởi động thành công sang Air Run khi chưa có xác nhận.

### Repose và điểm khác nhau giữa các tài liệu

Theo Fig.46:

- Emergency Stop, Overspeed và Oil Mist High được giám sát liên tục.
- Trip áp suất dầu bôi trơn thấp bắt đầu được giám sát sau `#14 + ST4`.
- Trip nhiệt độ nước HT cao được vẽ bắt đầu từ `#14`, không chờ ST4.
- Repose Starting kéo dài trong khi van khởi động hoạt động và thêm 5 giây.
- Nhóm khí xả có điều kiện nhiệt độ khí xả trung bình trên 200°C.
- Nhóm chênh áp lọc dầu có điều kiện nhiệt độ dầu trên 45°C.

**Lưu ý bổ sung cho bảng Automatic Trip ở phần trước:** bảng trong `Start Engine.pdf`, mục 2.6, ghi cả áp suất dầu thấp và nhiệt độ nước HT cao được giám sát từ #14T; nhưng Fig.46 vẽ riêng Trip HT từ #14. Đây là khác biệt giữa hai phần tài liệu, chưa được giải quyết. Không chọn một mốc làm logic PLC chính thức khi chưa đối chiếu phiên bản tài liệu và cấu hình DECSS của máy.

Timing chart xác nhận thời điểm điều khiển và bảo vệ, không thay thế bản vẽ cơ khí để giải thích piston FO tác động cụ thể lên governor/thanh răng như thế nào.

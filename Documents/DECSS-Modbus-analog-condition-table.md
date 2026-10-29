# Bảng điều kiện trạng thái các điểm analog DECSS Modbus

Nguồn tổng hợp: `Modbus List.pdf`, `Start Engine.pdf`, `Timing chart.pdf`, `Wiring Diagram.pdf` và tài liệu `DECSS-Modbus-monitoring-protection-and-interlock-notes.md`.

### Bảng tính trạng thái cho các điểm analog đang sử dụng

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



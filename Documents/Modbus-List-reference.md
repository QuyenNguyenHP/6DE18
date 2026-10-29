# Modbus List — Tài liệu tra cứu

Nguồn: `Modbus List.pdf` — Daihatsu Diesel MFG. Co., Ltd.

## Thông tin tài liệu

| Thuộc tính | Giá trị |
|---|---|
| Ship No. | DB66-01/02 B |
| List No. | AQA10036557B |
| Ngày | 2022-04-08 |
| Revision | B — 2022-11-03 — S.I |
| Số trang | 9 |

PDF trình bày song ngữ Anh–Nhật. File này giữ nội dung kỹ thuật tiếng Anh và bổ sung ghi chú tiếng Việt để dễ tìm kiếm.

## Tổng hợp

| Loại | Dải địa chỉ | Số vị trí | Có dấu `○` tại cột Use |
|---|---:|---:|---:|
| Digital ON/OFF | 10001–10136 | 136 | **48** |
| Analog PT100/4–20 mA | 30001–30056 | 56 | **19** |
| Tổng đang dùng | | | **67** |

`Use = ○` nghĩa là điểm được đánh dấu sử dụng trong PDF. `—` nghĩa là ô trống hoặc không áp dụng.

## Digital ON/OFF — 10001–10096

| No. | Address | Name | OFF (0) | ON (1) | Repose | Use |
|---:|---:|---|---|---|---:|:---:|
| 1 | 10001 | Lub. oil filter diff. press. — High | Normal | Alarm | 6 | ○ |
| 2 | 10002 | Control air pressure — Low | Normal | Alarm | — | — |
| 3 | 10003 | Fuel oil pressure — Low | Normal | Alarm | 2 | — |
| 4 | 10004 | Lub. oil inlet pressure — Low | Normal | Alarm | 2 | — |
| 5 | 10005 | High temp. cooling water pressure — Low | Normal | Alarm | 2 | — |
| 6 | 10006 | Low temp. cooling water pressure — Low | Normal | Alarm | 2 | — |
| 7 | 10007 | Fuel oil drain tank level — High | Normal | Alarm | — | — |
| 8 | 10008 | High temp. cooling water outlet temp. — High | Normal | Alarm | 1 | — |
| 9 | 10009 | Lub. oil for turbocharger pressure — Low | Normal | Alarm | 2 | ○ |
| 10 | 10010 | Fuel oil filter diff. press. — High | Normal | Alarm | — | — |
| 11 | 10011 | Starting air pressure — Low | Normal | Alarm | — | — |
| 12 | 10012 | Fuel oil leak tank level — High | Normal | Alarm | — | ○ |
| 13 | 10013 | Lub. oil sump tank level — Low | Normal | Alarm | — | — |
| 14 | 10014 | Lub. oil sump tank level — High | Normal | Alarm | — | — |
| 15 | 10015 | Oil mist detector pre-alarm | Normal | Alarm | — | — |
| 16 | 10016 | Oil mist detector trouble | Normal | Alarm | — | — |
| 17 | 10017 | Engine run | Stop | Running | — | ○ |
| 18 | 10018 | Ready to start | Not ready | Ready | — | ○ |
| 19 | 10019 | Over speed (stop) | Normal | Trip | — | ○ |
| 20 | 10020 | High temp. cooling water outlet temp. (stop) | Normal | Trip | 1 ※1 | ○ |
| 21 | 10021 | Lub. oil inlet pressure (stop) | Normal | Trip | 2 ※1 | ○ |
| 22 | 10022 | Oil mist detector high alarm (stop) | Normal | Trip | — | — |
| 23 | 10023 | Emergency stop (Remote / Local) | Normal | Trip | — | ○ |
| 24 | 10024 | Start failure | Normal | Trip | — | ○ |
| 25 | 10025 | Priming pump thermal failure | Normal | Alarm | — | ○ |
| 26 | 10026 | Priming lub. oil pressure (Low) | Normal | Alarm | 4 | ○ |
| 27 | 10027 | Priming pump run | Stop | Running | — | ○ |
| 28–32 | 10028–10032 | Reserved / blank | — | — | — | — |
| 33 | 10033 | System failure | Normal | Alarm | — | ○ |
| 34 | 10034 | Control module failure | Normal | Alarm | — | ○ |
| 35 | 10035 | Safety module failure | Normal | Alarm | — | ○ |
| 36 | 10036 | Link to engine condition display failure | Normal | Alarm | — | ○ |
| 37 | 10037 | Link to remote I/O-1 failure | Normal | Alarm | — | ○ |
| 38 | 10038 | Link to remote I/O-2 failure | Normal | Alarm | — | ○ |
| 39 | 10039 | Link to LCD gauge board failure | Normal | Alarm | — | — |
| 40 | 10040 | Reserved / blank | — | — | — | — |
| 41 | 10041 | No.1 alarm repose signal (#14) | OFF | Repose | — | ○ |
| 42 | 10042 | No.2 alarm repose signal (#14T) | OFF | Repose | — | ○ |
| 43 | 10043 | No.3 alarm repose signal (Exh. gas) | OFF | Repose | — | ○ |
| 44 | 10044 | No.4 alarm repose signal (Priming) | OFF | Repose | — | ○ |
| 45 | 10045 | No.5 alarm repose signal (Starting) | OFF | Repose | — | ○ |
| 46 | 10046 | No.6 alarm repose signal (Lub. oil filter diff. press.) | OFF | Repose | — | ○ |
| 47–48 | 10047–10048 | Reserved / blank | — | — | — | — |
| 49 | 10049 | No.1 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 50 | 10050 | No.2 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 51 | 10051 | No.3 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 52 | 10052 | No.4 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 53 | 10053 | No.5 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 54 | 10054 | No.6 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 55 | 10055 | No.7 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 56 | 10056 | No.8 cylinder exhaust gas temp. deviation alarm | Normal | Alarm | — | — |
| 57 | 10057 | Exhaust gas temp. deviation alarm — Common | Normal | Alarm | — | — |
| 58 | 10058 | Reserved / blank | — | — | — | — |
| 59 | 10059 | Emergency stop switch of external — Disconnection | Normal | Failure | — | ○ |
| 60 | 10060 | Emergency stop switch on ECD — Disconnection | Normal | Failure | — | ○ |
| 61 | 10061 | HT cooling water temp. switch — Disconnection | Normal | Failure | — | ○ |
| 62 | 10062 | Lub. oil pressure switch — Disconnection | Normal | Failure | — | ○ |
| 63 | 10063 | Oil mist detector high-alarm — Disconnection | Normal | Failure | — | — |
| 64 | 10064 | Emergency stop solenoid valve — Disconnection | Normal | Failure | — | ○ |
| 65 | 10065 | Fuel oil temp. engine inlet sensor failure | Normal | Failure | — | — |
| 66 | 10066 | Boost air temp. sensor failure | Normal | Failure | — | — |
| 67 | 10067 | Lub. oil engine inlet temp. sensor failure | Normal | Failure | — | ○ |
| 68 | 10068 | HT cooling water engine outlet temp. sensor failure | Normal | Failure | — | ○ |
| 69 | 10069 | HT cooling water engine inlet temp. sensor failure | Normal | Failure | — | — |
| 70 | 10070 | No.1 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 71 | 10071 | No.2 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 72 | 10072 | No.3 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 73 | 10073 | No.4 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 74 | 10074 | No.5 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 75 | 10075 | No.6 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | ○ |
| 76 | 10076 | No.7 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | — |
| 77 | 10077 | No.8 cylinder exhaust gas temp. sensor failure | Normal | Failure | — | — |
| 78 | 10078 | Exhaust gas T/C inlet 1 temp. sensor failure | Normal | Failure | — | ○ |
| 79 | 10079 | Exhaust gas T/C inlet 2 temp. sensor failure | Normal | Failure | — | ○ |
| 80 | 10080 | Exhaust gas T/C outlet temp. sensor failure | Normal | Failure | — | ○ |
| 81 | 10081 | HT cooling water pressure inlet sensor failure | Normal | Failure | — | ○ |
| 82 | 10082 | Boost air pressure sensor failure | Normal | Failure | — | — |
| 83 | 10083 | LT cooling water pressure inlet sensor failure | Normal | Failure | — | ○ |
| 84 | 10084 | Starting air pressure sensor failure | Normal | Failure | — | ○ |
| 85 | 10085 | Fuel oil engine inlet pressure sensor failure | Normal | Failure | — | ○ |
| 86 | 10086 | Control air pressure sensor failure | Normal | Failure | — | — |
| 87 | 10087 | Lub. oil T/C inlet pressure sensor failure | Normal | Failure | — | — |
| 88 | 10088 | Lub. oil filter diff. press sensor failure | Normal | Failure | — | — |
| 89 | 10089 | Lub. oil engine inlet pressure sensor failure | Normal | Failure | — | ○ |
| 90 | 10090 | All speed pickup sensor failure | Normal | Failure | — | ○ |
| 91 | 10091 | Load input failure | Normal | Failure | — | ○ |
| 92–96 | 10092–10096 | Reserved / blank | — | — | — | — |

`※1`: Reposed by DECCS.

## Digital Option I/O Module 2 — 10097–10136

Không điểm nào trong phần này có dấu `Use`.

| Address | Name / chức năng |
|---:|---|
| 10097–10112 | OPTION: REMOTE I/O 2 MODULE, D/I-1 đến D/I-16 |
| 10113 | Lub. oil engine outlet temp. sensor failure — Normal/Failure |
| 10114 | LT cooling water engine inlet sensor failure — Normal/Failure |
| 10115 | LT cooling water engine outlet sensor failure — Normal/Failure |
| 10116–10128 | OPTION: REMOTE I/O 2 MODULE, A/I-4 đến A/I-16 SENSOR FAIL |
| 10129 | T/C speed sensor failure — Normal/Failure |
| 10130–10136 | OPTION: REMOTE I/O 2 MODULE, A/I-18 đến A/I-24 SENSOR FAIL |

## Analog PT100/4–20 mA — 30001–30056

`Reg` là hệ số scale/độ phân giải ghi trong PDF.

| No. | Address | Name | Reg | Unit | Min | Max | Repose | Use |
|---:|---:|---|---:|---|---:|---:|---:|:---:|
| 1 | 30001 | Fuel oil temp. engine inlet | 1 | °C | 0 | 200 | — | — |
| 2 | 30002 | Boost air temp. | 1 | °C | 0 | 200 | — | — |
| 3 | 30003 | Lub. oil temp. engine inlet | 1 | °C | 0 | 200 | — | ○ |
| 4 | 30004 | HT cooling water temp. engine outlet | 1 | °C | 0 | 200 | 1 | ○ |
| 5 | 30005 | HT cooling water temp. engine inlet | 1 | °C | 0 | 200 | — | — |
| 6 | 30006 | No.1 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 7 | 30007 | No.2 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 8 | 30008 | No.3 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 9 | 30009 | No.4 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 10 | 30010 | No.5 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 11 | 30011 | No.6 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | ○ |
| 12 | 30012 | No.7 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | — |
| 13 | 30013 | No.8 cylinder exhaust gas temp. | 1 | °C | 0 | 700 | 3 | — |
| 14 | 30014 | Exhaust gas temp. T/C inlet 1 | 1 | °C | 0 | 700 | — | ○ |
| 15 | 30015 | Exhaust gas temp. T/C inlet 2 | 1 | °C | 0 | 700 | — | ○ |
| 16 | 30016 | Exhaust gas temp. T/C outlet | 1 | °C | 0 | 700 | — | ○ |
| 17 | 30017 | HT cooling water pressure engine inlet | 0.01 | MPa | 0 | 0.6 | 2 | ○ |
| 18 | 30018 | Boost air pressure | 0.001 | MPa | 0 | 0.6 | — | — |
| 19 | 30019 | LT cooling water pressure engine inlet | 0.01 | MPa | 0 | 0.6 | 2 | ○ |
| 20 | 30020 | Starting air pressure | 0.01 | MPa | 0 | 4 | 5 | ○ |
| 21 | 30021 | Fuel oil pressure engine inlet | 0.01 | MPa | 0 | 1.6 or 0.6 | 2 | ○ |
| 22 | 30022 | Control air pressure | 0.01 | MPa | 0 | 1.6 | — | — |
| 23 | 30023 | Lub. oil pressure T/C inlet | 0.01 | MPa | 0 | 1 | 2 | — |
| 24 | 30024 | Lub. oil filter differential | 0.01 | MPa | 0 | 0.2 | 6 | — |
| 25 | 30025 | Lub. oil pressure | 0.01 | MPa | 0 | 1 | 2 | ○ |
| 26 | 30026 | Engine speed | 1 | min⁻¹ | 0 | 1500 | — | ○ |
| 27 | 30027 | Reserved / blank | — | — | — | — | — | — |
| 28 | 30028 | Load | 1 | kW | 0 | `*` | — | ○ |
| 29 | 30029 | Running hour | 1 | 10 × hour | 00 | 65535 | — | ○ |
| 30–32 | 30030–30032 | Reserved / blank | — | — | — | — | — | — |
| 33 | 30033 | Lub. oil temp. engine outlet | 1 | °C | 0 | 200 | — | — |
| 34 | 30034 | LT-CW temp. engine inlet | 1 | °C | 0 | 200 | — | — |
| 35 | 30035 | LT-CW temp. engine outlet | 1 | °C | 0 | 200 | — | — |
| 36–48 | 30036–30048 | OPTION: REMOTE I/O 2 MODULE, PT100-4 đến PT100-16 | — | — | — | — | — | — |
| 49 | 30049 | T/C speed | 1 | 10 × min⁻¹ | 0 | 7000 | — | — |
| 50–56 | 30050–30056 | OPTION: REMOTE I/O 2 MODULE, 4–20 mA-18 đến 4–20 mA-24 | — | — | — | — | — | — |

Dấu `*` tại Max của `30028 Load` được giữ nguyên như PDF; PDF không có chú giải thêm.

## Chi tiết điều kiện Repose

### Repose 1 — #14 (`10041`)

Tốc độ động cơ bằng hoặc dưới 300 min⁻¹, hoặc động cơ không hoạt động. Van điện từ dừng hoặc dừng khẩn cấp đang được kích từ.

### Repose 2 — #14T (`10042`)

Tốc độ bằng hoặc dưới 300 min⁻¹ và trạng thái low-speed-or-higher chưa vượt timer ST4; hoặc động cơ không hoạt động.

### Repose 3 — Exhaust (`10043`)

Điều kiện #14 thỏa mãn, hoặc nhiệt độ khí xả trung bình thấp hơn giá trị Repose cài đặt; PDF nêu ví dụ 200°C.

### Repose 4 — Priming (`10044`)

Kích hoạt nếu có một trong các điều kiện:

- Động cơ đang chạy, tốc độ cao hơn low speed (#14).
- Bơm priming không chạy.
- Chưa quá 20 giây từ lúc bắt đầu vận hành.
- Bảo vệ nhiệt bơm priming đang tác động.

### Repose 5 — Starting (`10045`)

Kích hoạt khi van điện từ khởi động đang được kích từ và kéo dài 5 giây sau khi ngừng kích từ.

### Repose 6 — Filter differential pressure (`10046`)

Kích hoạt khi nhiệt độ dầu bôi trơn đầu vào bằng hoặc thấp hơn 45°C.

## Danh sách điểm đang dùng

### Digital — 48 điểm

`10001`, `10009`, `10012`, `10017`, `10018`, `10019`, `10020`, `10021`, `10023`, `10024`, `10025`, `10026`, `10027`, `10033`, `10034`, `10035`, `10036`, `10037`, `10038`, `10041`, `10042`, `10043`, `10044`, `10045`, `10046`, `10059`, `10060`, `10061`, `10062`, `10064`, `10067`, `10068`, `10070`, `10071`, `10072`, `10073`, `10074`, `10075`, `10078`, `10079`, `10080`, `10081`, `10083`, `10084`, `10085`, `10089`, `10090`, `10091`.

### Analog — 19 điểm

`30003`, `30004`, `30006`, `30007`, `30008`, `30009`, `30010`, `30011`, `30014`, `30015`, `30016`, `30017`, `30019`, `30020`, `30021`, `30025`, `30026`, `30028`, `30029`.

## Cách tra cứu

- Tìm địa chỉ cụ thể: tìm `10089` hoặc `30025`.
- Tìm Trip: tìm `| Trip |`.
- Tìm Failure: tìm `| Failure |`.
- Tìm điểm đang dùng: tìm ký hiệu `○`.
- Tìm áp suất: tìm `pressure`; nhiệt độ: tìm `temp.`; turbocharger: tìm `T/C`.

## Lưu ý kỹ thuật

- `Failure` báo lỗi cảm biến, dây, module hoặc đường truyền; không mặc nhiên đồng nghĩa với `Trip`.
- Giá trị analog và bit lỗi cảm biến là hai dữ liệu khác nhau. Ví dụ `30025` là giá trị áp suất dầu bôi trơn; `10089` là bit lỗi cảm biến liên quan.
- Các địa chỉ Option không có dấu `Use` trong tài liệu này.
- Trước khi cấu hình PLC/HMI cần xác nhận thêm function code, kiểu thanh ghi, byte order và cách đọc trong tài liệu giao thức DECCS.

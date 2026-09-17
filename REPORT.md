# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 2A202602299
- Ngày / CVAT local: 2026-09-17 / CVAT v2.74.1 tại `http://localhost:8080`
- Công cụ đã dùng: CVAT (Polygon vẽ tay, mask); gợi ý tự động từ model `seg-ai20k-semantic19` (semantic, chạy trong CVAT: Easy, cp3/cp4/cp6 và stuff của Hard) và `yolo11x-seg` của Ultralytics chạy trên máy (mask instance: Medium, thing của Hard, cp1/cp2/cp5). Toàn bộ thao tác CVAT (tạo task, chạy gợi ý, sửa, Save, export) được thực hiện qua Claude Code (AI agent) điều khiển CVAT API/trình duyệt; học viên duyệt kết quả sau từng bước. Không dùng SAM.

Ghi chú về cách tạo mask instance (hai phiên bản, ghi trung thực):

- **Bản 1 (trước khi có ground truth):** YOLO11 trên CVAT chỉ trả box nên không dùng box làm mask; mask từng vật = pixel người/xe của model semantic cắt theo box YOLO11. Tự đánh giá sau khi nhận ground truth: **20.6 / 82** (Easy 18.0, Medium 0.0, Hard 2.6). Nguyên nhân chính: model semantic (dashcam) bỏ sót nhiều người/xe máy trên ảnh COCO → recall Medium 0.49. Bản ZIP này được giữ lại kèm SHA-256 ngoài repo.
- **Bản 2 (sau khi xem điểm, bản đang nộp):** thay mask instance tự động bằng `yolo11x-seg` (conf 0.25, imgsz 1280; bỏ mask cùng class chồng >70% lên mask tin cậy cao hơn), giữ nguyên các object vẽ tay và stuff của Hard. Ground truth chỉ dùng để chạy scorer, **không dùng để vẽ hay chọn mask**. Bản này sửa sau khi đã xem điểm nên không phải bằng chứng làm độc lập.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `submissions/cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `submissions/cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `submissions/cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `submissions/cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `submissions/cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `submissions/cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Không có export lỗi. `python3 scripts/inspect_submissions.py --dir submissions` báo OK cho cả 9 ZIP (chỉ là kiểm cấu trúc, không phải điểm). Hạn chế đã biết: Medium nộp 89 mask trong khi scorer báo reference có 71 → còn mask thừa (người/xe rất nhỏ); Hard còn yếu ở `person`, `motorcycle`, `sidewalk`.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000373353.jpg`, xe buýt hai tầng màu đỏ giữa phố (khoảng x 272–366, y 215–345). Vẽ bằng Polygon 21 điểm **trước khi** chạy bất kỳ gợi ý nào trên task này. Object này do agent vẽ tay theo quy tắc (không lấy từ model), học viên duyệt lại.
- Class và quy tắc tôi dùng để chọn biên: `bus`. Chỉ vẽ phần nhìn thấy: dừng ở mép mui taxi vàng phía trước và ở đầu/vai người đi bộ áo xám che phía dưới, không đoán thân xe sau hai vật đó. Kính chắn gió và bảng tên tuyến giữ trong mask (không khoét lỗ).
- Nếu dùng gợi ý sau đó: ở cả hai bản gợi ý đều có một mask `bus` trùng xe này (IoU > 0.5) → **bỏ đề xuất, giữ polygon tay** vì polygon tay đã dừng đúng ở vật che. Ảnh `000000181542.jpg`: gợi ý bản 1 bỏ sót chiếc taxi trắng bên trái (có box nhưng không có pixel semantic) → vẽ tay polygon `car` phần nhìn thấy (x 0–121, y 121–192), dừng ở người đội mũ và người lái xe máy che; ở bản 2 `yolo11x-seg` có mask trùng taxi này → vẫn giữ polygon tay, bỏ mask máy.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp1_holes`, ảnh `000000144300.jpg` — chiếc mô tô Honda đỏ-trắng chiếm giữa ảnh.
- Lỗi thuộc loại: gộp-tách (một vật bị tách thành hai object).
- Bằng chứng tôi nhìn thấy: overlay mask của `yolo11x-seg` có **2 object** `motorcycle`: một mask phủ nửa trước (x 47–385, conf 0.81) và một mask phủ nửa sau (x 225–586, conf 0.55); hai mask chồng một phần nên bước lọc trùng không loại được.
- Quy tắc và hành động sửa: một vật vật lý = một instance; kính chắn gió và khe giữa khung giữ trong mask (quy tắc `cp1_holes`) → gộp hai mask thành **một** mask `motorcycle`.
- Sau sửa đã Save và export lại chưa? Đã Save trong CVAT và export lại `cp1_holes.zip` (COCO 1.0, 7 annotation).

Lỗi thật khác đã sửa và **còn trong bản nộp**: `easy_semantic` 7ee6d192 — polygon `sky` bị khoét quanh cột điện cao thế (x 383–410, y 254–297), thêm polygon `sky` (phủ vùng); `cp6_coverage` — 3 taxi vàng nhỏ dưới mái hiên (x 892–1137, y 320–365) bị gộp vào `building`, vẽ thêm 3 polygon `car` (sai lớp).

Ở bản 1 còn sửa `cp5_occlusion` (gộp 2 mảnh xe máy bị người lái che, tách 2 taxi vàng bị gộp), `cp1_holes` (xoá mask gán cho giá đỡ đỏ) và `hard_panoptic` 460147 (xoá `truck` gán cho cửa cuốn gỉ). Các mask đó đã được thay ở bản 2. Kiểm lại overlay bản 2: xe máy ở `cp5_occlusion` đã là một mask, nhưng **hai taxi vàng vẫn bị gộp thành một mask `car`** (x 373–639, y 203–423) → tách lại theo mép capo xe trước (xe sau x 408–639, y 203–278), Save và export lại `cp5_occlusion.zip` (39 annotation).

Soát lại bằng mắt sau bản 2 (dựa trên ảnh, không mở mask đáp án) và sửa thêm: `medium_instance` 373353 — SUV tối màu bên trái bị gán `truck` → đổi `car`; 458325 — mask xe xám bên trái lấn một mảng chữ nhật lên cản sau SUV đỏ phía trước → cắt phần lấn; mask xe ngoài cùng bên trái có mảnh rời nằm trong xe xám → xoá mảnh rời; `hard_panoptic` 460147 — mask `truck` phủ biển hiệu ENEOS/toà nhà/giá lốp → xoá; người đứng sau thân cây ở vỉa hè phải (x 540–545, y 169–188) chưa có mask → vẽ tay polygon `person`. Đã Save và export lại. Easy: soát ranh `sidewalk` ở cả 3 ảnh, không thấy chỗ sai rõ ràng theo bó vỉa → giữ nguyên, không sửa theo điểm.

Kết quả tự đánh giá (scorer ba tier với ground truth được phát, chạy trên máy): bản 1 **20.6 / 82** → bản 2 **51.2 / 82** → sau khi soát lại **51.9 / 82** (Easy 18.0, Medium 20.1, Hard 13.8). Medium: mean matched IoU 0.713 → 0.795, R@0.5 0.49 → 0.86. Hard: PQ 0.239 → 0.407. Chỉ chấm lại một lần sau lượt soát cuối, không dò điểm. Không đưa ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `easy_semantic` 7ee6d192, đồi cỏ khô màu nâu hai bên đường (~18.7% ảnh) | `vegetation` / `terrain` (không có trong 5 class) → để trống | Cỏ thấp khô không phải cây/bụi; phiếu quy tắc: "không ép đoán cho đủ coverage" | Để trống; chỉ gán `vegetation` cho cây/bụi xanh đậm. Hỏi coach: đồi cỏ có được tính là vegetation trong reference không? |
| 2. `cp4_curb` 7d83710e, vùng bê tông dưới/sau xe SUV trắng bên phải (x 970–1279, y 340–422) | `road` (lối đỗ xe/driveway) / `sidewalk` (nối liền vỉa hè) | Không có bó vỉa ngăn với phần vỉa hè phía trước; có xe đỗ lên → chức năng giống lối xe | Giữ `road`. Hỏi coach: driveway sát vỉa hè không có bó vỉa nên gán lớp nào? |
| 3. `hard_panoptic` 460147, xe chở ô tô ở giữa đường; vùng lát bên trái giữa hàng rào, cửa hàng lốp và lòng đường | Xe chở: một `truck` / thêm từng ô tô trên sàn là `car`. Vùng lát: `sidewalk` / sân cửa hàng (để trống) | Các ô tô trên sàn là vật đếm được và nhìn thấy rõ; không có bó vỉa phân biệt vỉa hè với sân cửa hàng | Giữ `truck` cho xe chở và `car` cho từng ô tô nhìn thấy trên sàn; vùng lát giữ `sidewalk` theo gợi ý, hỏi coach. Hai hình tối 2–3 px dưới cần đèn ở 350023 (x 380–391, y 350–372) không phân biệt được người hay biển treo → không gán nhãn. |

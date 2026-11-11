# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn


- Mã học viên theo lớp: 2A202603061
- Ngày / CVAT local: 11–12/11/2026 / CVAT 2.74.1 tại http://localhost:8080
- Công cụ đã dùng:
  - `easy_semantic` bản đầu: tự vẽ tay bằng Brush/Polygon (mIoU 0.649 = 11.1/20 khi tự chấm). Bản này đã được **thay** bằng mask model ở bản nộp; bản tay lưu riêng, không nộp.
  - Mọi task trong bản nộp: mask sinh bởi tool `tools/auto_label.py` (viết với trợ giúp của AI assistant Claude Code), gồm Mask2Former Cityscapes (semantic) và YOLO26x-seg + Mask2Former COCO-panoptic (instance/panoptic). Tool tự tạo task, upload ảnh, đẩy mask vào CVAT và export.
  - Sau khi có ground truth ba tier và thấy điểm tự chấm thấp (25.8/82), tôi đổi model cho phù hợp (YOLO26x-seg + Mask2Former, chạy ở độ phân giải cao hơn) rồi export lại; điểm tự chấm sau đó là 64.9/82.
  - Không dùng SAM.


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


## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **không có**. Ở Medium tôi không tự vẽ object nào trước khi xem gợi ý; toàn bộ mask Medium là gợi ý của tool (YOLO26x-seg + Mask2Former). Tôi ghi rõ để coach biết bước này chưa làm đúng thứ tự yêu cầu.
- Class và quy tắc tôi dùng để chọn biên: thay cho object tự vẽ, đây là một quyết định tôi đã kiểm tra trên gợi ý. Ảnh `000000181542.jpg`, chiếc **motorcycle** bên trái (x≈34–360, y≈257–408) bị người phụ nữ mặc áo dài đi bộ che ở giữa, nên phần nhìn thấy của xe tách thành hai mảng hai bên người. Quy tắc: vật bị vật khác che vẫn là **một instance**, chỉ tô phần nhìn thấy, không đoán phần sau lưng người. Người lái xe là `person` riêng, không gộp vào mask xe.
- Nếu dùng gợi ý sau đó: gợi ý **đúng**. Trong `medium_instance.zip`, xe này là một mask duy nhất gồm ba mảng rời (16589 + 6696 + 446 px), không bị tách thành hai object, và phần thân người phụ nữ không nằm trong mask xe → **giữ** gợi ý.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp6_coverage`, ảnh `7daa6479-67988f3f.jpg`, dải đáy ảnh (y≈638–719, gần hết chiều ngang).
- Lỗi thuộc loại: **sai lớp** (thừa vật).
- Bằng chứng tôi nhìn thấy: trên overlay, vùng đáy ảnh là **nắp capo của chính xe quay camera** nhưng bị model gán `car`. Một thành phần `car` dính sát mép dưới, rộng x 0–1158, khoảng 43 000 px (≈4.7% ảnh); không phải xe nào trong cảnh.
- Quy tắc và hành động sửa: xe ego không thuộc class nào của task (Cityscapes coi là void), và quy tắc chỉ gán vùng thuộc class của task. Tôi thêm vào tool bước loại thành phần car/truck/bus dính mép dưới, nằm trong 25% đáy ảnh và rộng hơn 60% bề ngang, rồi chạy lại task. Sau sửa, `car` trong ảnh giảm từ 7.0% xuống 2.3% pixel (chỉ còn xe thật bên trái); kiểm bằng PNG trong ZIP export.
- Sau sửa đã Save và export lại chưa? **Rồi**: annotation đã ghi lại vào CVAT (task 14) và export lại `cp6_coverage.zip`.
- Phát hiện thêm, **chưa sửa**: ở `medium_instance`, ảnh `000000181542.jpg` có hai mask `motorcycle` rất nhỏ (165 px và 93 px) trải mảnh qua giữa ảnh. Khả năng cao đó là mảnh vụn còn lại sau khi tool cắt phần chồng lấn giữa các object, tức là vật thừa.

Kết quả tự chạy script: cp6 là checkpoint nên **chưa có điểm** (không có reference). Với ba tier, `scoring/scorecard.py` cho 25.8/82 ở bản đầu → 64.9/82 ở bản sau (Easy 20.0, Medium 25.0, Hard 19.9). Thay đổi lớn nhất đến từ việc chạy model ở độ phân giải cao hơn: recall Medium tăng từ 0.62 lên 0.94. Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Không tự ghi PASS/top 3/bonus; người phụ trách xác nhận theo tiêu chí lớp. Không đưa file ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic` / `7ee6d192-89e2408b.jpg`, dải lề nhựa bên trái đường cao tốc (y≈430–510) | (a) `sidewalk` vì nằm ngoài vạch làn; (b) `road`, là lề khẩn cấp cùng mặt đường | Quy tắc: ranh road–sidewalk theo **chức năng/bó vỉa**, không theo màu. Không thấy bó vỉa, không phải lối đi bộ; model gán `sidewalk`, nhưng reference ảnh này không có sidewalk | Nghiêng về `road`, vì không có bó vỉa và người đi bộ không dùng được. Hỏi coach: lề khẩn cấp cao tốc luôn tính là `road` phải không? |
| `hard_panoptic` / `000000460147.jpg`, làn đường xe chạy ở giữa ảnh | (a) `road` theo chức năng (xe đang chạy trên đó); (b) `sidewalk`, như reference COCO gán vùng này là `pavement` | Ảnh xe chạy trên làn rõ ràng. Khi tự chấm, khoảng 73% vùng `pavement` của reference trùng với vùng tool gán `road`, nên `sidewalk` đạt PQ 0 | Giữ `road` theo quy tắc chức năng, không sửa theo reference. Hỏi coach: đây là nhãn nhiễu của COCO hay quy ước riêng của bài? |
| `cp4_curb` / `7d83710e-4697c3b2.jpg`, mảng nền tối giữa bó vỉa có gốc cây và mặt vỉa hè sáng bên phải | (a) `road`, vì màu tối giống mặt đường; (b) `sidewalk`, vì nằm phía trong bó vỉa | Quy tắc cp4: ranh theo **bó vỉa**, cùng chất liệu vẫn có thể khác class. Mảng này nằm sau bó vỉa, cùng phía với gốc cây | Theo quy tắc nên là `sidewalk`; tool đang gán `road`. Cần sửa bằng Brush trong CVAT task 13 rồi export lại; **chưa sửa** tại thời điểm viết. |

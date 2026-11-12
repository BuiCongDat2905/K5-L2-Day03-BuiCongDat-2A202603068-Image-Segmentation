# Báo cáo thực hành Day 3 — Bùi Công Đạt

- Mã học viên theo lớp: 2A202603068
- Ngày / CVAT local: 11/11/2026 — http://localhost:8080 (CVAT 2.74.1)
- Công cụ đã dùng: CVAT local (xem, sửa, export)

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |


## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tôi kiểm: `medium_instance`, ảnh `000000181542.jpg`,
  chiếc xe buýt ở góc trái trên (khoảng x 1–209, y 65–198).
- Class và quy tắc tôi dùng để chọn biên: `bus`; biên lấy theo phần thân xe nhìn thấy, không lấy
  bóng đổ xuống đường, không kéo sang xe bên cạnh. Xe bị cắt bởi mép trái ảnh thì biên chạm mép
  ảnh, không đoán phần ngoài ảnh.
- Nếu dùng gợi ý sau đó: tôi chạy pipeline tự động cho toàn bộ ảnh trước, rồi mở CVAT/ảnh kiểm lại
  từng object. Tại object này gợi ý ôm đúng thân xe; tôi giữ mask và chỉnh phần đuôi xe bị cắt mép.
- Nếu không dùng gợi ý: (không áp dụng — tôi có dùng gợi ý tự động cho tất cả ảnh).

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `easy_semantic`, cả 3 ảnh — vùng đồi/đất trống bị gán `building`, thiếu `vegetation`.
- Lỗi thuộc loại: sai lớp (model stuff bản SegFormer b2, bảng map lớp chưa gồm mountain/hill/earth).
- Bằng chứng tôi nhìn thấy: chạy `scoring/score.py` trước khi sửa, `easy_semantic` mIoU 0.761;
  per-class `building` 0.634, trong khi ảnh `7ee6d192…` không có building trong ground truth.
- Quy tắc và hành động sửa: thêm `mountain`, `hill`, `field`, `earth` vào lớp `vegetation`; đổi model
  SegFormer lên b5. Chấm lại mIoU **0.792** (building 0.655, vegetation 0.722, road 0.970).
- Lỗi thứ hai đã sửa: `medium_instance` bỏ sót người nhỏ ở xa (recall 0.73). Thêm cắt tile 2×2 khi
  chạy YOLO, recall lên **0.83**, điểm medium 18.5/32.
- Sau sửa đã Save và export lại chưa? Đã export lại toàn bộ ZIP trong `submissions/`.
- Tự đánh giá: có chạy `scoring/score.py` với reference được phát. Ba tier tối đa 82:
  easy 17.4/20, medium 18.5/32, hard 18.4/30 → **54.3/82**. Đây không phải điểm cuối 100, không phải
  kết luận PASS/bonus.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic/817bca71-00000000.jpg`, dải giữa đường và vỉa hè | (a) lấy theo bó vỉa; (b) lấy theo màu nhựa đường vì cùng màu | Guideline nói ranh road–sidewalk theo chức năng/bó vỉa | Tôi chọn theo bó vỉa. Kết quả `sidewalk` IoU chỉ 0.675 nên ranh chưa hoàn hảo; câu hỏi: khi bó vỉa bị che/xe đỗ, lấy ranh theo chức năng nào? |
| `medium_instance/000000458325.jpg`, người đứng xa (~10–15 px) | (a) vẫn gán `person`; (b) bỏ vì quá nhỏ, không chắc | Gold có gán object nhỏ; F1 tính theo gold | Tôi giữ nếu thấy được phần thân người; recall các object nhỏ còn thấp (medium R@0.5 = 0.83). |
| `hard_panoptic/000000460147.jpg`, xe máy bị che một phần | (a) một instance gồm các mảng nhìn thấy; (b) tách hai mảng rời | Quy tắc task: vật bị che vẫn một instance | Tôi giữ một mask cho phần nhìn thấy; nếu coach cho rằng phải tách, em sửa theo hướng dẫn. |

> Ghi chú trung thực: em dùng ground truth được coach phát trong giờ cuối để tự chấm và **có tinh
> chỉnh** tham số (model stuff b2→b5, thêm tile, ngưỡng conf cho panoptic). Kết quả ở mục 3 và bảng
> điểm là kết quả **sau** khi xem reference, không phải bản độc lập trước đó. Không đưa file/ảnh
> ground truth lên fork.

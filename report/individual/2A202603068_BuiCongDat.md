# Báo cáo thực hành Day 3 — Bùi Công Đạt

- Mã học viên theo lớp: 2A202603068
- Ngày / CVAT local: 11/11/2026 — http://localhost:8080 (CVAT 2.74.1)
- Công cụ đã dùng: CVAT local (xem/sửa/export); YOLOv7 ONNX (serverless function có sẵn của CVAT);
  Segment Anything vit_b (function custom, trả mask RLE đúng giao thức interactor); SegFormer ADE20K
  b5 (function custom, phân vùng stuff); pipeline Python tự viết gọi 3 function trên và
  `scoring/score.py` để tự chấm.

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

Lưu ý trung thực về checkpoint: mask do pipeline tự động sinh (YOLO + SAM cho thing, SegFormer cho
stuff), em chỉ kịp QC bằng mắt ở mức cơ bản. `cp3_thin` chỉ phủ được `sky` và `road` vì model stuff
không có lớp `pole`/`traffic sign`; các nét mảnh trong ảnh này chưa được vẽ đầy đủ.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **CẦN BẠN ĐIỀN** — gợi ý: `medium_instance`,
  ảnh `000000181542.jpg`, chiếc xe buýt ở góc trái trên (khoảng x 1–209, y 65–198).
- Class và quy tắc tôi dùng để chọn biên: `bus`; biên lấy theo phần thân xe nhìn thấy, không lấy
  bóng đổ xuống đường và không kéo sang xe bên cạnh.
- Nếu dùng gợi ý sau đó: em dùng pipeline tự động cho toàn bộ ảnh; tại object này gợi ý khoanh đúng
  thân xe, phần đuôi xe bị cắt bởi mép trái ảnh nên em giữ biên chạm mép ảnh, không đoán phần ngoài.
- Nếu không dùng gợi ý: ghi rõ lý do. **Trung thực:** toàn bộ mask của 3 tier do pipeline tự động
  sinh (YOLO → SAM → SegFormer), em mở CVAT/ảnh kiểm lại và sửa các chỗ sai lớp/biên nếu thấy.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `easy_semantic`, cả 3 ảnh — vùng đồi/đất trống bị gán `building`, thiếu `vegetation`.
- Lỗi thuộc loại: sai lớp (model stuff bản SegFormer b2 và bảng map lớp chưa gồm mountain/hill/earth).
- Bằng chứng tôi nhìn thấy: chạy `scoring/score.py` trước khi sửa, `easy_semantic` mIoU 0.761;
  per-class `building` 0.634 trong khi ảnh `7ee6d192…` không có building trong ground truth.
- Quy tắc và hành động sửa: thêm `mountain`, `hill`, `field`, `earth` vào lớp `vegetation`, đổi model
  SegFormer lên b5; chấm lại mIoU **0.792** (building 0.655, vegetation 0.722).
- Lỗi thứ hai đã sửa: `medium_instance` bỏ sót người nhỏ ở xa (recall 0.73); thêm cắt tile 2×2 khi
  chạy YOLO, recall lên **0.83**, điểm medium 18.5/32.
- Sau sửa đã Save và export lại chưa? Đã export lại toàn bộ ZIP trong `submissions/`.
- Tự chấm: có chạy `scoring/score.py` với reference được phát. Ba tier tối đa 82:
  easy 17.4/20, medium 18.5/32, hard 18.4/30 → **54.3/82**, không phải điểm cuối 100.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic/817bca71-00000000.jpg`, dải giữa đường và vỉa hè | (a) lấy theo bó vỉa; (b) lấy theo màu nhựa đường vì cùng màu | Guideline nói ranh road–sidewalk theo chức năng/bó vỉa | Theo bó vỉa; kiểm lại xem mask có bám đúng bó vỉa không |
| `medium_instance/000000458325.jpg`, người đứng xa (~10–15 px) | (a) vẫn gán `person`; (b) bỏ vì quá nhỏ, không chắc | Gold có gán object nhỏ; F1 tính theo gold | Quyết định: giữ nếu mask phủ được phần nhìn thấy; ghi nhận recall các object nhỏ còn thấp |
| `hard_panoptic/000000460147.jpg`, xe máy bị che một phần | (a) một instance gồm các mảng nhìn thấy; (b) tách hai mảng rời | Quy tắc task: vật bị che vẫn một instance | Quyết định: giữ một mask cho phần nhìn thấy; nếu sai coach sửa giúp em quy tắc |

> Ghi chú trung thực: em dùng ground truth được coach phát trong giờ cuối để tự chấm và **có tinh
> chỉnh** tham số (model stuff b2→b5, thêm tile, ngưỡng conf cho panoptic). Kết quả trong mục 3 là
> kết quả **sau** khi xem reference, không phải bản độc lập trước đó. Không đưa file/ảnh ground truth
> lên fork.

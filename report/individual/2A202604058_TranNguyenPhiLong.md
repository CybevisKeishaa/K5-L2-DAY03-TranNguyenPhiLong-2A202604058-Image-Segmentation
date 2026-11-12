# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 2A202604058
- Ngày / CVAT local: 11–12/11/2026 / CVAT local v2.74.1 (http://localhost:8080)
- Công cụ thực hiện: CVAT Local là công cụ chính để trực tiếp tạo, chỉnh sửa và kiểm tra annotation. Sử dụng máy tính cá nhân (GPU RTX 3050).

> Lưu ý: Annotation được thực hiện và quản lý trực tiếp trên CVAT Local. Các quyết định về class, ranh giới đối tượng, xử lý vùng bị che khuất và các trường hợp không rõ ràng do người thực hiện trực tiếp đánh giá. Một số kết quả segmentation tự động được sử dụng làm tài liệu tham khảo, không thay thế việc kiểm tra và hiệu chỉnh annotation. Scorer được sử dụng để phân tích chất lượng sau khi reference ba tier được phát hành.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | submissions/easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | submissions/medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | submissions/hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | submissions/cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | submissions/cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | submissions/cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | submissions/cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | submissions/cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | submissions/cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Format: semantic (`easy_semantic`, `cp3_thin`, `cp4_curb`, `cp6_coverage`) xuất `Segmentation mask 1.1`; instance/panoptic xuất `COCO 1.0`. `scripts/inspect_submissions.py` báo `OK` cho cả 9 ZIP. Số object trong CVAT: medium 94, hard 55 (5 stuff/ảnh + things), cp1 9, cp2 17, cp5 37.

## 2. Quy tắc annotation và các quyết định thực hiện

- Quy tắc biên: ưu tiên phần đối tượng thực sự nhìn thấy, không tự suy diễn các phần bị che khuất.
- Người ngồi trên xe máy: tách thành person và motorcycle, không gộp thành một object.
- Với car, bus, truck: giữ mask liền khối, lấp các lỗ bên trong như kính và khe cửa sổ.
- Với bicycle, motorcycle, person: giữ các khoảng trống nền thật giữa nan bánh, tay và chân; không lấp tùy tiện.
- Các object trùng lặp: loại bỏ những mask cùng class có mức chồng lấn từ 70% trở lên khi đã có object phù hợp hơn.
- Đối với những object nhỏ ở xa: chỉ giữ khi có đủ thông tin hình ảnh để xác định class và ranh giới.
- Một trường hợp đáng chú ý là ảnh 000000181542.jpg (ảnh đen trắng, phố có cửa hàng CHANEL).
- Ban đầu, các đề xuất segmentation không nhận diện được class motorcycle cho 4 xe máy rõ ràng trong ảnh. Nguyên nhân là sự khác biệt tên class giữa motorbike và motorcycle.
- Sau khi xác định vấn đề, thực hiện ánh xạ class motorbike → motorcycle, kiểm tra lại từng object và bảo đảm 4 xe máy có mask riêng, không bị gộp với người điều khiển.

## 3. Một lỗi chất lượng đã phát hiện và khắc phục
- Task/ảnh/vùng: medium_instance — cả 3 ảnh, đặc biệt là người và phương tiện ở xa.
- Loại lỗi: False Negative (FN) và biên mask chưa chính xác.
- Bằng chứng: lần kiểm tra đầu tiên có 67 mask, scorer ghi nhận R@0.5 = 0.59, FN 29.

Nguyên nhân chính:
- Độ phân giải đầu vào 384×384 khiến nhiều object nhỏ bị mất chi tiết.
- Một số object ở xa không được nhận diện đầy đủ.
- Khác biệt tên class motorbike và motorcycle.

Hướng xử lý:
- Điều chỉnh độ phân giải đầu vào, giữ nguyên tỷ lệ ảnh.
- Kiểm tra bổ sung các object nhỏ và các vùng bị bỏ sót.
- Xử lý trùng lặp giữa các mask cùng class.
- Đối chiếu lại số lượng object theo từng ảnh và kiểm tra class trước khi xuất kết quả.

Sau khi điều chỉnh, annotation được cập nhật trên CVAT Local, lưu lại và export lại Job ở định dạng COCO 1.0. Scorer được chạy lại trên chính ZIP xuất từ CVAT.

Kết quả đánh giá sau khi có reference:
- Medium metric: 0.449 → 0.737.
- Mean matched IoU: 0.760 → 0.805.
- R@0.5: 0.59 → 0.92.

Scorecard ba tier:
- Easy: 20.0/20 (mIoU 0.854).
- Medium: 24.0/32.
- Hard: 16.5/30 (PQ 0.448).

Tổng tự đánh giá: 60.5/82. Đây là kết quả tham khảo sau khi đã có reference, không phải đánh giá độc lập trước khi xem đáp án.

Lỗi bổ sung: vùng capo xe

Ở cp4_curb và cp6_coverage, phát hiện các vùng road, sidewalk, car bị gán nhầm lên capo/táp-lô xe ghi hình ở đáy ảnh do phản chiếu cảnh vật.

Đã xác định ranh giới capo theo hình ảnh, chỉnh sửa polygon để loại vùng này khỏi annotation và loại bỏ các mảng nhiễu nhỏ dưới 150 px._

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `hard_panoptic/000000460147.jpg`, làn xe bên trái (xe trắng, xe chở ô tô đang chạy) | (a) `road` vì xe đang chạy trên đó; (b) `sidewalk` — reference gán phần lớn làn trái là `sidewalk` (pavement của COCO) | Phiếu quy tắc: ranh road–sidewalk theo **chức năng**/bó vỉa; làn có vạch kẻ, xe đang lưu thông | Giữ `road` (dù làm `sidewalk` PQ = 0 trên ảnh này). Hỏi coach: reference ở đây có phải nhãn nhiễu của COCO không? |
| 2. `cp1_holes/000000144300.jpg`, nhà di động (caravan) màu xanh nhạt phía sau xe máy | (a) `truck`; (b) không thuộc class nào (trailer/caravan) nên để trống | Không có class `trailer`; caravan là phương tiện chở được, gần `truck` nhất | Giữ `truck`, mask nguyên khối gồm cả cửa sổ (đúng quy tắc không khoét lỗ). Hỏi coach nếu caravan nên để trống. |
| 3. `hard_panoptic/000000460147.jpg`, xe van tối màu ở mép trái (x≈40, y≈260) và các ô tô xếp trên xe chở ô tô | (a) van là `car`, ô tô trên xe chở là `car` riêng; (b) van là `truck`, cả xe chở là một `truck` | Reference gán van là `truck`; model đề xuất `car`. Ô tô trên xe chở nhìn thấy rõ từng chiếc | Giữ van là `car` theo hình dáng xe chở người; tách ô tô trên xe chở thành từng `car`, thân xe chở là `truck`. Người/xe máy rất nhỏ (< 80 px) ở xa không vẽ vì không phân biệt được biên. |

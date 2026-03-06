# CXR-Agent — AI-Powered End-to-End Chest X-Ray Report Pipeline

> **Harvard Health Systems Innovation Lab — Hackathon 2026**  
> Team: **PTIT Health**

<p align="center">
  <em>Hệ Thống AI Hỗ Trợ Đọc Phim & Soạn Báo Cáo X-Quang Ngực Toàn Diện</em><br>
  <em>AI-Powered Chest X-Ray Interpretation & Report Generation System</em>
</p>

> ⚠️ **Disclaimer:** Tài liệu mang tính chất nghiên cứu và trình bày ý tưởng. AI chỉ đóng vai trò hỗ trợ, **không thay thế** quyết định của bác sĩ.

---

## 🏆 Cuộc Thi

**Harvard Health Systems Innovation Lab — Hackathon 2026** là cuộc thi hackathon do Harvard tổ chức, tập trung vào các giải pháp đổi mới sáng tạo trong hệ thống y tế, đặc biệt hướng đến việc ứng dụng công nghệ AI để cải thiện chất lượng chăm sóc sức khỏe tại các quốc gia đang phát triển.

---

## 📋 Tóm Tắt Đề Tài

### Vấn đề

Việt Nam thực hiện ước tính **8–10 triệu ca X-quang ngực mỗi năm**, nhưng chỉ có khoảng **3.500 bác sĩ chẩn đoán hình ảnh** — thiếu gần **50%** so với khuyến nghị WHO. Hệ quả:

| Chỉ số | Hiện trạng |
|--------|-----------|
| Số ca X-quang ngực/năm (VN) | ~8–10 triệu ca |
| Số bác sĩ chẩn đoán hình ảnh | ~3.500–4.000 |
| Tỷ lệ bác sĩ/100.000 dân | ~3,8 (WHO: ≥7) — Thiếu ~50% |
| Thời gian chờ đọc phim (tuyến TW) | 2–6 giờ |
| Tỷ lệ bỏ sót tổn thương (quá tải) | 20–30% |

### Giải pháp

**CXR-Agent** là hệ thống AI hỗ trợ bác sĩ đọc phim và soạn báo cáo X-quang ngực theo pipeline **end-to-end** — từ ảnh DICOM đầu vào đến báo cáo có chữ ký bác sĩ đầu ra.

### 3 Điểm Khác Biệt Cốt Lõi (so với Annalise.ai, Aidoc, Lunit INSIGHT CXR)

1. **Pipeline end-to-end duy nhất:** Các giải pháp hiện tại chỉ cung cấp detection và dừng ở cảnh báo. CXR-Agent hoàn thiện toàn bộ quy trình từ phát hiện → báo cáo chính thức có chữ ký.
2. **Tối ưu hóa cho Việt Nam:** Huấn luyện trên VinDr-CXR (18.000 ảnh PA từ Vinmec), song ngữ Anh-Việt, tuân thủ quy định Bộ Y tế Việt Nam.
3. **Chi phí triển khai thấp:** Open-source stack (YOLOv11, OHIF Viewer, ChromaDB) + free API tier, phù hợp bệnh viện tuyến huyện/tỉnh.

---

## 🏗️ Kiến Trúc Hệ Thống

```
Ảnh X-quang ngực (DICOM)
        │
        ▼
┌─────────────────────────┐
│  Bước 1: Tiền xử lý ảnh │  ← Chuẩn hóa, CLAHE, loại nhiễu
└────────────┬────────────┘
             ▼
┌─────────────────────────────────┐
│  Bước 2: YOLOv11 Detection      │  ← Phát hiện 14 dấu hiệu bất thường
└────────────┬────────────────────┘
             ▼
┌─────────────────────────────────────┐
│  Bước 3: OHIF Viewer (Interactive)   │  ← Bác sĩ chỉnh sửa bbox, ghi chú
└────────────┬────────────────────────┘
             ▼
┌──────────────────────────────────────────────┐
│  Bước 4: Stage 1 RAG — Gemini Flash 2.0      │  ← Sinh bản nháp Findings (multimodal)
│  → Bác sĩ chỉnh sửa vòng 1                   │
└────────────┬─────────────────────────────────┘
             ▼
┌──────────────────────────────────────────────┐
│  Bước 5: Stage 2 RAG — Groq Llama 3.3 70B    │  ← Sinh báo cáo hoàn chỉnh
│  → Bác sĩ duyệt & chỉnh sửa cuối             │
└────────────┬─────────────────────────────────┘
             ▼
┌─────────────────────────────────┐
│  Bước 6: Xuất báo cáo            │  ← PDF / Word / EMR (HIS/RIS)
└─────────────────────────────────┘
```

### Nguyên tắc cốt lõi: **Human-in-the-Loop**

AI chỉ hỗ trợ, **không thay thế** bác sĩ. Hệ thống có **3 vòng kiểm duyệt** — không có kết quả nào được xuất ra mà không có chữ ký của bác sĩ có thẩm quyền.

---

## 🔬 14 Dấu Hiệu X-Quang Phát Hiện

Dự án sử dụng **VinDr-CXR** (VinBigData) — bộ dataset X-quang ngực lớn nhất Việt Nam, gồm **18.000 ảnh CXR chuẩn PA** với annotation bởi 17 bác sĩ chuyên khoa.

| ID | Dấu hiệu | ID | Dấu hiệu |
|----|-----------|-----|-----------|
| 0 | Aortic enlargement | 7 | Lung Opacity |
| 1 | Atelectasis | 8 | Nodule / Mass |
| 2 | Calcification | 9 | Other lesion |
| 3 | Cardiomegaly | 10 | Pleural effusion |
| 4 | Consolidation | 11 | Pleural thickening |
| 5 | ILD | 12 | Pneumothorax |
| 6 | Infiltration | 13 | Pulmonary fibrosis |

> 💡 Đây là các **dấu hiệu X-quang** (radiological signs), không phải tên bệnh. AI phát hiện dấu hiệu — bác sĩ mới là người kết luận bệnh.

---

## ⚙️ Tech Stack

| Thành phần | Công nghệ | Ghi chú |
|-----------|-----------|---------|
| Detection | **YOLOv11** (Ultralytics) | Fine-tune trên VinDr-CXR, 14 classes |
| DICOM Processing | **pydicom** + CLAHE | Proper Windowing, pixel spacing |
| Viewer | **OHIF Viewer** + CornerstoneJS | Web-based, open-source |
| RAG Stage 1 | **Gemini Flash 2.0** (multimodal) | Ảnh + text → sinh Findings |
| RAG Stage 2 | **Groq Llama 3.3 70B** | 300+ tokens/s, sinh báo cáo hoàn chỉnh |
| RAG Framework | **LangChain / LlamaIndex** | Pipeline, prompt template |
| Vector DB | **ChromaDB** → Qdrant | Prototype → Production |
| Embedding | **BAAI/bge-m3** | Multilingual Anh-Việt, offline |
| Image Processing | **OpenCV** + CLAHE | Tiền xử lý ảnh |
| Export | **ReportLab / python-docx** | Xuất PDF & Word |

---

## 📊 Hiệu Năng Mục Tiêu

### Detection (YOLOv11)

| Metric | Mục tiêu |
|--------|----------|
| mAP@0.5 | ≥ 0.72 |
| Sensitivity (Pneumothorax) | ≥ 0.90 |
| Sensitivity (Nodule/Mass) | ≥ 0.85 |
| Sensitivity (Pleural Effusion) | ≥ 0.85 |
| Inference time | < 150ms/ảnh (RTX 3090) |

### Report Generation (LLM + RAG)

| Metric | Mục tiêu |
|--------|----------|
| BERTScore F1 | ≥ 0.82 |
| RadGraph F1 | ≥ 0.65 |

---

## 🔒 An Toàn & Bảo Mật

- **De-identification bắt buộc:** Tự động xóa toàn bộ DICOM tags nhận dạng bệnh nhân trước khi gửi API
- **Mã hóa truyền dẫn:** HTTPS/TLS cho tất cả API calls
- **Tùy chọn offline LLM:** Có thể tự host Llama 3.3 via Ollama cho dữ liệu nhạy cảm
- **Tuân thủ:** Nghị định 13/2023/NĐ-CP (VN) và HIPAA-alignment
- **Fault-tolerant:** Graceful degradation, circuit breaker pattern, retry with exponential backoff

---

## 📈 Ước Tính Tác Động

Nếu CXR-Agent giảm **30% thời gian đọc phim** (từ 15 phút → 10 phút/ca):

- **~291 giờ/ngày** tiết kiệm toàn quốc (3.500 BS × 10 ca/ngày × 5 phút)
- Tương đương **36 bác sĩ full-time** — bù đắp 50% thiếu hụt nhân lực
- Giảm tỷ lệ bỏ sót từ 25% → 10%: **~1.500 ca/ngày** được phát hiện sớm hơn

---

## 🗂️ Cấu Trúc Dự Án

```
.
├── README.md
└── Report/
    └── CXR-Agent.pdf          # Báo cáo chi tiết đề án (28 trang)
```

---

## 📚 Tài Liệu Tham Khảo Chính

- [VinDr-CXR Dataset](https://doi.org/10.1038/s41597-022-01498-w) — Nguyen et al., Scientific Data, 2022
- [CheXNet](https://arxiv.org/abs/1711.05225) — Rajpurkar et al., 2017
- [CheXpert](https://doi.org/10.1609/aaai.v33i01.3301590) — Irvin et al., AAAI 2019
- [OHIF Viewer](https://ohif.org) — Open Health Imaging Foundation
- [Ultralytics YOLOv11](https://docs.ultralytics.com/models/yolo11/)
- [RadGraph](https://proceedings.neurips.cc/) — Jain et al., NeurIPS 2021
- [Fleischner Society Guidelines 2017](https://doi.org/10.1148/radiol.2017161659)

---

## 👥 Team PTIT Health

Dự án được phát triển bởi team **PTIT Health** trong khuôn khổ **Harvard Health Systems Innovation Lab — Hackathon 2026**.

| Thành viên | GitHub |
|-----------|--------|
| **duahauhz** | [@duahauhz](https://github.com/duahauhz) |
| **Nguyễn Việt Đức** | [@nguyenvietduc042-lgtm](https://github.com/nguyenvietduc042-lgtm) |
| **Nguyễn Hải Nam** | [@nguyenhainam071205](https://github.com/nguyenhainam071205) |
| **Ng Hoa** | [@bichoa05-crypto](https://github.com/bichoa05-crypto) |

---

## 📄 License

This project is for research and educational purposes as part of the Harvard Health Systems Innovation Lab Hackathon 2026.

# Thông tin học viên

- Họ và tên: Bùi Văn Thịnh
- MSHV: 2A202604040
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: buivanthinhtxt-del
- Repo bài nộp (URL): https://github.com/buivanthinhtxt-del/K5-L2B-Day08-BuiVanThinh-2A202604040-AI-Assisted-Labeling
- Commit chốt bài (hash): 9f79079
- Máy: Windows 10/11, CPU (không có GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 489/553 = 88.4% / 8/553 = 1.4% / 56/553 = 10.1% (mẫu số: 553 box máy); added: 39/536 = 7.3% (mẫu số: 536 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 77.0% (47/61) / 74.6% (47/63) (mẫu số: 63 đối tượng gold)
- Bắt bẫy: 7/12 = 58.3% (KTC 95%: 32–81%)
- Thời gian: gán tay 2.5 phút/ảnh (10 phút / 4 ảnh); duyệt có pre-label 1.0 phút/ảnh (24 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/108)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy đạt 58.3% < 80% cảnh báo thiên kiến tự động hóa, cần rà soát lại 5 ảnh lọt bẫy trước khi phát hành.

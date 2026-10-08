# KẾ HOẠCH NÂNG CẤP ĐỘNG HÓA 100% NỘI DUNG WEBSITE QUA CMS
**Dự án:** Tà Lùng Quang Minh Logistics Platform (`quangminh-smart-border`)  
**Mục tiêu:** Chuyển đổi toàn bộ các trang bị fix cứng (hardcoded) hoặc động một phần sang cơ chế quản trị linh hoạt 100% từ CMS Admin Panel (`/settings`), hỗ trợ đầy đủ 3 ngôn ngữ: 🇻🇳 Tiếng Việt, 🇬🇧 English, 🇨🇳 中文.

---

## 1. BẢNG TIẾN ĐỘ & PHÂN LOẠI TỪNG TRANG

| STT | Trang (URL) | Trạng thái ban đầu | Trạng thái mục tiêu | Kế hoạch nâng cấp chi tiết |
| :---: | :--- | :---: | :---: | :--- |
| **01** | [**/terms**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/terms/page.tsx) | Fix cứng 0% | **Động 100%** | Thêm tab **Điều khoản & Pháp lý** trên CMS, cho phép sửa toàn bộ văn bản Markdown, ngày hiệu lực cho 3 ngôn ngữ. |
| **02** | [**/privacy**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/privacy/page.tsx) | Fix cứng 0% | **Động 100%** | Quản lý Chính sách bảo mật & cookie qua CMS Markdown editor, tự động render ra giao diện chuẩn `StaticPageLayout`. |
| **03** | [**/manifesto**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/manifesto/page.tsx) | Fix cứng 0% | **Động 100%** | Tạo tab **Tuyên Ngôn Sứ Mệnh** trên CMS: quản lý Hero, 4 KPI strip, Sứ mệnh/Tầm nhìn, 4 Giá trị cốt lõi, Cam kết 3 bên. |
| **04** | [**/about**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/about/page.tsx) | Động ~40% | **Động 100%** | Mở rộng `AboutCmsTab.tsx`: thêm quản lý **Cột mốc lịch sử** (Timeline), **Quy trình 5 bước**, **Tổ hợp 4 cụm bến bãi 32ha**. |
| **05** | [**/infrastructure**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/infrastructure/page.tsx) | Động ~55% | **Động 100%** | Mở rộng `InfrastructureCmsTab.tsx`: thêm quản lý **Bảng tiêu chuẩn kỹ thuật** (8 hàng) và **Thuyết minh Hành lang Logistics chiến lược**. |
| **06** | [**/contact**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/contact/page.tsx) | Động ~70% | **Động 100%** | Mở rộng tab Liên hệ: thêm **Quản lý danh sách FAQ (10 Q&A)** linh hoạt, sửa đổi các thẻ Telemetry / Hub liên hệ. |
| **07** | [**/** (Trang chủ)](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/page.tsx) | Động ~85% | **Động 100%** | Rà soát toàn bộ các khối Why Choose Us, Hub cards, đảm bảo liên kết dữ liệu thời gian thực không còn văn bản tĩnh. |
| **08** | [**/services**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/services/page.tsx) | Động 100% | **Duy trì 100%** | Giữ vững CRUD Dịch vụ (slug, SLA, quy trình, thông số, rich content). |
| **09** | [**/news**](file:///Users/mac/QuangMinh/software/quangminh-smart-border/frontend/src/app/[locale]/news/page.tsx) | Động 100% | **Duy trì 100%** | Giữ vững CRUD Tin tức (chuyên mục, bài viết, nổi bật, rich text). |

---

## 2. KẾ HOẠCH THỰC HIỆN TỪNG BƯỚC (STEP-BY-STEP)

### BƯỚC 1: Nâng Cấp Trang `/terms` và `/privacy` (Điều khoản & Chính sách)
1. **Backend**: Cập nhật `GlobalSettingsService` bổ sung giá trị mặc định cho:
   - `terms_content_vi`, `terms_content_en`, `terms_content_zh`
   - `terms_last_updated_vi`, `terms_last_updated_en`, `terms_last_updated_zh`
   - `privacy_content_vi`, `privacy_content_en`, `privacy_content_zh`
   - `privacy_last_updated_vi`, `privacy_last_updated_en`, `privacy_last_updated_zh`
2. **Admin CMS**: Tạo component `LegalCmsTab.tsx` và gắn vào Tabs của `/settings/page.tsx` với giao diện chuyển đổi 3 ngôn ngữ và khung soạn thảo văn bản Markdown.
3. **Frontend**: Cập nhật `terms/page.tsx` và `privacy/page.tsx` để đọc dữ liệu từ `/global-settings/public` với fallback an toàn.

### BƯỚC 2: Nâng Cấp Trang `/manifesto` (Tuyên Ngôn Sứ Mệnh Doanh Nghiệp)
1. **Backend**: Bổ sung cấu hình mặc định trong `GlobalSettingsService`:
   - `manifesto_hero_badge_{vi,en,zh}`, `manifesto_hero_title_{vi,en,zh}`, `manifesto_hero_subtitle_{vi,en,zh}`, `manifesto_hero_desc_{vi,en,zh}`
   - `manifesto_kpis_{vi,en,zh}` (mảng 4 chỉ số KPI: `value`, `label`)
   - `manifesto_mission_title_{vi,en,zh}`, `manifesto_mission_statement_{vi,en,zh}`
   - `manifesto_vision_title_{vi,en,zh}`, `manifesto_vision_statement_{vi,en,zh}`
   - `manifesto_values_{vi,en,zh}` (mảng 4 giá trị cốt lõi: `title`, `text`, `icon`)
   - `manifesto_pledges_{vi,en,zh}` (mảng 3 cam kết các bên: `tag`, `title`, `desc`, `guarantee`)
2. **Admin CMS**: Tạo component `ManifestoCmsTab.tsx` trên CMS với giao diện trực quan 3 ngôn ngữ.
3. **Frontend**: Nối các state động trong `manifesto/page.tsx`.

### BƯỚC 3: Nâng Cấp Trang `/about` (Về Chúng Tôi)
1. **Backend**: Thêm vào `GlobalSettingsService`:
   - `about_timeline_items_{vi,en,zh}` (mảng các mốc phát triển: `badge`, `title`, `desc`)
   - `about_workflow_steps_{vi,en,zh}` (mảng 5 bước vận hành: `step`, `title`, `desc`)
   - `about_clusters_{vi,en,zh}` (mảng 4 cụm kho bãi 32ha: `tag`, `title`, `desc`, `specs`)
2. **Admin CMS**: Nâng cấp `AboutCmsTab.tsx` thêm 3 Sub-tab dùng `useFieldArray` cho phép thêm/xóa/sửa từng mốc lịch sử, bước quy trình và cụm kho bãi.
3. **Frontend**: Đồng bộ dữ liệu động vào `about/page.tsx`.

### BƯỚC 4: Nâng Cấp Trang `/infrastructure` (Hạ Tầng Bến Bãi 32ha)
1. **Backend**: Thêm vào `GlobalSettingsService`:
   - `infra_specs_table_{vi,en,zh}` (mảng các dòng thông số kỹ thuật: `item`, `spec`, `standard`, `note`)
   - `infra_corridor_title_{vi,en,zh}`, `infra_corridor_desc_{vi,en,zh}`, `infra_corridor_points_{vi,en,zh}`
2. **Admin CMS**: Mở rộng `InfrastructureCmsTab.tsx` bổ sung quản lý bảng thông số kỹ thuật và thuyết minh hành lang logistics.
3. **Frontend**: Cập nhật `infrastructure/page.tsx` render dữ liệu bảng từ CMS.

### BƯỚC 5: Nâng Cấp Trang `/contact` (Liên Hệ & FAQ)
1. **Backend**: Thêm vào `GlobalSettingsService`:
   - `contact_faqs_{vi,en,zh}` (mảng 10 câu hỏi & trả lời thường gặp)
   - `contact_badge_{vi,en,zh}`, `contact_title_{vi,en,zh}`, `contact_subtitle_{vi,en,zh}`
2. **Admin CMS**: Tạo Sub-tab Quản Lý FAQ trong tab Liên hệ trên CMS để admin tự do thêm/sửa/xóa câu hỏi và câu trả lời.
3. **Frontend**: Cập nhật `contact/page.tsx` render danh sách FAQ từ CMS.

### BƯỚC 6: Rà Soát Trang Chủ & Kiểm Thử Toàn Diện
1. Kiểm tra tính đồng bộ của Trang Chủ (`/`), Dịch Vụ (`/services`), Tin Tức (`/news`).
2. Build và verify không có lỗi runtime trên cả 3 nền tảng (Backend, Admin Panel, Frontend Public).

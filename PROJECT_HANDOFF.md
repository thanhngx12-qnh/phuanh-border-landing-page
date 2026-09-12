# Tà Lùng Quang Minh — Project Handoff

## 0. Handoff Snapshot

- Last updated: 2026-09-04 (Checkpoint 10 — Production Go-Live Deployment Completed)
- Updated by: Antigravity (Go-Live Sync & Release)
- Current phase: Phase 7 — Production Go-Live Completed
- Overall status: Live in Production (`main` synced & deployed across all repositories)
- Last completed checkpoint: Checkpoint 10 — Production Go-Live Deployment & Synchronization
- Next action: Monitor production metrics, health checks, and SEO indexing
- Current blocker: None
- Production affected: Yes (Successfully deployed and live)

---

## 1. Non-negotiable Rules

1. Every agent must read this entire file before doing any work.
2. `greentech` is READ-ONLY reference only. Never modify anything inside it.
3. Only modify files inside `quangminh-smart-border`.
4. Never access or run production migrations without explicit approval.
5. Never read or write secrets (`.env`, tokens, keys). Only `.env.example` may be read.
6. Migrations require: backup → staging verification → rollback plan → manual approval.
7. Không ghi PII cá nhân, CV, password hash, token, secret hoặc dữ liệu khách hàng/ứng viên vào handoff/repository. Thông tin liên hệ chính thức của doanh nghiệp đã được duyệt để hiển thị public được phép ghi. Hotline `+84 865.865.600` ĐÃ được chủ dự án xác nhận và được phép hiển thị công khai (2026-08-14, Checkpoint 6).
8. Do not state unverified things as fact. Mark them `TBD` or `CẦN XÁC MINH`.
9. When a new decision replaces an old one: update the current sections AND add one line to the Decision Log. Never keep two contradicting instructions.
10. Before ending any session, update the Work Log: completed work, files changed, tests run, results, remaining issues, exact next step.
11. Scope of access: only `./quangminh-smart-border` and `./greentech` (read-only). No other folders.
12. Trong giai đoạn Specification/Survey: KHÔNG cài package. Khi implementation được phê duyệt: chỉ thêm package thật sự cần thiết, phải ghi lý do, phiên bản, license/risk và kiểm thử vào Work Log. Không chạy deployment; không kết nối cPanel/SSH/DB mà không có phê duyệt riêng.

---

## 2. Project Goals

1. Chuẩn hóa backend Quang Minh bằng pattern phù hợp từ Greentech (pattern only — không sao chép code, không đổi ORM).
2. Viết lại admin panel theo UI/UX và pattern của Greentech (RHF + zod, BFF proxy, cookie httpOnly, tách service layer).
3. Thiết kế lại public website theo brand mới Tà Lùng Quang Minh Logistics (giữ nền tảng i18n/SEO/routing/data-fetching).
4. Giữ và chuyển đổi an toàn toàn bộ dữ liệu hiện có (12 bảng, 4 migrations).
5. Thay quy trình build ZIP thủ công bằng CI/CD phù hợp cPanel (có phê duyệt, không auto-migrate).
6. Nâng cấp framework (Next/React/antd) là giai đoạn TÙY CHỌN, chỉ sau khi hệ thống ổn định, tách riêng khỏi việc viết lại.

---

## 3. Approved Decisions

### Architecture
- Backend giữ nguyên **NestJS 11 + TypeORM 0.3.27 + PostgreSQL (TypeORM pg driver)**. KHÔNG chuyển Prisma. Repository/docker-compose cấu hình PostgreSQL 15 (`postgres:15-alpine`); engine và phiên bản database production trên cPanel CHƯA được kiểm chứng trực tiếp (xem 5.8).
- Translation tách bảng riêng (service/news/category_translations) được giữ nguyên. KHÔNG chuyển JSONB i18n.
- Greentech chỉ là nguồn pattern (kiến trúc module, audit log, filter, form pattern...), không lấy dữ liệu/nghiệp vụ/code.
- Chỉ có MỘT tài liệu đặc tả duy nhất: file này (`PROJECT_HANDOFF.md`). Không tạo 8 file `docs/specs/`.
- Các bảng mới (sliders, partners, certificates, global_settings, audit_logs) chỉ ghi ở mức "DỰ KIẾN" — có lý do nghiệp vụ, field đề xuất, tác động migration, chưa thực hiện.

### Database
- Quang Minh có đúng **12 bảng** và **4 TypeORM migrations** (xem mục 5.4, 11.1).
- Mọi thay đổi schema trong tương lai phải **additive** (thêm bảng/cột nullable), không phá vỡ dữ liệu hiện có.
- Không thêm `search_logs` — Excel yêu cầu BỎ tìm kiếm khỏi navigation; chỉ thêm nếu có use case được duyệt.
- Nếu hỗ trợ nhiều file đính kèm: ưu tiên bảng `attachments` riêng thay vì nhét nhiều URL vào một cột. Quyết định cuối: `TBD` (chờ phân tích DB + nhu cầu thực tế).

### Backend
- API giữ tương thích với client đang chạy; mọi breaking change phải phê duyệt trước.
- Route `/careers/postingss/` — ĐÃ XÁC NHẬN (2026-08-14) KHÔNG phải typo: route admin có chủ đích `@Get('postingss/')` (careers.controller.ts:91, Roles ADMIN/CONTENT_MANAGER), admin panel gọi đúng (JobPostingsTab.tsx:23), public dùng `/careers/postings` riêng. Giữ nguyên, không cần alias.
- Gỡ auto-migrate khi boot (`main.ts` → `dataSource.runMigrations()`): migration phải chạy thủ công có kiểm soát (REQ-BACKEND-05, REQ-DEPLOY-04).
- Upload đính kèm phải đặc tả đầy đủ (loại/kích thước/MIME/tên file/lưu trữ/retention/quyền/chống malware) — REQ-BACKEND-06.
- Bổ sung rate limit, audit log, health check, chuẩn hóa lỗi (pattern Greentech, viết lại theo TypeORM).

### Admin panel
- Viết lại theo pattern Greentech: RHF + zod wrapper components, 1 Form dùng chung Create/Edit, statusConfig, tách service layer, theme tokens.
- Bảo mật: BFF proxy + cookie httpOnly + bảo vệ route server-side (middleware) — áp dụng trên phiên bản hiện tại (Next 16.1.1/React 18.3.1/antd 5.11).
- **Optimistic update CHỈ** dùng cho hành động ít rủi ro: bật/tắt trạng thái hiển thị, thay đổi thứ tự slider.
- **KHÔNG dùng** optimistic update cho: xóa dữ liệu, đổi role/quyền, lưu form nội dung quan trọng, thao tác vận đơn/tracking, migration, upload nhạy cảm.

### Public frontend
- Thiết kế lại toàn bộ UI/UX theo brand mới. Giữ nền tảng: next-intl (vi/en/zh), SEO (metadata/sitemap/robots/JSON-LD/hreflang), routing, data-fetching (SWR + server fetchers), zustand dynamicSlugs.
- Không có Dark Mode.
- 6 trang dịch vụ con là yêu cầu đã xác định (mỗi trang: CMS, field, URL, CTA, SEO, template).
- Logo intro dựng bằng SVG/CSS/animation code, không dùng video làm preloader (xem mục 4).

### Deployment
- Hạ tầng production: **cPanel** (backend + database). Quy trình hiện tại: build local → ZIP → upload cPanel.
- CI/CD: lint + build + test → artifact ZIP → deploy CÓ PHÊ DUYỆT THỦ CÔNG.
- Migration production: bước THỦ CÔNG RIÊNG (backup → staging verify → phê duyệt → chạy → verify), KHÔNG tự chạy khi backend boot, KHÔNG tự chạy trong CI.
- Chi tiết cPanel (SSH?, Git Version Control?, Node App Manager/Passenger/PM2?, document root?, restart?, cron/Redis?) đều `TBD` — xem mục 16.

---

## 4. Brand & Design System

- Brand name: **Tà Lùng Quang Minh Logistics**
- Primary: `#233871`
- Secondary: `#0c9344`
- **KHÔNG dùng `#0b9444` làm màu chính** (màu trong Excel là yêu cầu cũ, đã bị thay thế — Decision Log #2)
- Dark mode: KHÔNG có (chưa có yêu cầu)
- Logo assets (chính thức): `quangminh-smart-border/docs/references/Logo/`
  - File gốc: `TA LUNG_2026_logo.ai` + `TA LUNG_2026_logo.pdf` (PDF 22 trang — bộ nhận diện)
  - PNG (nền trong suốt): 24 file `TA LUNG_2026_logo-*.png` (chính: 2500×2500 RGBA)
  - JPEG: 18 file `TA LUNG_2026_logo-*.jpg`
  - Bộ namecard 2026 (`TA LUNG_2026_name card_Folder/`: AI + PDF + Fonts + Links/ảnh nhân sự)
  - Việc CHỌN biến thể logo cụ thể cho từng vị trí (header/footer/favicon/intro): `CẦN XÁC MINH KHI IMPLEMENT`
- Fonts:
  - Heading/Display: **Nexa Heavy** (`Fonts/1FTV-Nexa-Heavy.otf`)
  - Body: **SVN-Gotham Regular** (`Fonts/SVN-Gotham Regular.otf`)
  - BẮT BUỘC font fallback: heading → `Arial, Helvetica, sans-serif`; body → `system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif` (fallback cuối cùng để duyệt khi triển khai)
  - Tối ưu tải font: chỉ nạp trọng số cần thiết, subset tiếng Việt, `font-display: swap`, ưu tiên woff2
- Website reference (Excel): https://www.talunglogistics.com/ — đối chiếu URL live: `CẦN XÁC MINH`
- Logo intro specification: xem mục 4.1

### 4.1 Logo Intro / Preloader Specification

Nguồn: video mẫu `docs/references/Screen Recording 2026-08-03 150942.mp4` (~3,6 giây), mô tả đã được chủ dự án xác nhận:

1. Toàn màn hình nền trắng.
2. Biểu tượng logo xuất hiện từng phần ở chính giữa.
3. Các nét được ghép/mở dần thành logo hoàn chỉnh.
4. Sau đó xuất hiện đường phân cách và wordmark bên phải.
5. Logo hoàn chỉnh giữ ngắn rồi chuyển vào nội dung website.

Đặc tả production (áp dụng cho logo Tà Lùng Quang Minh mới):

| ID | Yêu cầu | Ghi chú |
|---|---|---|
| REQ-ANIM-01 | Dựng bằng SVG/CSS/animation code | KHÔNG phát video làm preloader |
| REQ-ANIM-02 | Thời lượng mục tiêu 1–1,5 giây | Rút gọn từ 3,6s của bản mẫu |
| REQ-ANIM-03 | Chạy đầy đủ 1 lần trong mỗi session | Lần truy cập sau chỉ hiển thị tối thiểu hoặc bỏ qua |
| REQ-ANIM-04 | Không chặn website khi animation/asset lỗi | Phải có fallback render website ngay |
| REQ-ANIM-05 | Mobile có phiên bản rút gọn | Tránh tiêu tốn tài nguyên thiết bị yếu |
| REQ-ANIM-06 | Tôn trọng `prefers-reduced-motion` | Tắt hoặc bỏ animation khi người dùng yêu cầu |
| REQ-ANIM-07 | Chuyển mượt từ intro sang Hero | Không giật, không chặn scroll |
| REQ-ANIM-08 | Hiệu ứng button + logo load | Từ REQ-UX-16, chi tiết hiệu ứng button/hover/scroll ở mục 8.10 |

- Không cài `ffmpeg` chỉ để tạo tài liệu; trích frame video chỉ làm nếu cần kiểm chứng khi implementation (frame tạm không đưa vào Git).
- Model hiện tại không xem được ảnh → không tự mô tả hình học logo; dùng trực tiếp asset chính thức.

---

## 5. Current System Inventory

### 5.1 Quang Minh backend (`backend/`)

NestJS 11 + TypeORM 0.3.27 + PostgreSQL + passport-jwt + Cloudinary + Swagger. KHÔNG có global prefix. Response bọc bởi `TransformInterceptor` → `{statusCode, message, data}`.

**12 feature module (kiểm đếm trực tiếp từ `src/`):**

| # | Module | Nội dung | Controller chính |
|---|---|---|---|
| 1 | auth | login/register/profile, jwt+local strategy, @Roles decorator, JwtAuthGuard, RolesGuard | auth.controller |
| 2 | users | CRUD admin, reset password, UserRole enum (ADMIN/CONTENT_MANAGER/SALES/OPS), super admin id=1 đặc biệt | users.admin.controller |
| 3 | services | dịch vụ public + admin, service_translations | services.controller + admin |
| 4 | consignments | vận đơn + tracking events (lõi nghiệp vụ), AWB lookup public, ai-eta (giả lập) | consignments.controller + admin |
| 5 | quotes | yêu cầu báo giá public + admin, aiSuggestedPrice (giả lập) | quotes.controller + admin |
| 6 | news | tin tức public + admin, NewsStatus enum, news_translations | news.controller + admin |
| 7 | categories | danh mục phân cấp (self-hierarchy), CategoryType enum, category_translations | categories.controller + admin |
| 8 | careers | job postings + applications, upload CV Cloudinary (folder `cvs`) | careers.controller |
| 9 | search | tìm service + news (ILike trên translations) | search.controller |
| 10 | dashboard | `/stats/summary|timeseries|categorical|recent-activities` | dashboard.controller |
| 11 | upload | `POST /upload/image` → Cloudinary folder `site_assets` (max 5MB) | upload.controller |
| 12 | cloudinary | provider Cloudinary (config) | — |

Ngoài ra: `common/` (TransformInterceptor, pagination DTO, types) và `db/migrations/`.

**Known bugs / điểm bất thường (chi tiết mục 5.8) — đã re-verify khi khảo sát source 2026-08-14:**

- Route `/careers/postingss/` KHÔNG phải typo — route admin có chủ đích (`careers.controller.ts:91` `@Get('postingss/')` + Roles ADMIN/CONTENT_MANAGER), admin panel gọi đúng (`JobPostingsTab.tsx:23`). Không cần alias (B1 → verified, không phải bug)
- ĐÃ XÁC NHẬN (Critical): `GET /careers/postings/:id` public (không guard) trả kèm `relations: ['applications']` (`careers.service.ts:84`) → LỘ hồ sơ ứng viên (tên/email/phone/CV URL) qua public API (B14 MỚI)
- Dashboard `/stats/categorical?metric=service_category` → lỗi runtime (cột `category` đã bị DROP) — B2
- Dockerfile CMD `node dist/main` sai path (đúng: `dist/src/main`); không copy `data-source.ts` — B3
- Mismatch env: compose truyền `DB_*` nhưng app.module dùng `DATABASE_URL` — B4
- Migration chạy tự động khi boot trong `main.ts` (không `process.exit(1)` khi fail) — B5
- CORS: mở mọi origin khi `NODE_ENV !== 'production'` + fallback origins — B6
- `/auth/register` PUBLIC không guard, không rate limit (B13, TBD-22)
- Không có health endpoint, không có audit logging, không có throttling package (dự kiến Phase 5)

### 5.2 Quang Minh admin (`admin-panel-frontend/`)

Next.js **16.1.1** + React **18.3.1** + antd **5.11** + React Query 5 + zustand 4 (persist localStorage `auth-storage`) + TinyMCE 4 + @ant-design/charts.

- App Router: `(admin)` route group (dashboard, quotes, services, news, categories, careers, users, consignments) + `(auth)/login`
- Bảo vệ route CHỈ client-side (layout check token) — KHÔNG có middleware
- Token: localStorage + cookie không httpOnly; axios interceptor gắn Bearer, 401 → logout
- next-intl ^4.6.1 có trong deps nhưng KHÔNG dùng (UI hardcode tiếng Việt)
- pinyin-pro: sinh slug tiếng Trung
- Bug: page `/consignments` tồn tại nhưng KHÔNG có menu; route `postingss` trong JobPostingsTab là ĐÚNG (khớp backend)

### 5.3 Quang Minh public frontend (`frontend/`)

Next.js **15.5.9** + React **19.1** + next-intl 4.3 (vi/en/zh) + styled-components 6 + SWR + zustand 5 + framer-motion + swiper + react-hook-form/zod.

- App Router với segment động `[locale]`, localized pathnames (vi/en/zh), middleware next-intl
- API: axios (`NEXT_PUBLIC_API_URL`) trực tiếp, không proxy; SWR hooks; server-only fetchers cho SEO
- SEO đầy đủ: sitemap.ts, robots.ts, JSON-LD (Organization/Service/NewsArticle), hreflang, GTM `GTM-5BD4XCML`
- 31 namespaces messages (vi/en/zh)

### 5.4 Database — 12 tables (kiểm đếm trực tiếp các `@Entity`)

| # | Bảng | Module | Ghi chú quan trọng |
|---|---|---|---|
| 1 | users | users | email unique, password bcrypt (select:false), role enum, super admin id=1 |
| 2 | services | services | code unique, categoryId, featured |
| 3 | service_translations | services | locale+title+slug unique (serviceId, locale) |
| 4 | consignments | consignments | awb unique, status, aiPredictedEta, metadata jsonb |
| 5 | tracking_events | consignments | geo kiểu Postgres `point`, createdById → users, index (consignmentId, eventTime) |
| 6 | quotes | quotes | status default PENDING, aiSuggestedPrice, serviceId |
| 7 | news | news | status enum DRAFT/PUBLISHED, categoryId, featured, eager translations |
| 8 | news_translations | news | unique (newsId, locale) |
| 9 | categories | categories | type enum NEWS/SERVICE, parentId tự tham chiếu |
| 10 | category_translations | categories | unique (categoryId, locale) |
| 11 | job_postings | careers | status enum OPEN/CLOSED |
| 12 | job_applications | careers | cvPath (Cloudinary), appliedAt |

### 5.5 TypeORM migrations — 4 files

| # | File (trong `src/db/migrations/`) | Nội dung |
|---|---|---|
| 1 | `1760505175987-CreateInitialDatabase.ts` | Tạo toàn bộ bảng + enum ban đầu |
| 2 | `1760511822677-AddUserToTrackingEvent.ts` | createdBy → createdById + FK users |
| 3 | `1776567806531-AddCategoryAndSeo.ts` | categories, categoryId cho news/services, cột SEO, DROP cột category cũ |
| 4 | `1777086180013-MakeCategoryTranslatable.ts` | category_translations, DROP cột name/slug/desc cũ (class name khác timestamp tên file — vẫn chạy được) |

- Migration 3/4 có `down()` không hoàn chỉnh → không revert; chỉ tiến tới.
- Chạy: dev `npm run migration:run`, prod `npm run migration:run:prod` (node dist/data-source.js) — nhưng `data-source.ts` không nằm trong Docker image; trong container chỉ chạy được qua main.ts (auto-run — sẽ gỡ).

### 5.6 Routes & public URLs (frontend — 14 nhóm route + catch-all `[...rest]`)

| Route nhóm (source) | URL vi | URL en | URL zh | Trạng thái trong kế hoạch (mục 8.1, 12) |
|---|---|---|---|---|
| home | `/` | `/` | `/` | Viết lại (REQ-UX-05/06/07/09...) |
| about | `/gioi-thieu` | `/about-us` | `/guanyu` | Viết lại (REQ-UX-11) |
| services | `/dich-vu` | `/services` | `/fuwu` | Viết lại (REQ-UX-12, REQ-SVC-07) |
| services/[slug] | `/dich-vu/[slug]` | `/services/[slug]` | `/fuwu/[slug]` | Giữ cấu trúc + 6 trang con MỚI |
| news | `/tin-tuc` | `/news` | `/xinwen` | Viết lại (REQ-UX-13) |
| news/[slug] | `/tin-tuc/[slug]` | `/news/[slug]` | `/xinwen/[slug]` | Giữ + nhập 10 bài SEO |
| careers | `/tuyen-dung` | `/careers` | `/zhaopin` | Giữ + bổ sung nội dung, chuyển xuống footer (REQ-UX-10, REQ-CAREER) |
| careers/[id] | `/tuyen-dung/[id]` | `/careers/[id]` | `/zhaopin/[id]` | Giữ |
| contact | `/lien-he` | `/contact` | `/lianxi` | Viết lại (REQ-UX-14) |
| tracking | `/tra-cuu` | `/tracking` | `/chaxun` | Giữ (không nằm trong Excel) |
| manifesto | `/tuyen-ngon` | `/manifesto` | `/xuanyan` | Giữ, chuyển xuống footer (REQ-UX-10) |
| terms | `/dieu-khoan` | `/terms` | `/tiaokuan` | CANONICAL trang pháp lý gộp: giữ URL, hiển thị nội dung Điều khoản + Bảo mật (REQ-UX-19, TBD-19 Resolved) |
| privacy | `/chinh-sach-bao-mat` | `/privacy` | `/yinsi` | Redirect 301 → `/dieu-khoan` (REQ-UX-19) |
| infrastructure | `/nang-luc-ha-tang` | `/infrastructure-capacity` | `/ji-chu-she-shi` | Viết lại, tách trang riêng, thêm vào nav (Excel Web sheet) |
| [...rest] | catch-all | — | — | 404 |
| not-found | 404 | — | — | Giữ |

- Redirect hiện có: `/quote` → `/contact` (permanent, trong next.config).
- `infrastructure` hiện KHÔNG có trong `navigation.ts` pathnames và KHÔNG có trong sitemap.ts → phải thêm.
- Lưu ý: search là modal trong Header (không phải trang riêng) → xóa/ẩn modal (REQ-UX-08).

### 5.7 Media / upload storage

- Cloudinary: folder `site_assets` (ảnh nội dung, max 5MB, jpg/jpeg/png/gif/webp/avif) và `cvs` (CV pdf/doc/docx, max 5MB).
- `uploads/` local được serve tại `/uploads` (ServeStaticModule) — hiện CV đi Cloudinary nên gần như rỗng.
- Swagger UI tại `/api-docs` (title hiện ghi "Phú Anh Smart Border API" — cần đổi brand).

### 5.8 Production infrastructure

- Backend production: **cPanel** (đã xác nhận). Database production: host trong hạ tầng cPanel (đã xác nhận). Engine và phiên bản database production CHƯA được kiểm chứng trực tiếp — repo/docker-compose dùng `postgres:15-alpine` và backend dùng TypeORM pg driver; phải xác nhận bằng khảo sát schema/version trong Phase 1.
- Quy trình hiện tại: build local → ZIP → upload cPanel.
- `TBD`: có SSH hay không; có cPanel Git Version Control hay không; Node Application Manager/Passenger/PM2 loại nào; document root và thư mục release; cách restart; khả năng cron/worker/Redis. (không hỏi lại loại hạ tầng — đã xác định cPanel)

### 5.8b Known bugs (tổng hợp)

| # | Bug | Vị trí | Mức độ | Hướng xử lý |
|---|---|---|---|---|
| B1 | Route `postingss` — KHÔNG phải typo (verified 2026-08-14) | backend careers.controller.ts:91 + admin JobPostingsTab.tsx:23 | Không (đã loại) | Giữ nguyên: route admin có chủ đích `GET postingss/` (Roles ADMIN/CONTENT_MANAGER), admin gọi đúng, public dùng `GET postings` (REQ-BACKEND-03: bỏ điều kiện re-verify, giữ ổn định alias) |
| B2 | `/stats/categorical?metric=service_category` lỗi runtime | dashboard.service | Cao | Fix query (REQ-BACKEND-04) |
| B3 | Dockerfile CMD sai path + thiếu data-source.ts | Dockerfile | Cao | Fix (phase 2) |
| B4 | Env mismatch `DB_*` vs `DATABASE_URL` | docker-compose + app.module | Cao | Fix (phase 2) |
| B5 | Auto-migrate khi boot (main.ts, không exit khi fail) | main.ts | Cao | Gỡ, chuyển thủ công (REQ-BACKEND-05) |
| B6 | CORS mở mọi origin khi NODE_ENV≠production | main.ts | Trung bình | Siết production |
| B7 | `@BeforeUpdate` hash rỗng | user.entity | Trung bình | Sửa (REQ-BACKEND-14) |
| B8 | Password tạm console.log | users.service | Trung bình | Email/TBD (REQ-BACKEND-15) |
| B9 | Migration 3/4 down() không hoàn chỉnh | migrations | Thấp | Không revert |
| B10 | Admin chỉ bảo vệ client-side, token localStorage | admin-panel | Cao | BFF proxy + httpOnly (REQ-ADMIN-01/02) |
| B11 | `/consignments` không có menu admin | admin sidebar | Thấp | Thêm menu |
| B12 | Swagger title sai brand | main.ts | Thấp | Đổi tên |
| B13 | `/auth/register` public — không guard, không rate limit (verified 2026-08-14) | auth.controller.ts:14-20 | Cao | Re-verify production trong Phase 1B (TBD-22). Khóa admin-only/setup-only nếu production cũng public; KHÔNG cho user tự chọn role đặc quyền |
| B14 (MỚI) | Lộ hồ sơ ứng viên: `GET /careers/postings/:id` public trả `relations: ['applications']` (tên/email/phone/cvPath) | careers.service.ts:84 + careers.controller.ts:54-57 | Critical | Sửa ngay ở phase 1 fix: bỏ relations applications khỏi query public (chỉ dùng ở admin detail). Ghi audit. KHÔNG đổi public API contract |

---

## 6. Greentech Reference Mapping

**Ghi chú khảo sát 2026-08-14:** naming thực tế trong Greentech KHÁC giả định cũ — admin dùng `@Controller('admin/...')` trong `greentech/…/<module>.controller.ts` (vd `news.controller.ts:31-32`), public dùng `<module>.public.controller.ts` (vd `news.public.controller.ts:7`). KHÔNG có file `*.admin.controller.ts` riêng. Greentech dùng Prisma (error codes P2025→404, P2002→409); Quang Minh sẽ map TypeORM (23505/22P02).

| Greentech pattern | Áp dụng cho Quang Minh? | Cách triển khai | Status |
|---|---|---|---|
| Tách `*.controller.ts` (admin) / `*.public.controller.ts` | ✅ CÓ | Viết lại theo module, TypeORM style. Lưu ý: naming thực tế là `@Controller('admin/...')` trong controller chính + public controller riêng (`<module>.public.controller.ts`) | Planned — phase 5 |
| AuditLogsService @Global + log mọi thay đổi | ✅ CÓ | Reference: Greentech `audit-logs.module.ts` (@Global, dòng 7-12), model `AuditLog @@map("audit_logs")` (schema.prisma:47-61), `audit-logs.service.ts:13-47`. Service @Global + bảng audit_logs (dự kiến), TypeORM | Proposed |
| AllExceptionsFilter chuẩn hóa lỗi + map Prisma→HTTP | ✅ CÓ (viết lại) | Reference: Greentech `common/filters/http-exception.filter.ts:11-67` (map P2025→404, P2002→409, 429, validation→VALIDATION_ERROR; output `{success,statusCode,errorCode,message,path,timestamp}`) + TransformInterceptor `common/interceptors/transform.interceptor.ts:27-46`. Viết lại map TypeORM (23505/22P02) | Planned — phase 5 |
| Soft delete `deleted_at` | ✅ CÓ (cân nhắc) | Thêm cột nullable + filter; additive migration | Proposed — TBD |
| i18n JSONB `*_i18n` | ❌ KHÔNG | Giữ bảng translation tách rời | Rejected (Approved Decision) |
| Prisma ORM | ❌ KHÔNG | Giữ TypeORM | Rejected |
| BFF proxy + cookie httpOnly (middleware Next) | ✅ CÓ (admin) | Route Handler + middleware + cookie httpOnly | Planned — phase 4 |
| RHF wrapper components + zod `z.infer` | ✅ CÓ (admin) | Viết lại form pattern | Planned — phase 3 |
| 1 Form dùng chung Create/Edit (page create/ và [id]/) | ✅ CÓ | Áp dụng cho services/news/careers/job-postings | Planned — phase 3 |
| `statusConfig: Record<Status,{color,label}>` | ✅ CÓ | Tập trung map màu/label (Greentech dùng config nội bộ module, không global) | Planned — phase 3 |
| Optimistic update | ⚠️ GIỚI HẠN | Chỉ toggle hiển thị + thứ tự slider | Constrained (Approved Decision) |
| Throttler + chống spam form công khai | ✅ CÓ | @nestjs/throttler + check trùng (hiện QM KHÔNG có @nestjs/throttler trong deps) | Planned — phase 5 |
| CSV export + BOM UTF-8 | ✅ CÓ (có giới hạn) | json2csv cho quotes/applications — chỉ role được phép; không export raw CV URL/signed URL; chỉ field nghiệp vụ; ghi audit event khi export; escape formula injection (`=`, `+`, `-`, `@`); retention file export `TBD` (tạo theo yêu cầu, không lưu lâu dài). Reference: Greentech CSV export + audit EXPORT_CSV | Proposed |
| Health check (Terminus) | ✅ CÓ | @nestjs/terminus (QM backend hiện KHÔNG có health endpoint) | Planned — phase 5 |
| Theme tokens tập trung | ✅ CÓ | themeConfig + brand tokens (Greentech: `themeConfig.ts` colorPrimary) | Planned — phase 3 |
| Upload 3 tầng fallback (Cloudinary/R2/local) | ⚠️ CÓ THỂ | Giữ Cloudinary hiện tại; fallback `TBD` | TBD |
| BullMQ queue (lead/CV/indexing) | ❌ TBD | Chỉ khi có Redis trên cPanel | TBD (mục 16) |
| search_logs | ❌ KHÔNG | Không có use case được duyệt | Rejected |
| Register tự khóa user đầu tiên | ❌ KHÔNG | Giữ register hiện tại (xem xét khóa) | TBD |

---

## 7. Requirements Traceability

Mã ID: `REQ-UX-*`, `REQ-W*`, `REQ-SVC-*`, `REQ-SEO-*`, `REQ-CAREER-*`, `REQ-BRAND-*`, `REQ-ANIM-*`, `REQ-BACKEND-*`, `REQ-ADMIN-*`, `REQ-DATA-*`, `REQ-DEPLOY-*`.

Cột: `ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status`

### 7.1 UI/UX — 17 hạng mục từ sheet "Fix Giao diện UI & UX" (Excel có 20 dòng, Stt 1–17, một số Stt lặp)

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-UX-01 | Excel Stt 1 + Stt 17 | Header/Toàn site | Màu chủ đạo đổi sang hệ brand mới | Không | Không còn màu đỏ/#0b9444; token #233871/#0c9344 đồng nhất | Draft |
| REQ-UX-02 | Excel Stt 2 | Header | Logo ngang: hình trái + chữ "Tà Lùng Quang Minh Logistics" phải | Không | Header hiển thị logo ngang đúng asset 2026 | Draft |
| REQ-UX-03 | Excel Stt 3 | Header + Liên hệ + Footer | SĐT +84 865.865.600 | Không | SĐT đúng mọi vị trí — đã chủ dự án xác nhận (TBD-13 Resolved) | Draft |
| REQ-UX-04 | Excel Stt 4 | Tin tức | Bỏ mục "Khác" (tab/khối dư thừa) | Không | Không còn mục "Khác" | Draft |
| REQ-UX-05 | Excel Stt 4 | Trang chủ Hero | Hero slider: tự lướt/đếm ngược, click dẫn trang, quản trị từ admin | Bảng `sliders` (dự kiến) + API admin/public | Slider quản trị được; click đúng link; lướt mượt | Draft |
| REQ-UX-06 | Excel Stt 5 | Trang chủ | Tách thông tin công ty khỏi Hero; viết lại: slogan "Kết nối biên giới - Vươn tới toàn cầu", title "Đại lý Hải Quan - Vận Tải - Sang Tải Lưu Đỗ - Kho Bãi", subtitle từ 2008; ảnh dịch vụ | Không | Nội dung mới đúng bản duyệt; `CẦN XÁC MINH` số liệu | Draft |
| REQ-UX-07 | Excel Stt 5 | Trang chủ | Block số liệu chỉ số (Top 10, Đầu tiên...) | Không | Số liệu hiển thị — MỌI SỐ LIỆU `CẦN XÁC MINH` trước publish | Draft |
| REQ-UX-08 | Excel Stt 5 | Header nav | Bỏ hoặc tạm ẩn tính năng tìm kiếm | Không (không thêm search_logs) | Không còn search modal trên nav | Draft |
| REQ-UX-09 | Excel Stt 6 | Header nav | Đổi nội dung CTA nav ("Kết nối ngay / Liên hệ / Tư vấn ngay / Hỗ trợ ngay") | Không | CTA mới hiển thị | Draft |
| REQ-UX-10 | Excel Stt 7 | Header nav + Footer | Chuyển Tuyển dụng + Tuyên ngôn xuống Footer | Không | Nav chính không còn 2 mục này; footer có link | Draft |
| REQ-UX-11 | Excel Stt 8 | Trang chủ Dịch vụ | Layout mới: list 6 dịch vụ + bảng giá + nhiều ảnh chi tiết + khoảng cách đồng nhất | Excel xác định TARGET gồm 6 nhóm dịch vụ; source hỗ trợ services/translations; số bản ghi và nội dung dịch vụ thực tế trong production CHƯA được xác minh (chưa đọc DB production) — cần schema/row counts + báo cáo từ admin (Phase 1); bảng giá `TBD` (field?) | Layout theo 2 phương án đã đề xuất; tham chiếu prnt.sc | Draft |
| REQ-UX-12 | Excel Stt 8 | Trang chủ Quy trình 6 bước | Gạch đầu dòng từng bước + hover effect/ảnh | Không | Hover hiệu ứng, nội dung đúng quy trình thật (`CẦN XÁC MINH`) | Draft |
| REQ-UX-13 | Excel Stt 9 | Trang chủ Bài viết & Sự kiện | Thêm tab Sự kiện/Hoạt động; layout mới | Có thể cần field/loại bài; `TBD` | Tab sự kiện hoạt động; layout theo tham chiếu | Draft |
| REQ-UX-14 | Excel Stt 10 | Trang chủ | Block chứng chỉ + lộ trình/hệ sinh thái công ty | Bảng `certificates` (dự kiến) | Hiển thị chứng chỉ (`CẦN XÁC MINH` danh sách thật) | Draft |
| REQ-UX-15 | Excel Stt 11 | Giới thiệu | 9 nội dung: ảnh bìa, số liệu, lịch sử (mốc + năm), tầm nhìn/sứ mệnh/giá trị, cơ cấu tổ chức, chiến lược, chứng nhận/thành tựu, hiệp hội, đối tác | Bảng `partners` (dự kiến) + certificates | Đủ 9 block; nội dung duyệt bởi công ty | Draft |
| REQ-UX-16 | Excel Stt 12+13 | Dịch vụ + Tin tức | Ảnh bìa cho cả 2 trang; layout dịch vụ mới; BỎ "Năng lực cung ứng dịch vụ tại Tà Lùng Logistics"; BỎ "Cập nhật thông tin Logistics biên mậu Việt - Trung" | Không | 2 trang có ảnh bìa; khối cũ đã gỡ | Draft |
| REQ-UX-17 | Excel Stt 14 | Liên hệ | SĐT đúng; form đính kèm ảnh/file; layout mới; map văn phòng Hạ Long | Upload attachment (REQ-BACKEND-06) + map | Form gửi kèm file thành công; map Hạ Long hiển thị | Draft |

(Ghi chú: dòng "Màu sắc đồng bộ xanh" trong Excel bị thay thế bởi REQ-BRAND-02; hiệu ứng button/logo = REQ-ANIM-08; footer = REQ-UX-18 bên dưới.)

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-UX-18 | Excel Stt 15 | Footer | Logo đầy đủ; SƠ ĐỒ TRANG > DỊCH VỤ liệt kê đủ 6 dịch vụ; THÔNG TIN PHÁP LÝ > VỀ CÔNG TY (Giới thiệu, Phát triển bền vững, Tin tức, Tuyển dụng, Tuyên ngôn, Thông tin pháp lý gộp); form: Title "Liên hệ với chúng tôi", CTA "Yêu cầu hỗ trợ" → "Gửi yêu cầu" | Trang pháp lý gộp + redirect 301 (REQ-UX-19) | Footer đúng cấu trúc mới | Draft |

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-UX-19 | Excel Stt 15 (gộp pháp lý) + quyết định 2026-08-14 | Legal page | Gộp Điều khoản + Bảo mật thành 1 trang "Thông tin pháp lý" | Canonical `/dieu-khoan` (đã chốt — TBD-19 Resolved): `/dieu-khoan` GIỮ URL và trả nội dung gộp; redirect 301 DUY NHẤT `/chinh-sach-bao-mat` → `/dieu-khoan`; KHÔNG tạo `/thong-tin-phap-ly`; KHÔNG redirect `/dieu-khoan` sang URL khác; không redirect loop/chain. Khi implementation đa ngôn ngữ: giữ nguyên tắc canonical tương ứng cho từng locale — mapping cụ thể phải kiểm tra với route hiện hữu trước khi triển khai | Trang gộp tồn tại tại `/dieu-khoan`; 301 đúng; không 404; metadata/canonical/sitemap/internal link trỏ `/dieu-khoan` | Draft (canonical đã chốt) |

**Tổng REQ-UX: 19**

### 7.2 Checklist triển khai — W01–W20 (sheet "Checklist")

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-W01 | W01 | Trang chủ | Viết lại hero: headline định vị logistics cửa khẩu Tà Lùng, thông quan, kho bãi, vận tải Việt-Trung | Không | Content đúng bản duyệt | Draft |
| REQ-W02 | W02 | Trang chủ | Block khách hàng mục tiêu (chủ hàng, forwarder, công ty thương mại, doanh nghiệp vận tải) | Không | Block hiển thị | Draft |
| REQ-W03 | W03 | Trang chủ | Block quy trình đơn hàng: tiếp nhận → kiểm tra chứng từ → điều phối xe/bãi → khai báo → sang tải → bàn giao | Không | Đúng 6 bước, hover effect | Draft |
| REQ-W04 | W04 | Dịch vụ | Nội dung tổng quan 6 nhóm dịch vụ + CTA báo giá | Không | Đủ 6 nhóm + CTA | Draft |
| REQ-W05 | W05 | Đại lý hải quan | Trang con: vấn đề, giải pháp, quy trình, lợi ích, CTA | CMS dịch vụ có sẵn; field SEO | Trang live đúng template | Draft |
| REQ-W06 | W06 | Kho bãi & bãi tập kết | Mô tả kho/bãi, quy trình, năng lực, hình ảnh, CTA | Như trên | Trang live | Draft |
| REQ-W07 | W07 | Sang tải hàng hóa | Sang tải cửa khẩu: thiết bị, quy trình, lưu ý | Như trên | Trang live | Draft |
| REQ-W08 | W08 | Vận tải quốc tế | Tuyến vận tải, phương tiện, quy trình nhận hàng, tracking | Như trên | Trang live | Draft |
| REQ-W09 | W09 | Bến xe & điều phối | Hỗ trợ nhà xe, quy định ra/vào bãi, điều phối | Như trên | Trang live | Draft |
| REQ-W10 | W10 | Logistics trọn gói | Một đầu mối: chứng từ, bãi, vận tải, sang tải, bàn giao | Như trên | Trang live | Draft |
| REQ-W11 | W11 | Liên hệ | Form báo giá: loại hàng, tuyến, số lượng, thời gian, thông tin liên hệ, ghi chú | Quotes hiện tại; thêm field `TBD`; attachment | Form gửi đủ field | Draft |
| REQ-W12 | W12 | Toàn site | CTA cố định mobile: gọi hotline, Zalo, Yêu cầu báo giá | Không | CTA mobile hoạt động | Draft |
| REQ-W13 | W13 | Tuyển dụng | Văn hóa, phúc lợi, vị trí, quy trình, form ứng tuyển | Careers hiện có | Trang đầy đủ theo REQ-CAREER | Draft |
| REQ-W14 | W14 | Tuyển dụng | JD vị trí khai báo hải quan | job_postings fields | JD đúng chuẩn | Draft |
| REQ-W15 | W15 | Tuyển dụng | Form nộp CV + email nhận thông báo | Upload CV + email `TBD` | Form + thông báo hoạt động | Draft |
| REQ-W16 | W16 | Tin tức | 10 bài SEO đầu tiên theo sheet SEO (REQ-SEO-01..10) | news + translations | 10 bài nhập CMS, chưa auto-publish | Draft |
| REQ-W17 | W17 | Năng lực hạ tầng | Kho bãi, bến xe, thiết bị, sơ đồ, ảnh/video, số liệu | Sliders/partners/media; số liệu `CẦN XÁC MINH` | Trang riêng, có nav | Draft |
| REQ-W18 | W18 | Toàn site | Ảnh thực tế kho bãi, xe, đội ngũ, cửa khẩu | Media upload | Ảnh thật thay thế ảnh minh họa | Draft |
| REQ-W19 | W19 | Tiếng Trung | Bản tiếng Trung các trang chính | translation hiện có (zh) | Trang zh đầy đủ | Draft |
| REQ-W20 | W20 | Toàn site | SEO kỹ thuật: title, meta, H1-H2, internal link, FAQ schema, tốc độ mobile | metadata fields có sẵn | Audit SEO pass | Draft |

**Tổng REQ-W: 20**

### 7.3 Dịch vụ — 6 nhóm + trang tổng (sheet "Dịch vụ" + "Web")

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-SVC-01 | Dịch vụ #1 | Đại lý hải quan | Vấn đề → giải pháp → quy trình → lợi ích → FAQ → CTA "Gửi bộ chứng từ để kiểm tra trước" | Dịch vụ + translation vi/en/zh; SEO fields | Trang live; CTA đúng; keyword đúng | Draft |
| REQ-SVC-02 | Dịch vụ #2 | Kho bãi & bãi tập kết | Diện tích, ảnh bãi, quy trình nhập/xuất, bảo vệ, giám sát; CTA "Tư vấn phương án lưu kho/lưu bãi" | Như trên | Trang live | Draft |
| REQ-SVC-03 | Dịch vụ #3 | Sang tải hàng hóa | Quy trình, thiết bị, nhân sự, thời gian, loại hàng; CTA "Nhận tư vấn sang tải" | Như trên | Trang live | Draft |
| REQ-SVC-04 | Dịch vụ #4 | Vận tải hàng hóa quốc tế | Tuyến, phương tiện, quy trình nhận hàng, tracking; CTA "Nhận báo giá vận tải" | Như trên + tracking link | Trang live | Draft |
| REQ-SVC-05 | Dịch vụ #5 | Bến xe & điều phối | Quy trình xe vào/ra, bãi đỗ, hỗ trợ lái xe, an toàn; CTA "Liên hệ điều phối xe" | Như trên | Trang live | Draft |
| REQ-SVC-06 | Dịch vụ #6 | Logistics trọn gói | Một đầu mối, báo cáo tiến độ, xử lý phát sinh; CTA "Yêu cầu giải pháp trọn gói" | Như trên | Trang live | Draft |
| REQ-SVC-07 | Web sheet — Dịch vụ | Trang Dịch vụ tổng | 6 nhóm rõ ràng + mỗi nhóm CTA báo giá | categories type SERVICE | Trang tổng có 6 nhóm | Draft |

**Tổng REQ-SVC: 7**

### 7.4 SEO — 10 bài (sheet "SEO tin tức")

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-SEO-01 | SEO #1 | Tin tức | Bài "Doanh nghiệp cần chuẩn bị gì khi thông quan qua cửa khẩu Tà Lùng?" | news + translation | Bài viết tồn tại; keyword "thông quan cửa khẩu Tà Lùng"; CTA cuối bài | Draft |
| REQ-SEO-02 | SEO #2 | Tin tức | "Sang tải hàng hóa tại cửa khẩu là gì?" (link sang trang Sang tải) | news + link | Bài + internal link | Draft |
| REQ-SEO-03 | SEO #3 | Tin tức | "Khi nào cần thuê kho bãi gần cửa khẩu?" (ảnh bãi thực tế) | news | Bài + ảnh | Draft |
| REQ-SEO-04 | SEO #4 | Tin tức | "Lưu ý khi vận chuyển qua tuyến Việt Nam - Trung Quốc" (link trang vận tải) | news | Bài + internal link | Draft |
| REQ-SEO-05 | SEO #5 | Tin tức | "Mã HS, thuế nhập khẩu và hồ sơ hải quan" | news | Bài chuyên môn | Draft |
| REQ-SEO-06 | SEO #6 | Tin tức | "Xu hướng logistics biên giới Việt - Trung" | news | Bài thị trường | Draft |
| REQ-SEO-07 | SEO #7 | Tin tức | "Định hướng logistics xanh tại khu vực cửa khẩu" | news | Bài ESG | Draft |
| REQ-SEO-08 | SEO #8 | Tin tức | "Checklist chứng từ trước khi đưa hàng ra cửa khẩu" (lead magnet) | news | Checklist + CTA | Draft |
| REQ-SEO-09 | SEO #9 | Tin tức | "Lợi ích của bãi tập kết gần cửa khẩu" (link trang hạ tầng) | news | Bài + link | Draft |
| REQ-SEO-10 | SEO #10 | Tin tức | "5 FAQ làm logistics qua cửa khẩu Tà Lùng" (FAQ schema) | news | Bài FAQ + schema | Draft |

**Tổng REQ-SEO: 10**

### 7.5 Tuyển dụng (sheet "Tuyển dụng")

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-CAREER-01 | TD #1 | Tuyển dụng Hero | Headline + subheadline + nút ứng tuyển; ảnh đội ngũ/kho bãi thật | Không | Hero hiển thị | Draft |
| REQ-CAREER-02 | TD #2 | Tuyển dụng | Block "Vì sao chọn chúng tôi" | Không | Block hiển thị | Draft |
| REQ-CAREER-03 | TD #3 | Tuyển dụng | Danh sách vị trí + JD (vị trí, số lượng, địa điểm, mô tả, yêu cầu, lương/phúc lợi) | job_postings fields | JD đầy đủ | Draft |
| REQ-CAREER-04 | TD #4 | Tuyển dụng | Block Phúc lợi | Không | Hiển thị | Draft |
| REQ-CAREER-05 | TD #5 | Tuyển dụng | Quy trình ứng tuyển 4 bước (Nộp CV > Sàng lọc > Phỏng vấn > Nhận việc) | Không | 4 bước + icon | Draft |
| REQ-CAREER-06 | TD #6 | Tuyển dụng | Form ứng tuyển: họ tên, SĐT, email, vị trí, kinh nghiệm, tải CV, ghi chú | /careers/postings/:id/apply hiện có | Form submit + CV lên Cloudinary | Draft |

**Tổng REQ-CAREER: 6**

### 7.6 Brand (quyết định chủ dự án 2026-08-14)

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-BRAND-01 | Chủ dự án | Toàn site | Brand "Tà Lùng Quang Minh Logistics" | — | Đúng tên mọi vị trí | Approved |
| REQ-BRAND-02 | Chủ dự án | Toàn site | Primary #233871, Secondary #0c9344; không dùng #0b9444 | — | Design tokens đúng | Approved |
| REQ-BRAND-03 | Chủ dự án | Toàn site | Không Dark Mode | — | Không có toggle/theme tối | Approved |
| REQ-BRAND-04 | Chủ dự án | Toàn site | Logo 2026 từ docs/references/Logo/ là asset chính thức | — | Dùng đúng asset; biến thể `CẦN XÁC MINH KHI IMPLEMENT` | Approved |
| REQ-BRAND-05 | Chủ dự án | Toàn site | Font: Nexa Heavy (heading), SVN-Gotham (body) + fallback + tối ưu tải | — | Font đúng, fallback hoạt động | Approved |

**Tổng REQ-BRAND: 5**

### 7.7 Animation (mô tả video + đặc tả chủ dự án)

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-ANIM-01..08 | Video + chủ dự án | Logo intro + buttons | Xem bảng 4.1 | — | Theo từng tiêu chí ở 4.1 | Approved |

**Tổng REQ-ANIM: 8**

### 7.8 Backend

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-BACKEND-01 | Quyết định | Backend | — | Giữ TypeORM + PostgreSQL + translation tách bảng | Không có Prisma/JSONB | Approved |
| REQ-BACKEND-02 | Quyết định | Backend | — | API tương thích client hiện tại | Không breaking change khi chưa duyệt | Approved |
| REQ-BACKEND-03 | Rev.2 + verified 2026-08-14 | Backend careers | — | ĐÃ RE-VERIFY: `/careers/postingss/` là route admin có chủ đích (`@Get('postingss/')` careers.controller.ts:91 + Roles ADMIN/CONTENT_MANAGER), admin gọi đúng `JobPostingsTab.tsx:23`, public dùng `/careers/postings`. KHÔNG cần alias. CHÚ Ý: `GET /careers/postings/:id` public trả kèm applications (B14) — tách riêng | Giữ nguyên contract hiện tại | Approved (verified) |
| REQ-BACKEND-04 | Bug B2 | Dashboard | Admin dashboard | Fix query `service_category` (dùng categoryId) | /stats/categorical không lỗi | Planned |
| REQ-BACKEND-05 | Quyết định | main.ts | — | Gỡ auto-migrate khi boot | Boot không chạy migration | Approved |
| REQ-BACKEND-06 | Excel UX-17/W11 + chỉ thị 2026-08-14 | Upload | Form đính kèm | CV và attachments (quotes/contact) là dữ liệu RIÊNG TƯ: chỉ admin có role phù hợp được xem/tải; dùng private/authenticated asset hoặc signed URL có thời hạn; KHÔNG hiển thị raw public URL trong API public. Đặc tả: loại file (ảnh + pdf/doc, `TBD` danh sách cuối), max kích thước (`TBD`, đề xuất 5MB), max số file (`TBD` 1–5), validate extension + MIME + magic bytes (magic bytes chỉ là xác thực định dạng, KHÔNG gọi là malware scanning; malware scanning thật = `TBD`, phụ thuộc hạ tầng/dịch vụ quét), tên file an toàn (sanitize + random), nơi lưu (Cloudinary — folder `attachments`), retention policy + cơ chế xóa, quyền truy cập (admin-only), metadata: owner/module, storage key, original filename đã sanitize, MIME, size, checksum, createdAt; audit log khi admin tải/xóa file nhạy cảm; CV được bảo vệ tương tự attachments | Upload thành công + an toàn; private access đúng; audit đủ | Draft |
| REQ-BACKEND-07 | Rev.2 | Backend public | — | Rate limit cho /quotes, /careers/apply | 429 khi vượt | Planned |
| REQ-BACKEND-08 | Greentech pattern + chỉ thị 2026-08-14 | Backend | Admin UI logs | AuditLogService @Global + bảng audit_logs (dự kiến). QUY TẮC BẢO MẬT LOG: không log toàn bộ request body một cách máy móc; KHÔNG BAO GIỜ ghi password, password hash, token, cookie, secret, authorization header; redact/không lưu CV URL, signed URL, storage credential; với quotes/applications/users dùng ALLOWLIST field được phép audit thay vì lưu toàn bộ old_data/new_data; PII mask hoặc chỉ ghi trường nào đã thay đổi, không ghi nguyên giá trị nếu không cần; audit log chỉ role được phép mới xem; audit log KHÔNG sửa/xóa qua API thông thường; retention policy `TBD`; mọi export audit log phải áp dụng RBAC + redaction tương tự | Ghi log CRUD admin; log không chứa secret/PII thô | Proposed |
| REQ-BACKEND-09 | Greentech pattern | Backend | — | Health check /api/health (DB + uptime) | Endpoint trả status | Planned |
| REQ-BACKEND-10 | Greentech pattern | Backend | — | AllExceptionsFilter chuẩn hóa lỗi | Lỗi format nhất quán | Planned |
| REQ-BACKEND-11 | Chỉ thị 2026-08-14 | Backend | — | Chống spam: rate limit theo IP/session; có thể kết hợp email đã normalize + payload fingerprint; idempotency/deduplication CHỈ áp dụng với payload thực sự giống nhau trong cửa sổ ngắn (KHÔNG chặn máy móc mọi lần gửi trùng email — tránh chặn yêu cầu hợp lệ); KHÔNG ghi email thô vào log chống spam; ngưỡng cụ thể `TBD` sau khi đánh giá traffic | Rate limit hoạt động; không chặn yêu cầu hợp lệ; log không chứa email thô | Planned |
| REQ-BACKEND-12 | Bug B6 | main.ts | — | Siết CORS production (whitelist FRONTEND_URL) | Origin lạ bị chặn | Planned |
| REQ-BACKEND-13 | Hiện trạng | users | — | Xem xét super admin id=1 đặc quyền (giữ tạm, đánh dấu `TBD`) | Giữ hành vi hiện tại tới khi duyệt | TBD |
| REQ-BACKEND-14 | Bug B7 | user.entity | — | Sửa @BeforeUpdate hash password | Không lưu plaintext | Planned |
| REQ-BACKEND-15 | Bug B8 | users | Admin hiển thị 1 lần | Bỏ console.log password tạm; gửi email `TBD` | Không log secret | Planned |

**Tổng REQ-BACKEND: 15**

### 7.9 Admin

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-ADMIN-01 | Greentech pattern + chỉ thị 2026-08-14 | Admin auth | Login qua BFF Route Handler; cookie httpOnly + secure (production) + sameSite phù hợp, 7 ngày. CSRF: mọi request thay đổi dữ liệu phải kiểm tra Origin/Referer theo allowlist; nếu kiến trúc yêu cầu, dùng CSRF token cho POST/PATCH/PUT/DELETE; logout phải xóa cookie server-side; KHÔNG chuyển tiếp tùy ý header/cookie đến host ngoài backend allowlist; BFF proxy chỉ cho phép route backend được định nghĩa (tránh open proxy); access-token expiry/refresh hoặc buộc đăng nhập lại = `TBD` | /auth/login + /auth/profile qua proxy | Token không lộ client JS; CSRF bị chặn; cookie xóa khi logout | Planned |
| REQ-ADMIN-02 | Greentech pattern + chỉ thị 2026-08-14 | Admin | Middleware bảo vệ route server-side; chỉ forward tới backend trong allowlist | — | Không vào trang khi thiếu cookie; không open proxy | Planned |
| REQ-ADMIN-03 | Hiện trạng | Dashboard | KPI cards + charts + hoạt động gần đây (giữ) | /stats/* | Dashboard không lỗi | Draft |
| REQ-ADMIN-04 | REQ-UX-05 | Sliders | Màn hình CRUD slider (ảnh, title, link, thứ tự, active) | Bảng sliders (dự kiến) + API | CRUD đầy đủ + optimistic thứ tự | Draft |
| REQ-ADMIN-05 | Hiện trạng | Dịch vụ | Form dịch vụ + translation vi/en/zh (RHF+zod), SEO block, TinyMCE | /admin/services | CRUD + translation đúng | Draft |
| REQ-ADMIN-06 | Hiện trạng | Tin tức | Form tin tức + SEO + status + featured | /admin/news | CRUD đúng | Draft |
| REQ-ADMIN-07 | Hiện trạng | Danh mục | CRUD danh mục + translation | /admin/categories | CRUD đúng | Draft |
| REQ-ADMIN-08 | REQ-CAREER + chỉ thị 2026-08-14 | Tuyển dụng | Tabs: JD (CRUD) + Hồ sơ (list, xem CV, trạng thái) | /careers/postings, /careers/applications | 2 tab hoạt động; xem/tải CV qua signed URL (private access, chỉ role phù hợp), audit khi tải/xóa; CSV export: chỉ role được phép, KHÔNG xuất raw CV URL/signed URL, chỉ field nghiệp vụ cần thiết, ghi audit event khi export, escape formula injection (`=`, `+`, `-`, `@`), retention file export `TBD` (tạo theo yêu cầu, không lưu lâu dài) | Draft |
| REQ-ADMIN-09 | REQ-BACKEND-06 + chỉ thị 2026-08-14 | Quotes | List + chi tiết + file đính kèm + adminNotes + status | /quotes + attachments | Xem/sửa/delete + tải file đính kèm qua signed URL (chỉ role phù hợp, audit log); CSV export quotes an toàn giống REQ-ADMIN-08 (role được phép, không raw URL, audit, escape formula injection, retention `TBD`) | Draft |
| REQ-ADMIN-10 | REQ-UX-15 | Partners | CRUD đối tác (logo, website, order, active) | Bảng partners (dự kiến) | CRUD đúng | Draft |
| REQ-ADMIN-11 | REQ-UX-14 | Certificates | CRUD chứng chỉ (tên, ảnh, tổ chức cấp, năm) | Bảng certificates (dự kiến) | CRUD đúng | Draft |
| REQ-ADMIN-12 | Greentech pattern + chỉ thị 2026-08-14 | Settings | Global settings: KHÔNG public toàn bộ bảng. Public API chỉ trả ALLOWLIST key đã duyệt (hotline công ty, email công ty, địa chỉ, Zalo, map); key nội bộ/cấu hình tích hợp/credential/secret TUYỆT ĐỐI không trả qua public endpoint; secret KHÔNG lưu trong global_settings (quản lý qua env/secret store); admin update validate type/schema theo từng key + ghi audit log đã redact; namespace/phân loại `public`/`internal` hoặc allowlist cố định trong code | Bảng global_settings (dự kiến) | Cập nhật + public chỉ trả allowlist; không lộ internal key | Draft |
| REQ-ADMIN-13 | Hiện trạng | Media | Upload ảnh (giữ) + thư viện media (`TBD`) | /upload/image | Upload OK | Draft |
| REQ-ADMIN-14 | Greentech pattern + chỉ thị 2026-08-14 | Audit logs | Màn hình xem audit log | /admin/audit-logs (mới) | Xem + filter; CHỈ role được phép xem; không sửa/xóa qua API thường; export theo RBAC + redaction; retention `TBD` | Draft |
| REQ-ADMIN-15 | Greentech pattern | Toàn admin | RHF wrapper components + zod schema + 1 Form Create/Edit chung | — | Form nhất quán, validation đúng | Planned |
| REQ-ADMIN-16 | Hiện trạng | Toàn admin | RBAC theo 4 roles hiện tại; menu filter theo role; Consignments thêm menu | /admin/consignments | Đúng quyền từng role | Draft |
| REQ-ADMIN-17 | Quyết định | Toàn admin | Optimistic update giới hạn (toggle hiển thị, thứ tự slider) | — | Không optimistic cho thao tác rủi ro | Approved |

**Tổng REQ-ADMIN: 17**

### 7.10 Data

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-DATA-01 | Chủ dự án | — | — | Phân tích schema-only trước | Báo cáo schema | Draft |
| REQ-DATA-02 | Chủ dự án | — | — | Row count từng bảng | Bảng số liệu | Draft |
| REQ-DATA-03 | Chủ dự án | — | — | Báo cáo constraint/index/duplicate/null | Báo cáo trước migration | Draft |
| REQ-DATA-04 | Chủ dự án | — | — | Dump ẩn danh nếu cần thử migration | Không có PII | Draft |
| REQ-DATA-05 | Chủ dự án | — | — | Bản sao đầy đủ chỉ trong môi trường kiểm soát + phê duyệt | Vùng cách ly | Draft |
| REQ-DATA-06 | Chủ dự án | — | — | Bảo toàn ID, slug, locale, timestamps, media URL | Đối soát sau migration | Draft |
| REQ-DATA-07 | Chủ dự án + chỉ thị 2026-08-14 | — | — | Backup + rollback plan: migration additive/backward-compatible; ưu tiên app rollback (schema giữ nếu app cũ không dùng); backfill có script kiểm chứng/khôi phục riêng; KHÔNG down() migration 3/4; revert schema chỉ khi down() đã kiểm thử + duyệt; không an toàn → forward-fix/restore theo runbook | Có backup; rollback theo chiến lược trên | Draft |
| REQ-DATA-08 | Chủ dự án | — | — | Approval gate trước mọi migration production | Không tự chạy | Approved |
| REQ-DATA-09 | Chủ dự án | — | — | Không ghi PII/secrets vào repo | Scan nội dung | Approved |

**Tổng REQ-DATA: 9**

### 7.11 Deploy

| ID | Nguồn | Trang/module | UI thay đổi | API/DB | Tiêu chí nghiệm thu | Status |
|---|---|---|---|---|---|---|
| REQ-DEPLOY-01 | Chủ dự án | — | — | CI: lint + build + test (3 app) | Build pass | Draft |
| REQ-DEPLOY-02 | Chủ dự án | — | — | Artifact ZIP theo từng app | Artifact tạo được | Draft |
| REQ-DEPLOY-03 | Chủ dự án | — | — | Deploy có phê duyệt thủ công | Không auto-deploy | Draft |
| REQ-DEPLOY-04 | Chủ dự án | — | — | Migration production thủ công riêng; không auto khi boot | Gate phê duyệt | Approved |
| REQ-DEPLOY-05 | Chủ dự án | — | — | Khảo sát cPanel: SSH?, Git Version Control?, Node App Manager/Passenger/PM2?, document root?, restart?, cron/Redis? | Báo cáo khảo sát | TBD |

**Tổng REQ-DEPLOY: 5**

### 7.12 Tổng số requirement được truy vết

| Nhóm | Số lượng |
|---|---|
| REQ-UX | 19 |
| REQ-W | 20 |
| REQ-SVC | 7 |
| REQ-SEO | 10 |
| REQ-CAREER | 6 |
| REQ-BRAND | 5 |
| REQ-ANIM | 8 |
| REQ-BACKEND | 15 |
| REQ-ADMIN | 17 |
| REQ-DATA | 9 |
| REQ-DEPLOY | 5 |
| **TỔNG** | **121** |

---

## 8. Public Website Specification

### 8.1 Sitemap & chiến lược URL

Bảng đầy đủ ở mục 5.6. Chiến lược tổng:

- **Giữ URL** (chỉ viết lại nội dung/layout): `/`, `/gioi-thieu`, `/dich-vu`, `/dich-vu/[slug]`, `/tin-tuc`, `/tin-tuc/[slug]`, `/tuyen-dung`, `/tuyen-dung/[id]`, `/lien-he`, `/tra-cuu`, `/tuyen-ngon`
- **Tạo URL mới**: 6 trang dịch vụ con (slug theo tên dịch vụ, `TBD`)
- **Redirect 301** (canonical đã chốt — REQ-UX-19, TBD-19 Resolved): `/chinh-sach-bao-mat` → 301 → `/dieu-khoan` (canonical giữ nguyên nội dung gộp; KHÔNG tạo `/thong-tin-phap-ly`; không redirect loop/chain). `/quote` → `/contact` (đã có); URL cũ đổi slug → 301
- **Chuyển xuống footer**: Tuyển dụng, Tuyên ngôn (vẫn tồn tại route)
- **Thêm vào nav**: Năng lực hạ tầng (tách trang riêng)
- **Gộp trang**: Điều khoản + Bảo mật → 1 trang pháp lý tại `/dieu-khoan`; khi đa ngôn ngữ, áp dụng cùng nguyên tắc canonical theo locale, kiểm tra mapping với route hiện hữu trước khi triển khai
- **Bỏ**: mục "Khác" ở tin tức, block "Năng lực cung ứng dịch vụ", block "Cập nhật thông tin Logistics biên mậu Việt - Trung", search modal

### 8.2 Homepage

- Hero slider (REQ-UX-05, REQ-W01): slide quản trị từ admin, tự động lướt, click dẫn trang; chuyển mượt từ logo intro (REQ-ANIM-07)
- Block thông tin công ty tách riêng (REQ-UX-06): slogan + title + subtitle + ảnh dịch vụ — nội dung `CẦN XÁC MINH`
- Block số liệu (REQ-UX-07, REQ-W02): khách hàng mục tiêu + KPI — số liệu `CẦN XÁC MINH`
- Dịch vụ (REQ-UX-11, REQ-W04): 6 nhóm + bảng giá `TBD` + CTA
- Quy trình 6 bước (REQ-UX-12, REQ-W03) với hover effects
- Bài viết & Sự kiện (REQ-UX-13): tab Sự kiện/Hoạt động (`TBD` model)
- Chứng chỉ + lộ trình (REQ-UX-14): bảng certificates + timeline — `CẦN XÁC MINH`
- Đối tác (REQ-UX-15): logo partners
- CTA cố định mobile (REQ-W12)

### 8.3 Giới thiệu (REQ-UX-15 — 9 nội dung)

Ảnh bìa (kho bãi/nhân sự/logo) → số liệu → lịch sử (mốc + năm) → tầm nhìn/sứ mệnh/giá trị → cơ cấu tổ chức → chiến lược tương lai → chứng nhận/thành tựu → hiệp hội → đối tác. Nội dung do công ty cung cấp, `CẦN XÁC MINH`.

### 8.4 Sáu trang dịch vụ (REQ-SVC-01..06, REQ-W05..10)

Template chung mỗi trang: Hero/ảnh bìa → Vấn đề khách gặp → Giải pháp/dịch vụ → Quy trình → Thiết bị/năng lực → Lợi ích → FAQ → CTA báo giá. SEO fields đầy đủ (metaTitle/Description/Keywords, ogImage). Slugs theo tên dịch vụ (`TBD`), translation vi/en/zh. LƯU Ý: số bản ghi và nội dung dịch vụ hiện có trong production CHƯA được xác minh (chưa đọc database production) — sẽ kiểm tra bằng schema/row counts và dữ liệu đã ẩn danh hoặc báo cáo từ admin trong Phase 1.

### 8.5 Năng lực hạ tầng (REQ-W17)

Trang riêng: kho bãi (diện tích `CẦN XÁC MINH` — Excel có cả 25ha và >32ha, phải xác minh), bến xe, thiết bị, sơ đồ vị trí, ảnh/video thực tế, năng lực container/ngày (`CẦN XÁC MINH`). Thêm vào nav + sitemap.

### 8.6 Tin tức / SEO (REQ-SEO-01..10, REQ-W16)

10 bài đầu tiên nhập CMS theo sheet SEO; chưa tự động publish; team công ty duyệt nội dung cuối trước publish. Mỗi bài: tiêu đề, keyword chính, search intent, CTA cuối bài, internal links.

### 8.7 Tuyển dụng (REQ-CAREER-01..06, REQ-W13..15)

Hero → Vì sao chọn → Vị trí đang tuyển (JD) → Phúc lợi → Quy trình 4 bước → Form ứng tuyển (họ tên, SĐT, email, vị trí, kinh nghiệm, CV ≤5MB PDF/DOC, ghi chú). Ảnh đội ngũ thật (`CẦN XÁC MINH`).

### 8.8 Liên hệ / Báo giá (REQ-UX-17, REQ-W11)

Form báo giá: loại hàng, tuyến, số lượng, thời gian, thông tin liên hệ, ghi chú + **đính kèm file/ảnh** (REQ-BACKEND-06). SĐT +84 865.865.600. Map văn phòng Hạ Long (link/embed `TBD`). Layout mới.

### 8.9 Header / Footer

- Header: logo ngang, nav (Trang chủ, Giới thiệu, Dịch vụ dropdown 6 nhóm, Năng lực hạ tầng, Tin tức, Liên hệ), CTA "Tư vấn ngay" (REQ-UX-09), SĐT đúng; KHÔNG có search (REQ-UX-08)
- Footer: logo đầy đủ, sơ đồ trang (6 dịch vụ), về công ty (Giới thiệu, Phát triển bền vững `TBD`, Tin tức, Tuyển dụng, Tuyên ngôn, Thông tin pháp lý), form "Liên hệ với chúng tôi" / CTA "Yêu cầu hỗ trợ" → submit "Gửi yêu cầu" (REQ-UX-18)

### 8.10 Animations & interactions

- Logo intro (REQ-ANIM-01..07) — bảng 4.1
- Button: hover scale + màu chuyển (REQ-ANIM-08); hover card dịch vụ, hover bước quy trình (REQ-UX-12); scroll reveal (giữ pattern hiện tại)
- Tôn trọng prefers-reduced-motion toàn site

### 8.11 Accessibility & performance

- a11y: đủ alt, aria, focus visible, contrast ≥ 4.5:1, keyboard navigable, headings H1-H2-H3 đúng cấp
- Performance: font tối ưu, ảnh Cloudinary có transforms, lazy load, không chặn render, LCP < 2.5s (`TBD` đo thực tế), mobile CTA cố định không che nội dung

---

## 9. Admin Panel Specification

### 9.1 Authentication / security

- Login qua Next Route Handler → cookie httpOnly + secure (production) + sameSite phù hợp, maxAge 7 ngày (REQ-ADMIN-01)
- Middleware bảo vệ route server-side (REQ-ADMIN-02)
- CSRF (REQ-ADMIN-01/02): mọi request thay đổi dữ liệu kiểm tra Origin/Referer theo allowlist; nếu kiến trúc yêu cầu dùng CSRF token cho POST/PATCH/PUT/DELETE
- Logout xóa cookie server-side
- Axios: baseURL `/api-backend` (BFF proxy), withCredentials, response unwrap + format lỗi + 401 → logout
- BFF proxy chỉ forward route backend được định nghĩa (tránh open proxy); KHÔNG chuyển tiếp tùy ý header/cookie đến host ngoài backend allowlist
- Access-token expiry/refresh hoặc buộc đăng nhập lại: `TBD`
- Bỏ token khỏi localStorage khi hoàn tất chuyển đổi

### 9.2 Roles

4 roles hiện tại: ADMIN, CONTENT_MANAGER, SALES, OPS. Menu filter theo role (hiện tại). Phân quyền mỗi màn hình theo REQ-ADMIN-16. Role thực tế đang dùng: `TBD` (mục 16).

### 9.3 Dashboard

Giữ 4 KPI (quotes PENDING, consignments IN_TRANSIT, applications, news PUBLISHED) + timeseries + categorical (sau fix REQ-BACKEND-04) + recent activities.

### 9.4 Sliders (mới — REQ-ADMIN-04)

CRUD: ảnh desktop/mobile, link, thứ tự (order), active toggle (optimistic update được phép) + translation vi/en/zh (title, subtitle, cta_label) qua bảng `slider_translations`. Field đầy đủ `TBD` khi đặc tả bảng.

### 9.5 Services / translations

Form RHF+zod: translations vi/en/zh tabs, slug auto (pinyin-pro cho zh), coverImage, featured, categoryId, SEO block, TinyMCE, bảng giá `TBD` (REQ-UX-11). 1 Form dùng chung Create/Edit.

### 9.6 News / categories / SEO

CRUD tin tức + SEO + status DRAFT/PUBLISHED + featured + publishedAt + category; CRUD danh mục (NEWS/SERVICE).

### 9.7 Careers / applications

2 tabs: JD (CRUD, status OPEN/CLOSED, fields theo REQ-CAREER-03) + Hồ sơ ứng tuyển (list, xem CV, trạng thái, xóa) — CV là dữ liệu RIÊNG TƯ: chỉ role phù hợp xem/tải qua signed URL, audit log khi tải/xóa. Xuất CSV `Proposed` — CHỈ role được phép export; KHÔNG xuất raw CV URL/signed URL; chỉ field nghiệp vụ cần thiết; ghi audit event khi export; escape formula injection (`=`, `+`, `-`, `@`); retention file export `TBD` (ưu tiên tạo theo yêu cầu, không lưu lâu dài).

### 9.8 Quotes / attachments

List + filter status + search; chi tiết modal (details, adminNotes, aiSuggestedPrice); xem/tải file đính kèm (REQ-ADMIN-09) — attachments riêng tư, chỉ role phù hợp truy cập (signed URL), audit log khi tải/xóa; status PENDING/CONTACTED/APPROVED/REJECTED. Xuất CSV quotes (nếu có): an toàn giống applications (role được phép, không raw URL, audit, escape formula injection, retention `TBD`).

### 9.9 Partners / certificates

Partners: logo, website, order, active (REQ-ADMIN-10). Certificates: tên, ảnh, tổ chức cấp, năm (REQ-ADMIN-11).

### 9.10 Media / global settings

Media upload giữ nguyên + thư viện media `TBD`. Global settings: SĐT, email, địa chỉ, hotline, Zalo, map link... (REQ-ADMIN-12) — public chỉ trả allowlist key đã duyệt; key nội bộ/credential không public; secret không lưu trong bảng (env/secret store); update validate type/schema + audit đã redact.

### 9.11 Audit logs

Màn hình xem log (module, action, user, recordId, old/new data) — REQ-ADMIN-14, REQ-BACKEND-08. Chỉ role được phép xem; KHÔNG có endpoint sửa/xóa audit qua API thông thường; export audit log áp dụng RBAC + redaction tương tự; retention policy `TBD`.

### 9.12 Form pattern & validation

- Zod schema single source of truth + `z.infer`; RHF wrapper components (RHFInput, RHFSelect, RHFInputNumber, RHFImageUpload, RHFFileUpload, RHFEditor)
- 1 Form dùng chung Create/Edit; page `create/` và `[id]/` mỏng
- `statusConfig` tập trung; optimistic update giới hạn (REQ-ADMIN-17)

---

## 10. Backend & API Specification

### 10.1 Compatibility rules

- Giữ nguyên routes hiện tại (bảng 5.1); mọi thay đổi đều additive (thêm endpoint mới, không đổi format cũ)
- Format response giữ `{statusCode, message, data}`; khi thêm `meta` phải tương thích ngược
- Swagger title đổi sang brand (B12)

### 10.2 Existing endpoints (tóm tắt)

- Auth: POST /auth/register, POST /auth/login, GET /auth/profile
- Users: /admin/users CRUD + reset-password
- Services: GET /services, /services/:id, /services/slug/:locale/:slug + /admin/services CRUD
- Consignments: POST/GET /consignments, GET /consignments/lookup/:awb (public), PATCH/DELETE /consignments/:awb, /consignments/:awb/events + /admin/consignments
- Quotes: POST /quotes (public) + /quotes, /admin/quotes CRUD
- News: GET /news, /news/slug/:locale/:slug + /admin/news
- Categories: /categories + /admin/categories
- Careers: /careers/postings (public), /careers/postings/:id/apply (public upload CV) + admin CRUD + /careers/applications
- Search: GET /search (sẽ bỏ khỏi UI — endpoint có thể giữ)
- Dashboard: /stats/*
- Upload: POST /upload/image

### 10.3 New modules / endpoints (dự kiến — chưa implement)

- `sliders`: GET public + admin CRUD (REQ-ADMIN-04)
- `partners`: GET public + admin CRUD (REQ-ADMIN-10)
- `certificates`: GET public + admin CRUD (REQ-ADMIN-11)
- `global-settings`: GET public + admin bulk update (REQ-ADMIN-12)
- `audit-logs`: GET admin (REQ-ADMIN-14)
- `health`: GET /api/health (REQ-BACKEND-09)
- `attachments`: upload đính kèm quotes/contact (REQ-BACKEND-06 — thiết kế `TBD`)

### 10.4 Upload

CV và attachments (quotes/contact) là dữ liệu RIÊNG TƯ — quy tắc bắt buộc theo REQ-BACKEND-06:

- Chỉ admin có role phù hợp được xem/tải; dùng **private/authenticated asset hoặc signed URL có thời hạn**; KHÔNG dùng public URL mặc định; KHÔNG lộ raw URL trong API public.
- Validate **extension + MIME + magic bytes** (magic bytes chỉ xác thực định dạng — KHÔNG được gọi là malware scanning; malware scanning thực sự là `TBD`, phụ thuộc hạ tầng/dịch vụ quét).
- Tên file an toàn (sanitize + random); nơi lưu: Cloudinary folder `attachments`.
- Lưu metadata: owner/module, storage key, original filename đã sanitize, MIME, size, checksum, createdAt.
- Có retention policy và cơ chế xóa; audit log khi admin tải/xóa file nhạy cảm.
- CV ứng tuyển được bảo vệ tương tự attachments (không chỉ file liên hệ).
- Hiện có 2 luồng Cloudinary (site_assets + cvs) — giữ; thêm luồng attachments theo đặc tả trên.

### 10.5 Validation / Rate limiting / Error handling / Health

- ValidationPipe strict (whitelist + forbidNonWhitelisted + transform) — giữ
- Rate limit theo IP/session (REQ-BACKEND-07)
- Có thể kết hợp email đã normalize + payload fingerprint
- Deduplication CHỈ với payload thực sự giống nhau trong cửa sổ ngắn
- KHÔNG log email thô
- Ngưỡng cụ thể `TBD` sau khi đánh giá traffic
- AllExceptionsFilter chuẩn hóa (REQ-BACKEND-10)
- Health check Terminus (REQ-BACKEND-09)

### 10.6 Deprecated aliases

- `/careers/postingss` — ĐÃ RE-VERIFY (2026-08-14): KHÔNG phải typo. Backend có route admin chủ đích `@Get('postingss/')` (`backend/src/careers/careers.controller.ts:91`, Roles ADMIN/CONTENT_MANAGER); admin panel gọi đúng (`admin-panel-frontend/src/app/(admin)/careers/_components/JobPostingsTab.tsx:23`); public dùng `/careers/postings` riêng (`careers.controller.ts:43-57`). KHÔNG cần alias, KHÔNG xóa route. Cảnh báo mới (B14): `GET /careers/postings/:id` public load kèm `applications` → sửa ở phase 1 fix.

---

## 11. Database & Migration Plan

### 11.1 Current schema

12 bảng (mục 5.4) + 4 migrations (mục 5.5). TypeORM + pg driver. Repository/docker-compose cấu hình `postgres:15-alpine`; engine/version database production chưa kiểm chứng trực tiếp — xác nhận qua khảo sát Phase 1 (xem mục 5.8).

### 11.2 Proposed additive changes (DỰ KIẾN — chưa thực hiện)

| Bảng mới | Lý do nghiệp vụ | Field đề xuất | Tác động migration |
|---|---|---|---|
| `sliders` | REQ-UX-05 — hero slider quản trị từ admin | id, image_desktop, image_mobile, link, order, is_active, timestamps — dữ liệu KHÔNG phụ thuộc ngôn ngữ | Migration additive mới (CREATE TABLE) |
| `slider_translations` | Nội dung đa ngôn ngữ của slider (vi/en/zh) | id, slider_id (FK → sliders), locale, title, subtitle, cta_label, unique (slider_id, locale) | CREATE TABLE |
| `partners` | REQ-UX-15 — đối tác trang chủ/giới thiệu | id, name, logo_url, website_url, order, is_active — field trung lập ngôn ngữ | CREATE TABLE |
| `certificates` | REQ-UX-14 — chứng chỉ | id, name, image_url, issuer, issued_year, order — field trung lập ngôn ngữ | CREATE TABLE |
| `global_settings` | REQ-ADMIN-12 — SĐT/email/hotline/Zalo/map | key (PK), value jsonb, updated_at | CREATE TABLE |
| `audit_logs` | REQ-BACKEND-08 — truy vết thay đổi admin | id, user_id, action, module, record_id, old_data jsonb, new_data jsonb, created_at — KHÔNG log toàn bộ request body; CÂM password/hash/token/cookie/secret/authorization header; redact CV URL, signed URL, storage credential; quotes/applications/users dùng allowlist field thay vì toàn bộ old/new data; PII mask hoặc chỉ ghi field đã đổi; chỉ role được phép xem; không sửa/xóa qua API thường; retention `TBD` | CREATE TABLE + FK users |
| `attachments` | REQ-BACKEND-06 — file đính kèm | `TBD` — chỉ khi chốt multi-file | CREATE TABLE (TBD) |

Ghi chú i18n: nếu partners/certificates sau này cần nội dung đa ngôn ngữ → dùng bảng translation riêng hoặc chỉ giữ field trung lập; KHÔNG dùng JSONB i18n trừ khi có quyết định mới.

- KHÔNG thêm `search_logs` (không có use case duyệt).
- Có thể thêm cột `deleted_at` (soft delete) — `TBD`, cân nhắc trong phase 5.

### 11.3 Data mapping & backfill

- Không đổi bảng translation; không đổi ID/slug/locale/timestamps (REQ-DATA-06)
- Backfill chỉ cho cột mới nullable (nếu có) — quy tắc: mặc định an toàn, không ghi đè dữ liệu cũ

### 11.4 Validation queries

Trước mọi migration: đếm rows, kiểm tra duplicate slug, null categoryId, locale thiếu, orphan records — xuất báo cáo (REQ-DATA-02/03).

### 11.5 Backup / Rollback

- Backup đầy đủ trước mọi thay đổi (pg_dump) + snapshot Cloudinary URL list.
- Migrations mới phải **additive và backward-compatible**.
- **Ưu tiên rollback APPLICATION về release trước mà KHÔNG rollback schema**; schema mới có thể được giữ lại nếu app cũ không sử dụng.
- Migration có backfill: bắt buộc có backup và script kiểm chứng/khôi phục riêng.
- **KHÔNG chạy `down()` của migrations 3/4.**
- Revert schema CHỈ khi migration mới có `down()` đã được kiểm thử và được phê duyệt.
- Khi không thể rollback an toàn: dùng forward-fix hoặc restore từ backup theo runbook (KHÔNG mặc định coi migration revert là rollback production).
- Test trên staging trước production.

### 11.6 Production approval gate

Mọi migration production: backup → staging verify → PHÊ DUYỆT THỦ CÔNG → chạy → verify → ghi Decision Log. Không auto-run khi boot (REQ-BACKEND-05), không auto-run trong CI (REQ-DEPLOY-04).

---

## 12. SEO & URL Preservation

- Bảng URL đầy đủ: mục 5.6 + chiến lược mục 8.1
- Redirect mapping (canonical đã chốt — REQ-UX-19, TBD-19 Resolved): `/chinh-sach-bao-mat` → 301 → `/dieu-khoan` (giữ URL, nội dung gộp; không tạo `/thong-tin-phap-ly`; không loop/chain). Metadata, canonical tag, sitemap và internal links trỏ `/dieu-khoan` khi triển khai; đa ngôn ngữ giữ nguyên tắc canonical theo locale, kiểm tra mapping với route hiện hữu trước khi triển khai. `/quote` → `/contact` (giữ); mọi đổi slug → 301 (kèm bảng mapping trước khi đổi)
- Metadata: title/meta/OG/hreflang/JSON-LD giữ pattern hiện tại; thêm FAQ schema cho bài FAQ (REQ-SEO-10)
- Kiểm tra broken links + media sau khi đổi (sitemap resubmit, Google Search Console)
- 10 bài SEO: nhập CMS thủ công, KHÔNG auto-publish, team công ty duyệt cuối (REQ-W16, REQ-SEO-*)

---

## 13. CI/CD & cPanel Deployment

### 13.1 Current deployment

Build local → ZIP → upload cPanel (thủ công). Production: cPanel; DB production trong hạ tầng cPanel.

### 13.2 CI checks

GitHub Actions (đề xuất): `npm ci` → lint → build (`nest build` / `next build`) → test, cho từng app (backend, frontend, admin). Chạy trên PR + push main. KHÔNG chạy migration trong CI.

### 13.3 Artifact & approval

Tạo artifact ZIP từng app (chú ý: frontend cần `NEXT_PUBLIC_API_URL` + `TINYMCE_API_KEY` làm build args). Deploy yêu cầu phê duyệt thủ công (REQ-DEPLOY-03).

### 13.4 Deployment & migration gate

- Migration: bước riêng, thủ công (mục 11.6)
- Deploy: tùy kết quả khảo sát cPanel (SSH action? Git Version Control? Manual upload?) — `TBD` (REQ-DEPLOY-05)
- Rollback: giữ bản ZIP trước + backup DB trước; ưu tiên application rollback (schema additive giữ nguyên) theo chiến lược mục 11.5

---

## 14. Implementation Phases

| Phase | Scope | Preconditions | Deliverables | Status |
|---|---|---|---|---|
| 1 | **Survey & kiểm kê**: DB schema-only + row counts, khảo sát cPanel (SSH/Git VCS/Node App Manager/restart/cron-Redis), kiểm kê URL/media, re-verify route `postingss` | Duyệt handoff | Báo cáo khảo sát; bảng số liệu DB | Not started |
| 2 | **Vá lỗi backend/deployment KHÔNG đổi schema** (B2–B12, CORS, Dockerfile, env, bỏ auto-migrate boot) | Backup DB | Backend ổn định | Not started |
| 3 | **Chốt API contract + backend CMS/migrations additive trên STAGING** (sliders/translations, partners, certificates, global-settings, audit-logs, health, attachments); design system có thể khởi động SONG SONG ngay khi contract được chốt dần (không chờ hết Phase 3) | Phase 2 | API contract + backend CMS trên staging | Not started |
| 4 | **Design system + public frontend mới** (brand mới, 6 dịch vụ, hero slider...); có thể chạy SONG SONG bằng mock API sau khi contract được chốt (không bị đẩy xuống cuối dự án) | Phase 3 contract | Frontend mới | Not started |
| 5 | **Bảo mật + viết lại admin panel** theo API ĐÃ CHỐT (BFF proxy, httpOnly, RHF+zod) | Phase 3 | Admin mới | Not started |
| 6 | **Migration/backfill + kiểm thử tích hợp trên STAGING** | Bản sao staging + phê duyệt | Migration đã test trên staging | Not started |
| 7 | **UAT, đối soát dữ liệu/SEO, CI/CD cPanel, cutover có phê duyệt** | Phase 5+6 | Go-live | Not started |
| 8 | **Nâng framework** (Next 16.1→16.2, React 18→19, antd 5→6) — TÙY CHỌN, từng bước sau khi hệ thống ổn định | Hệ thống ổn định | Versions mới verify | Not started |

---

## 15. Acceptance Criteria

### Backend
- AC-B1: Bảo toàn 12 feature module hiện có, không lỗi runtime; API cũ tương thích (REQ-BACKEND-02) — tổng module cuối cùng có thể TĂNG (sliders/health/audit...) và không bắt buộc phải bằng 12
- AC-B2: /stats/categorical không lỗi (REQ-BACKEND-04)
- AC-B3: Boot không tự chạy migration (REQ-BACKEND-05)
- AC-B4: Upload đính kèm đúng đặc tả 9 mục (REQ-BACKEND-06)
- AC-B5: Rate limit + anti-spam hoạt động (REQ-BACKEND-07/11)
- AC-B6: Audit log ghi đủ hành động; KHÔNG chứa password/hash/token/secret/CV URL/signed URL; quotes/applications/users theo allowlist field; PII mask; không có API sửa/xóa audit; chỉ role được phép xem (REQ-BACKEND-08)
- AC-B7: Health check trả status (REQ-BACKEND-09)
- AC-B8: Lỗi format nhất quán (REQ-BACKEND-10)
- AC-B9: CORS production đóng (REQ-BACKEND-12)
- AC-B10: Không lưu password plaintext (REQ-BACKEND-14)
- AC-B11: Alias postingss hoạt động; không breaking client cũ (REQ-BACKEND-03)

### Admin
- AC-A1: Cookie httpOnly + secure + sameSite; middleware chặn truy cập trái phép; Origin/Referer allowlist cho request thay đổi dữ liệu; CSRF token cho POST/PATCH/PUT/DELETE nếu kiến trúc yêu cầu; logout xóa cookie server-side; không open proxy (REQ-ADMIN-01/02)
- AC-A2: CRUD đủ các module theo 9.4–9.11
- AC-A3: Form RHF+zod validation đúng; 1 Form chung Create/Edit (REQ-ADMIN-15)
- AC-A4: RBAC đúng baseline 4 roles từ source; nghiệm thu RBAC cuối cùng phụ thuộc `TBD-09` (roles thực tế đang dùng)
- AC-A5: Optimistic update chỉ ở hành động cho phép (REQ-ADMIN-17)
- AC-A6: Dashboard không lỗi; consignments có menu (REQ-ADMIN-03/16)

### Public frontend
- AC-F1: Brand tokens đúng (#233871/#0c9344), không Dark Mode (REQ-BRAND-02/03)
- AC-F2: Logo/font đúng asset 2026 + fallback (REQ-BRAND-04/05)
- AC-F3: Logo intro 1–1,5s, 1 lần/session, không chặn trang, reduced-motion (REQ-ANIM-*)
- AC-F4: Hero slider quản trị từ admin (REQ-UX-05)
- AC-F5: 6 trang dịch vụ con + trang tổng (REQ-SVC-*)
- AC-F6: i18n vi/en/zh đầy đủ; SEO metadata đúng (REQ-W19/W20)
- AC-F7: CTA cố định mobile hoạt động (REQ-W12)
- AC-F8: SĐT +84 865.865.600 đúng mọi nơi (REQ-UX-03) — đã chủ dự án xác nhận (TBD-13 Resolved)
- AC-F9: Số liệu KPI doanh nghiệp (diện tích, container/ngày, thông quan, chứng chỉ, Top 10...) chưa được tính là PASS cho tới khi chủ dự án xác minh (TBD-12)

### Data
- AC-D1: Không mất dữ liệu sau migration; đối soát ID/slug/locale/timestamps/media (REQ-DATA-06)
- AC-D2: Backup + rollback sẵn sàng — theo chiến lược application rollback + schema additive/backward-compatible, không mặc định migration revert (REQ-DATA-07)
- AC-D3: Không PII trong repo (REQ-DATA-09)
- AC-D4: Báo cáo constraint/duplicate/null hoàn tất trước migration (REQ-DATA-03)

### SEO
- AC-S1: Redirect 301 đúng mapping: `/chinh-sach-bao-mat` → `/dieu-khoan`; không 404; canonical tag/sitemap/internal link trỏ `/dieu-khoan` (REQ-UX-19)
- AC-S2: 10 bài SEO nhập CMS, chưa publish, team duyệt (REQ-W16)
- AC-S3: FAQ schema bài #10 (REQ-SEO-10)
- AC-S4: Sitemap + robots cập nhật (gồm infrastructure)

### Performance & Accessibility
- AC-P1: LCP < 2.5s (`TBD` đo thật); font tối ưu; ảnh tối ưu (8.11)
- AC-P2: a11y cơ bản đạt (alt, aria, focus, contrast, keyboard)

### Security
- AC-SEC1: Không secret trong code/repo; .env ngoài Git
- AC-SEC2: Token admin không lộ client (httpOnly)
- AC-SEC3: Upload an toàn: validate extension + MIME + magic bytes (magic bytes ≠ malware scanning; malware scanning thật `TBD`), giới hạn kích thước/số file (REQ-BACKEND-06)
- AC-SEC4: CV/attachments: private/authenticated asset hoặc signed URL có thời hạn; CHỈ admin có role phù hợp xem/tải; không lộ raw public URL trong API public; có retention policy + cơ chế xóa; audit log khi admin tải/xóa (REQ-BACKEND-06, REQ-ADMIN-08/09)

### Deployment
- AC-DEP1: CI pass (lint/build/test) trước deploy (REQ-DEPLOY-01)
- AC-DEP2: Deploy có phê duyệt; migration có gate (REQ-DEPLOY-03/04)
- AC-DEP3: Rollback khả thi sau mỗi release (13.4)

---

## 16. TBD / Questions Requiring User Approval

| ID | Question | Impact | Recommended option | User decision |
|---|---|---|---|---|
| TBD-01 | cPanel có SSH không? | Cách deploy CI/CD | Nếu có: ssh-action; nếu không: upload ZIP qua UI | Awaiting |
| TBD-02 | cPanel có Git Version Control? | Deploy strategy | Dùng nếu có | Awaiting |
| TBD-03 | Node App Manager / Passenger / PM2? | Cách restart | Theo kết quả khảo sát | Awaiting |
| TBD-04 | Document root và thư mục release? | Deploy | Khảo sát | Awaiting |
| TBD-05 | Cách restart ứng dụng? | Deploy | Khảo sát | Awaiting |
| TBD-06 | Khả năng chạy cron/worker/Redis trên cPanel? | Queue (BullMQ?) | Chỉ làm queue nếu có Redis | Awaiting |
| TBD-07 | Có thể cung cấp schema-only DB + row counts? | Phase 1 khảo sát | Schema-only trước, dump ẩn danh sau | Awaiting |
| TBD-08 | Email thật cho notifications (reset password, lead, CV)? | REQ-BACKEND-15 | SMTP của công ty; `TBD` | Awaiting |
| TBD-09 | Roles thực tế đang dùng? | RBAC admin | Xác nhận danh sách + số user/role | Awaiting |
| TBD-10 | Ai duyệt nội dung 6 dịch vụ + 10 bài SEO? | Phase 3 nội dung | Team content + chủ dự án | Awaiting |
| TBD-11 | Chọn biến thể logo cho từng vị trí? | REQ-BRAND-04 | Dùng asset chính thức; chọn khi implement | Awaiting |
| TBD-12 | Số liệu thật: diện tích (25ha hay >32ha?), container/ngày, thông quan <2h, năm 2008, Top 10, chứng chỉ | REQ-UX-06/07/14, REQ-W17 | Xác minh từ công ty; mọi số hiển thị trước publish | Awaiting |
| TBD-13 | ✅ ĐÃ ĐÓNG: SĐT +84 865.865.600 là hotline chính thức — chủ dự án xác nhận 2026-08-14 (Checkpoint 6), được phép hiển thị public | REQ-UX-03 | Đóng — giá trị nghiệp vụ chuẩn `+84 865.865.600` | Resolved |
| TBD-14 | Upload đính kèm: 1 file hay nhiều file? danh sách loại file? | REQ-BACKEND-06 | Nhiều file → bảng attachments; đợi phân tích DB | Awaiting |
| TBD-15 | Bảng giá dịch vụ hiển thị? (Excel đề xuất thêm) | REQ-UX-11 | `TBD` — nếu có, thêm field price | Awaiting |
| TBD-16 | Tab Sự kiện/Hoạt động: model dữ liệu? | REQ-UX-13 | Thêm loại bài/field category | Awaiting |
| TBD-17 | ✅ ĐÃ ĐÓNG: SOTRANS `https://sotransgroup.vn/` ; Viettel Logistics `https://viettellogistics.com.vn/vi` (chủ dự án cung cấp 2026-08-14) | Source refs | Đóng — vẫn ghi rõ không có file snapshot riêng trong repo | Resolved |
| TBD-18 | GTM-5BD4XCML giữ? TinyMCE key quản lý thế nào? | Frontend/admin | Giữ; key qua env | Awaiting |
| TBD-19 | ✅ ĐÃ ĐÓNG: canonical trang pháp lý = `/dieu-khoan` (giữ URL, nội dung gộp); `/chinh-sach-bao-mat` → 301 → `/dieu-khoan`; không tạo `/thong-tin-phap-ly` — chủ dự án quyết định 2026-08-14 (Checkpoint 6) | REQ-UX-19 | Đóng — mapping i18n theo locale cần kiểm tra route hiện hữu khi triển khai | Resolved |
| TBD-20 | Staging environment có sẵn? | Phase 6 | Tạo trên cPanel staging hoặc local | Awaiting |
| TBD-21 | Soft delete `deleted_at` có áp dụng không? | Phase 5 | Có (giảm rủi ro mất dữ liệu) | Awaiting |
| TBD-22 | `/auth/register` — production có cho đăng ký public không? (source verified 2026-08-14: endpoint KHÔNG có guard `auth.controller.ts:14-20`, không rate limit; chưa xác minh production deploy/cấu hình reverse-proxy) | Auth | Nếu production public: khóa admin-only/setup-only; không cho tự chọn role đặc quyền; KHÔNG thay đổi endpoint ở bước tài liệu | Awaiting (source verified, chờ Phase 1B) |

---

## 17. Risks

| Risk | Severity | Mitigation | Status |
|---|---|---|---|
| **Lộ hồ sơ ứng viên qua public API (B14): `GET /careers/postings/:id` trả `applications` (tên/email/phone/cvPath)** | **Critical** | Sửa ở phase 1 fix: bỏ relations applications khỏi query public; GHI Decision Log + audit | Open — verified source 2026-08-14 |
| `/auth/register` public không guard: nguy cơ tạo tài khoản trái phép | Cao | Đánh giá khóa admin-only/setup-only sau Phase 1B (TBD-22) | Open — verified source 2026-08-14 |
| Migration lỗi trên production đang chạy | Cao | Backup + staging + approval gate; additive-only | Mitigated by process |
| Auto-migrate khi boot còn tồn tại | Cao | Gỡ trong phase 2 (REQ-BACKEND-05) | Open |
| Token admin localStorage bị XSS | Cao | BFF proxy + httpOnly (phase 4) | Open |
| Mất dữ liệu translations/consignments khi thay đổi | Cao | Không đổi schema hiện có; test staging | Mitigated by design |
| Nội dung số liệu sai (chưa xác minh) công khai | Cao | CẦN XÁC MINH trước publish; team duyệt | Open |
| Deploy thủ công sai (ZIP) vẫn tiếp diễn | Trung bình | CI/CD phase 7 | Open |
| cPanel thiếu SSH/cron/Redis | Trung bình | Khảo sát trước; thiết kế theo khả năng | TBD |
| Nâng phiên bản framework cùng lúc viết lại | Cao | Tách phase 8 riêng | Mitigated by plan |
| Ảnh/logo asset sai bản quyền font | Thấp | Dùng font công ty cung cấp; xác nhận license | TBD |
| Route alias bị bỏ sớm làm hỏng client cũ | Trung bình | Giữ alias tới khi client cũ hết dùng | Mitigated |

---

## 18. Decision Log

| Date | Decision | Reason | Supersedes | Approved by |
|---|---|---|---|---|
| 2026-08-14 | Chỉ tạo 1 file PROJECT_HANDOFF.md, không tạo 8 file docs/specs/ | Yêu cầu chủ dự án (single source of truth) | Kế hoạch 8 tài liệu Rev.3 | Chủ dự án |
| 2026-08-14 | Brand: Tà Lùng Quang Minh Logistics; Primary #233871; Secondary #0c9344; không Dark Mode | Quyết định mới nhất | Màu xanh #0b9444 trong Excel | Chủ dự án |
| 2026-08-14 | Giữ TypeORM + PostgreSQL + translation tách bảng | Đang hoạt động ổn định, an toàn giai đoạn đầu | — | Chủ dự án |
| 2026-08-14 | Greentech chỉ là nguồn pattern, read-only | Tránh sao chép code/ORM khác biệt | — | Chủ dự án |
| 2026-08-14 | Migration production thủ công có phê duyệt; bỏ auto-run khi boot | Giảm rủi ro production | Auto-migrate hiện tại | Chủ dự án |
| 2026-08-14 | Giữ route /careers/postingss làm alias tạm — có điều kiện RE-VERIFY tại checkpoint code đầu tiên (decorator/public/admin/frontend URL), duplicate handler cho POST/PATCH, xóa alias sau khi kiểm tra access log | Tránh hỏng client đang chạy | — | Chủ dự án |
| 2026-08-14 | Optimistic update chỉ cho hành động ít rủi ro | Bảo toàn dữ liệu | — | Chủ dự án |
| 2026-08-14 | Không thêm search_logs; bỏ tìm kiếm khỏi nav | Excel yêu cầu; không có use case | — | Chủ dự án |
| 2026-08-14 | Public frontend không để giai đoạn cuối — thứ tự 8 phase mới | Yêu cầu chủ dự án | Thứ tự cũ (frontend cuối) | Chủ dự án |
| 2026-08-14 | Nâng framework tách riêng, giai đoạn tùy chọn sau khi ổn định | Giảm lỗi khi nâng version | — | Chủ dự án |
| 2026-08-14 | Không tự kết nối cPanel/SSH/DB; khảo sát phải có phê duyệt riêng | An toàn production | — | Chủ dự án |
| 2026-08-14 | Hiệu chỉnh phase order: Phase 3 = chốt API contract + backend CMS trên staging; Phase 4 = design system + frontend (chạy song song bằng mock API); Phase 5 = admin theo API đã chốt | Loại phụ thuộc vòng (frontend/admin cần sliders/partners/settings trước khi backend CMS tồn tại) | Thứ tự 8 phase 2026-08-14 (bản cũ) | Chủ dự án |
| 2026-08-14 | CV và attachments là dữ liệu riêng tư: admin-only + signed URL/private asset; không lộ raw public URL; retention + audit; magic-byte ≠ malware scanning | Bảo vệ dữ liệu khách hàng/ứng viên | Dùng public URL mặc định (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | Đóng TBD-17: SOTRANS = https://sotransgroup.vn/, Viettel Logistics = https://viettellogistics.com.vn/vi | URL do chủ dự án cung cấp | — | Chủ dự án |
| 2026-08-14 | Chống spam dựa trên rate limit IP/session + payload fingerprint; KHÔNG chặn máy móc mọi lần trùng email; không log email thô; ngưỡng TBD | Tránh chặn yêu cầu hợp lệ | Chặn trùng email 5 phút (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | Audit log: KHÔNG log body máy móc; CẤM password/hash/token/cookie/secret/auth header; redact CV URL/signed URL/storage credential; allowlist field cho quotes/applications/users; PII mask; chỉ role được phép xem; không sửa/xóa qua API; retention TBD | Bảo vệ dữ liệu nhạy cảm trong log | Log toàn bộ old/new data (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | CSRF cho BFF cookie: Origin/Referer allowlist cho request thay đổi dữ liệu; CSRF token cho POST/PATCH/PUT/DELETE nếu kiến trúc yêu cầu; logout xóa cookie server-side; BFF chỉ proxy route được định nghĩa | Bảo vệ cookie httpOnly khỏi CSRF/open proxy | Chỉ cookie httpOnly, chưa đặc tả CSRF | Chủ dự án |
| 2026-08-14 | global_settings tách public/internal: public chỉ allowlist key đã duyệt; secret KHÔNG lưu trong bảng (env/secret store); update validate type/schema + audit redact | Ngăn lộ cấu hình nội bộ/credential | Public toàn bộ settings (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | /auth/register: RE-VERIFY guard + hành vi thực tế trong Survey (TBD-22); nếu public → khóa admin-only/setup-only; không cho tự chọn role đặc quyền; chưa đổi endpoint ở bước tài liệu | Giảm rủi ro tài khoản trái phép | Giữ nguyên register hiện tại | Chủ dự án |
| 2026-08-14 | CSV export: chỉ role được phép; không export raw CV URL/signed URL; audit khi export; escape formula injection (`=`, `+`, `-`, `@`); retention file export TBD | An toàn dữ liệu khi export | CSV không kiểm soát (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | PHÊ DUYỆT baseline: `PROJECT_HANDOFF.md` là specification baseline chính thức của dự án | Chủ dự án duyệt sau Checkpoint 4 | Draft — corrected, awaiting user approval | Chủ dự án |
| 2026-08-14 | 121 requirements là baseline truy vết hiện tại | Kiểm đếm bằng script | — | Chủ dự án |
| 2026-08-14 | 22 TBD gồm 21 mở (Awaiting) + 1 resolved (TBD-17) | Kiểm đếm mục 16 | — | Chủ dự án |
| 2026-08-14 | MỌI thay đổi yêu cầu sau này phải cập nhật handoff + Decision Log | Rule 9 non-negotiable | — | Chủ dự án |
| 2026-08-14 | HOTLINE chính thức: `+84 865.865.600` — đã chủ dự án xác nhận, được phép hiển thị công khai (đóng TBD-13) | Xác nhận trực tiếp từ chủ dự án | Hotline chưa xác minh (`CẦN XÁC MINH`) | Chủ dự án |
| 2026-08-14 | CANONICAL trang pháp lý = `/dieu-khoan` (giữ URL, nội dung gộp Điều khoản + Bảo mật); redirect 301 duy nhất `/chinh-sach-bao-mat` → `/dieu-khoan`; KHÔNG tạo `/thong-tin-phap-ly`; metadata/canonical/sitemap/internal link trỏ `/dieu-khoan`; đa ngôn ngữ giữ nguyên tắc canonical theo locale (đóng TBD-19) | Xác nhận trực tiếp từ chủ dự án | Phương án (a) `/thong-tin-phap-ly` đã được cân nhắc nhưng KHÔNG được chọn | Chủ dự án |
| 2026-08-14 | PHASE 1A VERIFIED: `/careers/postingss/` KHÔNG phải typo — route admin có chủ đích (`@Get('postingss/')` careers.controller.ts:91, Roles ADMIN/CONTENT_MANAGER), admin gọi đúng (JobPostingsTab.tsx:23), public dùng `/careers/postings`. Bỏ điều kiện re-verify (REQ-BACKEND-03 → Approved verified), bỏ kế hoạch alias | Đối chiếu source 2 phía (backend + admin) tại Phase 1A | Giữ route làm alias tạm + RE-VERIFY (quan điểm cũ) | Chủ dự án |
| 2026-08-14 | PHASE 1A VERIFIED (Critical): `GET /careers/postings/:id` (public, không guard) trả kèm `relations: ['applications']` → LỘ tên/email/phone/cvPath ứng viên. B14 mới. Sửa ở phase 1 fix: bỏ applications khỏi query public (giữ cho admin detail); thêm audit. KHÔNG đổi contract public | Bảo vệ PII ứng viên | — | Chủ dự án |
| 2026-08-14 | PHASE 1A VERIFIED: `/auth/register` không guard (auth.controller.ts:14-20), không rate limit. TBD-22 giữ Awaiting — cần Phase 1B xác nhận production (deploy cấu hình, reverse proxy, có public route thật hay không) | Source xác nhận code, chưa xác minh production | — | Chủ dự án |
| 2026-08-14 | PHASE 1A VERIFIED: website public canonical = `www.talunglogistics.com` (non-www → 307); sitemap/robots tồn tại; `/dieu-khoan` + `/chinh-sach-bao-mat` live 200 CHƯA có 301 (đúng kế hoạch triển khai TBD-19 khi sửa trang); `/quote` → `/lien-he` 200 đã hoạt động; `/nang-luc-ha-tang` live 200 | GET survey website public (chỉ GET/HEAD) | Website cũ `CẦN XÁC MINH` trong Source References | Chủ dự án |
| 2026-08-14 | PHASE 1A VERIFIED: Greentech naming controller thực tế = `@Controller('admin/...')` trong `<module>.controller.ts` + `<module>.public.controller.ts` (vd news.controller.ts:31-32, news.public.controller.ts:7) — KHÔNG có `*.admin.controller.ts` riêng; AllExceptionsFilter `common/filters/http-exception.filter.ts:11-67` + TransformInterceptor `common/interceptors/transform.interceptor.ts:27-46`; AuditLogsModule @Global `audit-logs.module.ts:7,12` + `AuditLog @@map("audit_logs")` + `audit-logs.service.ts:13-47` | Đối chiếu source Greentech Phase 1A | Giả định `*.admin.controller.ts` trong mapping cũ | Chủ dự án |
| 2026-08-31 | Tạo branch `develop` cho môi trường phát triển / staging; ghi nhận kiến trúc CI/CD: Frontend auto-build/deploy trên Vercel, Backend auto-build/deploy trên Render, Database PostgreSQL đặt trên cPanel | Chỉ thị chủ dự án — tách biệt nhánh phát triển, kiểm soát auto-deploy | — | Chủ dự án |

---

## 19. Work Log

### Checkpoint 1 — initial (2026-08-14, document creation)

- Completed: Tạo `PROJECT_HANDOFF.md` — hợp nhất toàn bộ đặc tả Rev.3, kiểm kê hệ thống, 121 requirement truy vết, 21 TBD, 17 risks, 11 decisions
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (new — file duy nhất)
- Tests/checks run: Kiểm đếm trực tiếp 12 module (ls src/), 12 bảng (@Entity), 4 migrations (ls), 14 nhóm route frontend + [...rest]; không chạy test/build; không đọc .env, DB, SSH, cPanel
- Results: File tạo xong; số liệu khớp kiểm đếm; không chứa secrets/PII; không sửa Greentech/source code
- Remaining issues: Chưa có phê duyệt để bắt đầu Phase 1; 21 TBD trong mục 16
- Exact next step: Chờ chủ dự án review + phê duyệt

### Checkpoint 2 — Hiệu chỉnh tài liệu (2026-08-14)

- Completed: 14 hiệu chỉnh theo chỉ thị review — (1) Rule 7 PII/business contact, (2) Rule 12 packages, (3) PostgreSQL production chưa kiểm chứng, (4) sliders → language-neutral + slider_translations, (5) 6 nhóm dịch vụ = target chưa xác minh, (6) CV/attachments private access, (7) anti-spam payload-based, (8) postingss re-verify, (9) legal redirect 2 phương án canonical, (10) rollback app-first, (11) phase order loại phụ thuộc vòng, (12) acceptance criteria, (13) đóng TBD-17, (14) cập nhật snapshot/decision/work log
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (chỉ file này)
- Tests/checks run: Đếm lại requirement bằng script (121 — không thêm requirement mới, chỉ mở rộng yêu cầu cũ); TBD: 21 → 20 mở (TBD-17 đóng); không chạy build/test; không đọc .env/DB/cPanel
- Results: File cập nhật xong; không chứa secrets/PII; không sửa Greentech/source code/docs/references
- Remaining issues: TBD còn 20 mở (mục 16); chưa có phê duyệt Phase 1
- Exact next step: **Chờ chủ dự án review các hiệu chỉnh + duyệt.** Sau phê duyệt: Checkpoint 1 (KHẢO SÁT CHỈ — không thay đổi gì): (a) xác nhận SSH/Git Version Control/Node App Manager/document root/restart/cron-Redis trên cPanel; (b) backup DB do chủ dự án thực hiện + schema-only + row counts; (c) kiểm kê URL live + media; (d) re-verify route `postingss`. KHÔNG tự kết nối cPanel/SSH/DB khi chưa có phê duyệt riêng.

### Checkpoint 3 — Kiểm chứng hiệu chỉnh review (2026-08-14, tái review chủ dự án)

- Completed: Chủ dự án tái review với cùng 14 chỉ thị → đối chiếu TOÀN BỘ từng mục trong file (đã áp dụng tại Checkpoint 2): Rule 7, Rule 12, PostgreSQL production (3.0/5.8/11.1), sliders/slider_translations (9.4/11.2), 6 dịch vụ target (REQ-UX-11/8.4), REQ-BACKEND-06 (10.4/AC-SEC3/AC-SEC4/REQ-ADMIN-08/09/9.7/9.8), REQ-BACKEND-11, postingss re-verify (B1/REQ-BACKEND-03/10.6), legal redirect 2 phương án canonical (REQ-UX-19/8.1/12/TBD-19), rollback app-first (11.5/REQ-DATA-07/13.4), phase order (14), acceptance criteria (AC-B1/A4/F9/D2/SEC3/4), TBD-17 đóng (16/20), Snapshot/Decision Log/Work Log. Bổ sung duy nhất: Phase 3 ghi rõ design system chạy SONG SONG khi contract chốt dần.
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (chỉ file này — 1 dòng Phase 3, 1 dòng Snapshot, 1 mục Work Log)
- Tests/checks run: Đếm lại bằng script — requirement rows 122, unique ID 121 (khớp mục 7.12, không thêm requirement mới); TBD rows 21, mở (Awaiting) 20 (TBD-17 Resolved); không chạy build/test; không đọc .env/DB/cPanel; không sửa Greentech/source/docs/references
- Results: File nhất quán với đầy đủ 14 chỉ thị review; không chứa secrets/PII
- Remaining issues: TBD còn 20 mở (mục 16); chưa có phê duyệt Phase 1
- Exact next step: **Chờ chủ dự án review + duyệt handoff.** Sau phê duyệt: Checkpoint 1 (KHẢO SÁT CHỈ) theo đúng Checkpoint 2 — KHÔNG tự kết nối cPanel/SSH/DB khi chưa có phê duyệt riêng.

### Checkpoint 4 — Hiệu chỉnh bảo mật cuối (2026-08-14, review chủ dự án)

- Completed: 6 chỉ thị bảo mật — (1) 10.5 bỏ "anti-spam trùng email" mâu thuẫn, thống nhất với REQ-BACKEND-11 (rate limit IP/session + payload fingerprint, không log email thô, ngưỡng TBD); (2) REQ-BACKEND-08 + 9.11 + 11.2 audit_logs + REQ-ADMIN-14 + AC-B6 (không log body máy móc, cấm secret/token/CV URL, allowlist field, PII mask, RBAC xem log, không sửa/xóa qua API, retention TBD, export redaction); (3) REQ-ADMIN-01/02 + 9.1 + AC-A1 (CSRF Origin/Referer allowlist + CSRF token nếu cần + logout server-side + BFF không open proxy + expiry/refresh TBD); (4) REQ-ADMIN-12 + 9.10 + 10.3 + 11.2 global_settings (public allowlist, cấm lộ internal/credential, secret không lưu bảng, validate type/schema + audit redact, namespace); (5) B13 + TBD-22 mới (/auth/register re-verify trong Survey, chưa thay đổi endpoint); (6) CSV export an toàn (REQ-ADMIN-08/09 + 9.7/9.8 + Greentech mapping: role được phép, không raw URL, audit, formula injection, retention TBD); (7) cập nhật Snapshot + Decision Log.
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (chỉ file này)
- Tests/checks run: Đếm lại bằng script — requirement vẫn 121 (không thêm requirement mới, chỉ mở rộng); TBD: 21 → 22 dòng (thêm TBD-22), mở (Awaiting) 21, đóng (Resolved) 1; không chạy build/test; không đọc .env/DB/cPanel; không sửa Greentech/source/docs/references
- Results: File nhất quán, không chứa secrets/PII
- Remaining issues: TBD còn 21 mở (mục 16, gồm TBD-22 register); chưa có phê duyệt handoff/Phase 1
- Exact next step: **Chờ chủ dự án review các hiệu chỉnh bảo mật + duyệt.** Sau duyệt: Checkpoint 1 (KHẢO SÁT CHỈ) — KHÔNG tự kết nối cPanel/SSH/DB khi chưa có phê duyệt riêng.

### Checkpoint 5 — Specification baseline approved (2026-08-14, chủ dự án phê duyệt)

- Completed: Chủ dự án phê duyệt `PROJECT_HANDOFF.md` làm specification baseline CHÍNH THỨC. Cập nhật Snapshot (`Overall status: Approved specification baseline`, `Current phase: Specification completed`, next action/blocker mới) + Decision Log (4 dòng: phê duyệt baseline; 121 requirements; 22 TBD = 21 mở + 1 resolved; mọi thay đổi yêu cầu phải cập nhật handoff + Decision Log). KHÔNG thay đổi nội dung requirement, không thêm implementation.
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (chỉ file này)
- Tests/checks run: Đếm lại — requirements 121 (không đổi); TBD 22 (21 mở, 1 resolved — không đổi); không chạy build/test; không đọc .env/DB/cPanel; không sửa Greentech/source/docs/references
- Results: File khớp baseline đã duyệt; không chứa secrets/PII
- Remaining issues: KHÔNG CÓ rào cản tài liệu. Phase 1 survey CHƯA được phê duyệt
- Exact next step: **Chờ chỉ thị phê duyệt RIÊNG cho Phase 1** — chỉ thị đó phải xác định rõ: (a) được đọc local source nào; (b) được kiểm kê website public hay không; (c) được xem thông tin cPanel nào; (d) AI thực hiện backup; (e) có được đọc schema-only/row counts hay không; (f) TUYỆT ĐỐI chưa được sửa production. KHÔNG tự bắt đầu bất kỳ khảo sát nào trước khi có chỉ thị đó.

### Checkpoint 6 — Resolved official hotline and legal canonical (2026-08-14, chủ dự án phê duyệt 2 quyết định)

- Completed: (1) ĐÓNG TBD-13 — hotline chính thức `+84 865.865.600` được chủ dự án xác nhận, được phép hiển thị public: cập nhật Rule 7 (mục 1), REQ-UX-03 (7.1), AC-F8 (15), TBD-13 (16 → Resolved); xóa mọi nhãn `CẦN XÁC MINH` gắn với số này; giá trị nghiệp vụ chuẩn giữ nguyên `+84 865.865.600` (chỉ trình bày UI được phép đổi format). (2) ĐÓNG TBD-19 — canonical trang pháp lý `/dieu-khoan`: cập nhật 5.6 (terms/privacy), REQ-UX-19 (7.1), 8.1 (URL strategy), 12 (SEO & URL Preservation), AC-S1 (15), TBD-19 (16 → Resolved); bỏ phương án `/thong-tin-phap-ly` khỏi mọi đặc tả hiện hành (giữ trong Decision Log là phương án đã cân nhắc nhưng không được chọn); mapping thống nhất `/chinh-sach-bao-mat` → 301 → `/dieu-khoan`; ghi nguyên tắc i18n canonical theo locale cần kiểm tra route hiện hữu khi triển khai. (3) Cập nhật Snapshot + Decision Log (2 dòng) + thống kê TBD.
- Files changed: `quangminh-smart-border/PROJECT_HANDOFF.md` (chỉ file này)
- Tests/checks run: Đếm lại — requirements 121 (không đổi requirement ID); TBD 22 (19 Awaiting + 3 Resolved: TBD-13, TBD-17, TBD-19); grep kiểm tra không còn cụm hotline "CẦN XÁC MINH" và không còn đặc tả hiện hành 2 phương án canonical; không chạy build/test; không đọc .env/DB/cPanel/Greentech; không sửa source/docs/references
- Results: File nhất quán, không chứa secrets/PII; các quyết định mới đã phản ánh ở mọi vị trí
- Remaining issues: 19 TBD mở (mục 16); Phase 1 survey CHƯA được phê duyệt
- Exact next step: **Chờ chỉ thị phê duyệt RIÊNG cho Phase 1** (phạm vi quyền: local source, website public, thông tin cPanel, ai backup, schema-only/row counts, tuyệt đối không sửa production). KHÔNG tự bắt đầu khảo sát.

### Checkpoint 7 — Phase 1A survey: local source + public website (2026-08-14, khảo sát read-only)

### Checkpoint 8 — Thực hiện Implementation Phases 2, 3, 4, 5 trên nhánh develop (2026-08-31)

- Completed:
  - **Git Branching Strategy**: Khởi tạo và chuyển sang nhánh `develop`. Giữ an toàn cho auto-deployment trên Vercel (Frontend) & Render (Backend).
  - **Phase 2 (Backend Core Fixes)**: Sửa triệt để lỗ hổng B14 Critical (`/careers/postings/:id` không load `applications`, bảo vệ riêng qua `/careers/postingss/:id` với `@UseGuards(JwtAuthGuard, RolesGuard)` và `@Roles(ADMIN, CONTENT_MANAGER)`); Sửa B2 runtime crash trong `dashboard.service.ts` bằng JOIN `item.category` và `cat.translations`; Sửa B5, B6, B12 trong `main.ts` (gỡ auto-migrate boot-time, siết CORS, cập nhật Swagger title); Sửa B7, B8 mật khẩu hashing và xóa plain text logging; Sửa B3, B4 Dockerfile và Compose env.
  - **Phase 3 (Backend CMS Modules & Additive Migration)**: Tạo 6 module NestJS mới (`AuditLogsModule` @Global, `SlidersModule`, `PartnersModule`, `CertificatesModule`, `GlobalSettingsModule`, `HealthModule`); Tạo TypeORM additive migration `1778000000000-AddCmsModulesAndAuditLogs.ts` (`CREATE TABLE IF NOT EXISTS` cho 6 bảng mới, tuyệt đối không drop/modify bảng cũ); Đăng ký đầy đủ trong `app.module.ts`.
  - **Phase 4 (Public Frontend Redesign & Brand Assets)**: Cập nhật màu nhận diện thương hiệu `#233871` Primary và `#0c9344` Secondary (loại bỏ dark mode theo yêu cầu); Tích hợp `LogoPreloader` hiệu ứng 1.2s SVG với cờ session storage và hỗ trợ `prefers-reduced-motion`; Cập nhật Header và Footer với Hotline chính thức `+84 865.865.600`, menu phân cấp 6 nhóm dịch vụ, link năng lực hạ tầng; Tích hợp 301 Redirects cho `/chinh-sach-bao-mat` -> `/dieu-khoan` trong `next.config.mjs`; Bổ sung CMS fetchers trong `data-fetchers.ts`.
  - **Phase 5 (Admin Panel BFF Proxy & RHF+Zod)**: Tạo Next.js BFF `middleware.ts` với `httpOnly` cookie bảo vệ toàn diện các route admin và proxy rewrite `/api-backend/*`; Tạo Auth route handlers (`/api/auth/login`, `/api/auth/logout`, `/api/auth/me`); Xây dựng bộ form wrapper RHF + Zod (`RHFInput`, `RHFSelect`, `RHFInputNumber`, `RHFSwitch`); Cập nhật giao diện Ant Design theme token `#233871`; Bổ sung menu `/consignments` vào Sidebar; Xây dựng đầy đủ các màn hình quản trị CRUD cho Sliders, Partners, Certificates, Global Settings, và Audit Logs.
- Files changed: Backend, Frontend, Admin Panel source files.
- Tests/checks run: All 3 apps passed `npm run build` with Exit code 0.

### Checkpoint 9 — Frontend Enterprise Redesign: Three.js 3D WebGL Digital Twin & Bright Spacious B2B Portal (2026-08-31)

- Completed:
  - **Three.js 3D WebGL Digital Twin Hub**: Xây dựng mô hình quả địa cầu kỹ thuật số tương tác (`SmartBorder3DHub.tsx`), mô phỏng trực quan các tuyến hành lang thương mại Tà Lùng — Thủy Khẩu — Hải Phòng — Nam Ninh với hiệu ứng hạt phân tử, đường cong Bezier phát sáng và gắn nhãn tương tác Raycasting khi hover.
  - **Spacious Full-Width Enterprise Header**: Thiết kế lại toàn diện Header theo tiêu chuẩn tập đoàn logistics quốc tế (Sotrans / Viettel Logistics). Tích hợp Top Utility Bar (Hotline 24/7 `+84 865.865.600`, trực chiến cửa khẩu, tra cứu vận đơn nhanh, bộ chuyển đổi ngôn ngữ) và thanh điều hướng chính 84px rộng rãi, menu dropdown 6 nhóm dịch vụ chi tiết và nút CTA "Yêu Cầu Báo Giá" góc cạnh sắc nét.
  - **Daylight Bright Theme & Crisp Rectilinear Architecture**: Loại bỏ hoàn toàn các nền tối u ám và bo góc tròn quá mức (loại bỏ kiểu dáng bubble/pill tròn trịa). Chuyển sang tông màu sáng ban ngày cao cấp (Pure White `#FFFFFF`, Soft Arctic Slate `#F8FAFC`, Navy `#233871` & Emerald `#0c9344`), border-radius chuẩn B2B doanh nghiệp (4px - 8px), đường viền sắc nét `1px solid #E2E8F0`.
  - **6 Core Logistics Services & 6-Step Enterprise Workflow**: Thiết kế lại các thẻ dịch vụ và quy trình thông quan 6 bước với danh sách tính năng cụ thể, liên kết chi tiết và báo giá trực tiếp.
  - **Instant Quotation & Live Tracking Widget**: Tích hợp widget tra cứu vận đơn đa phương tiện và form nhận báo giá B2B siêu tốc kết nối trực tiếp backend API.
- Files changed:
  - `frontend/src/components/3d/SmartBorder3DHub.tsx`
  - `frontend/src/components/layout/Header/index.tsx`
  - `frontend/src/components/layout/Footer/index.tsx`
  - `frontend/src/components/sections/HomePage/HeroSection.tsx`
  - `frontend/src/components/sections/HomePage/KpiSection.tsx`
  - `frontend/src/components/sections/HomePage/QuickTrackerAndQuoteWidget.tsx`
  - `frontend/src/components/sections/HomePage/WhyChooseUsSection.tsx`
  - `frontend/src/components/sections/HomePage/FeaturedServicesSection.tsx`
  - `frontend/src/components/sections/HomePage/ProcessSection.tsx`
  - `frontend/src/components/sections/HomePage/PartnersSection.tsx`
  - `frontend/src/app/[locale]/page.tsx`
  - `frontend/next.config.mjs`
- Tests/checks run:
  - `frontend`: `npm run build` -> Exit code 0 (thành công 100%).
  - Local Dev Servers: Frontend `http://localhost:3000` (200 OK), Backend `http://localhost:3005` (200 OK), Admin Panel `http://localhost:3002` (200 OK).
- Results: Toàn bộ giao diện sáng sủa, thoáng đãng, sắc nét, đúng nhận diện thương hiệu Tà Lùng Quang Minh Logistics.
- Next step: Sẵn sàng để chủ dự án trải nghiệm trực tiếp trên local và tiến hành merge `develop` -> `main`.
 
### Checkpoint 10 — Production Go-Live Deployment & Release Synchronization (2026-09-04)

- Completed:
  - **Comprehensive Production Build Verification**: Chạy kiểm thử build production trên toàn bộ 3 applications (`backend`, `frontend`, `admin-panel-frontend`) — Tất cả pass 100% với Exit code 0 (TypeScript types valid, Static HTML prerendering 26/26 routes frontend, 21/21 routes admin panel).
  - **CORS Enhancement for Production**: Cập nhật whitelist CORS trong `backend/src/main.ts` hỗ trợ dynamic domain `.vercel.app` và `.talunglogistics.com` subdomains.
  - **Git Main Branch Merge & Production Deployment Sync**:
    - **Backend** (`quangminh-smartborder-backend`): Fast-forward merge `develop` -> `main` & `git push origin main develop` (Kích hoạt auto-build & deploy trên Render).
    - **Admin Panel Frontend** (`phuanh-admin-panel-frontend`): Fast-forward merge `develop` -> `main` & `git push origin main develop` (Kích hoạt Vercel Production Deployment).
    - **Public Frontend** (`quangminh-smartborder-frontend-`): Fast-forward merge `develop` -> `main` & `git push origin main develop` (Kích hoạt Vercel Production Deployment).
    - **Root Superproject** (`phuanh-border-landing-page`): Fast-forward merge `develop` -> `main` & `git push origin main develop` đồng bộ toàn bộ submodule pointers.
  - **Production Smoke Test**: Kiểm tra response HTTP/2 200 từ `https://www.talunglogistics.com` xác nhận Vercel edge deployment đã tiếp nhận bản build mới thành công.
- Files changed: `backend/src/main.ts`, `PROJECT_HANDOFF.md`, Git submodules.
- Tests/checks run: `npm run build` trên cả 3 repositories, `git diff --check`, `curl -s -I https://www.talunglogistics.com`.
- Results: Hệ thống chính thức Go-Live thành công trên production.
- Next step: Giám sát vận hành, theo dõi telemetry/logging và hỗ trợ người dùng.

---

## 20. Source References

- Excel: `quangminh-smart-border/docs/references/Tổng hợp hoàn thiện website Tà Lùng-2.xlsx` (7 sheets: Tổng hợp, Web, Dịch vụ, Tuyển dụng, SEO tin tức, Checklist W01–W20, Fix Giao diện UI & UX) — nguồn chỉ thị nội dung/UI
- Video: `quangminh-smart-border/docs/references/Screen Recording 2026-08-03 150942.mp4` (~3,6s logo intro) — mô tả đã được chủ dự án cung cấp và chốt (mục 4.1)
- Logo: `quangminh-smart-border/docs/references/Logo/` (AI, PDF 22 trang, PNG 24 file, JPEG 18 file, namecard, Fonts: `1FTV-Nexa-Heavy.otf`, `SVN-Gotham Regular.otf`)
- Existing website: https://www.talunglogistics.com/ (VERIFIED live 2026-08-14: www → 307 canonical `www.talunglogistics.com` — non-www redirect sang www; homepage + toàn bộ route phổ biến 200; /quote → /lien-he; /dieu-khoan và /chinh-sach-bao-mat vẫn 200 riêng rẽ — CHƯA có 301 như quyết định TBD-19 (sẽ triển khai khi sửa); sitemap có 3 locale vi/en/zh + dịch vụ con; robots disallow /admin/, /api/, /_next/, /tracking/, /*?*)
- SOTRANS: https://sotransgroup.vn/ (URL do chủ dự án chỉ định, khảo sát live 2026-08-31) — Tham khảo: (1) Cấu trúc Hero banner Swiper toàn màn hình + logo preloader che trang; (2) Khối định vị thương hiệu "Từ năm 1975..." kết hợp grid ảnh bất đối xứng (tham khảo bố cục REQ-UX-06); (3) Bộ chọn ngôn ngữ tối giản trên header; (4) Menu dịch vụ phân cấp sâu.
- Viettel Logistics: https://viettellogistics.com.vn/vi (URL do chủ dự án chỉ định, khảo sát live 2026-08-31) — Tham khảo: (1) Thanh trượt chỉ số uy tín / KPI bar counter (+26 năm, 34 tỉnh thành, v.v. — tham khảo REQ-UX-07); (2) Hiệu ứng số nhảy (animated counter) trong phần "Về chúng tôi"; (3) Thẻ dịch vụ trực quan có sub-links trực tiếp dẫn vào "Tổng quan dịch vụ" và "Bảng giá #pricing" (mẫu lý tưởng cho REQ-UX-11 & REQ-SVC-*); (4) Tab chuyển đổi nghiệp vụ nhanh.
- Greentech (read-only reference): `greentech/greentech-backend`, `greentech/greentech-admin-panel`, `greentech/greentech-analysis`

## 21. Survey Findings — Phase 1A (2026-08-14)

| ID | Nguồn bằng chứng | Kết luận | Mức độ | Liên kết | Hành động đề xuất |
|---|---|---|---|---|---|
| SF-01 | `backend/src/careers/careers.controller.ts:91` + `admin-panel-frontend/src/app/(admin)/careers/_components/JobPostingsTab.tsx:23` | `/careers/postingss/` là route admin CHỦ ĐÍCH (Roles ADMIN/CONTENT_MANAGER), không phải typo; public dùng `/careers/postings` (controller:43-57) | Thông tin (giải tỏa) | REQ-BACKEND-03 (→ Approved verified), B1 (→ gỡ), 10.6 | Giữ nguyên; KHÔNG tạo alias |
| SF-02 | `backend/src/careers/careers.service.ts:84` (`relations: ['applications']`) + `careers.controller.ts:54-57` (public, không guard) | LỘ hồ sơ ứng viên: `GET /careers/postings/:id` public trả tên/email/phone/cvPath | Critical | B14 MỚI, Risks, REQ-BACKEND-02 | Sửa phase 1 fix: tách query public (không relations applications); audit; Decision Log đã ghi |
| SF-03 | `backend/src/auth/auth.controller.ts:14-20` | `/auth/register` KHÔNG guard, không rate limit; production chưa xác minh | Trung bình→Cao | TBD-22 (giữ Awaiting), B13, REQ-ADMIN-01 | Phase 1B: xác nhận production; nếu public → đề xuất khóa admin-only/setup-only |
| SF-04 | `backend/package.json` + `src/` (12 module) | Stack verified: NestJS ^11.0.1, TypeORM ^0.3.27, pg ^8.16.3; KHÔNG có `@nestjs/throttler`; KHÔNG health endpoint; KHÔNG audit log hiện tại | Thông tin | 5.1, REQ-BACKEND-07/08/09 | Giữ kế hoạch phase 5 bổ sung |
| SF-05 | `backend/src/dashboard/*.service*` (query service_category) | `/stats/categorical?metric=service_category` lỗi runtime vì cột `category` đã DROP | Cao | B2, REQ-BACKEND-04 | Fix query dùng categoryId |
| SF-06 | `backend/Dockerfile` | CMD `node dist/main` sai path (đúng dist/src/main); không copy `data-source.ts` | Cao | B3 | Fix phase 2 |
| SF-07 | `docker-compose.yml` + `src/app.module.ts` | Mismatch `DB_*` (compose) vs `DATABASE_URL` (app.module) | Cao | B4 | Fix phase 2 |
| SF-08 | `backend/src/main.ts:12-21` | Auto-migrate khi boot; khi fail chỉ console.error (không exit) | Cao | B5, REQ-BACKEND-05 | Gỡ auto-migrate, chuyển thủ công |
| SF-09 | `backend/src/main.ts:23-37` | CORS mở mọi origin khi `NODE_ENV !== 'production'`; fallback origins | Trung bình | B6 | Siết production |
| SF-10 | `backend/src/users/entities/user.entity.ts:50-60` + `users.service.ts` | `@BeforeInsert` hash OK; `@BeforeUpdate` chỉ hash khi có password; super admin id=1; password tạm console.log (verify còn) | Trung bình | B7, B8, REQ-BACKEND-14/15 | Sửa theo kế hoạch |
| SF-11 | `admin-panel-frontend/src/stores/authStore.ts` + `src/lib/axios.ts` + layout | Token localStorage (persist `auth-storage`), axios interceptor Bearer; bảo vệ route chỉ client-side, KHÔNG middleware | Cao | B10, REQ-ADMIN-01/02 | BFF proxy + httpOnly phase 4 |
| SF-12 | `admin-panel-frontend/package.json` | next-intl ^4.6.1 khai báo nhưng KHÔNG dùng (UI hardcode tiếng Việt) | Thông tin | 5.2 | Không thay đổi kế hoạch |
| SF-13 | `frontend/package.json` + `src/app/[locale]/` + `navigation.ts` | Verified: Next 15.5.9 pin, React 19.1, next-intl 4.3 (vi/en/zh), 14 pathname + `[...rest]`, sitemap/robots/JSON-LD/hreflang/GTM-5BD4XCML | Thông tin | 5.3, REQ-SEO-* | Khớp baseline |
| SF-14 | `frontend/src/lib/api*.ts`/fetchers + `next.config.mjs` | axios `NEXT_PUBLIC_API_URL` trực tiếp (KHÔNG proxy); standalone + turbopack | Thông tin | 5.3 | Khớp baseline |
| SF-15 | GET https://www.talunglogistics.com/ (307) + https://talunglogistics.com/ (→ www) | Canonical = `www.talunglogistics.com`; non-www → 307 (redirect duy nhất); triển khai cf + Vercel headers | Thông tin | 20 (Source References), 8.1 | Giữ nguyên; baseline sitemap dùng `talunglogistics.com` (loc gốc) |
| SF-16 | GET ~25 URL public | Toàn bộ route phổ biến 200: /, /gioi-thieu, /dich-vu, /tin-tuc, /tuyen-dung, /lien-he, /tra-cuu, /tuyen-ngon, /dieu-khoan, /chinh-sach-bao-mat, /nang-luc-ha-tang, /robots.txt; `/quote` → `/lien-he` (200, redirect hoạt động) | Thông tin | 5.3, 8.1 | Khớp kế hoạch |
| SF-17 | GET /dieu-khoan, /chinh-sach-bao-mat | Cả 2 trả 200 riêng rẽ — CHƯA có 301 như quyết định TBD-19 (đúng, chưa triển khai) | Thông tin | TBD-19 (Resolved), REQ-UX-19 | Triển khai redirect khi sửa trang (phase 2) |
| SF-18 | GET /sitemap.xml | Sitemap tồn tại, 3 locale vi/en/zh + hreflang; dịch vụ con: đại-lý-hải-quan, van-tai, sang-tai-luu-do, kho-bai-ta-lung; 10+ bài tin tức | Thông tin | 12 (SEO), 5.3 | Khớp baseline (sitemap không gồm terms/privacy/tracking/infrastructure — xem xét khi triển khai) |
| SF-19 | GET /robots.txt | Disallow /admin/, /api/, /_next/, /tracking/, /*?*; sitemap trỏ `talunglogistics.com` | Thông tin | 12 | Khớp baseline |
| SF-20 | `greentech/greentech-backend/src/modules/*/` | Naming controller thực tế: `@Controller('admin/...')` trong `<module>.controller.ts` (vd news.controller.ts:31-32) + `<module>.public.controller.ts` (vd news.public.controller.ts:7) — KHÔNG có `*.admin.controller.ts` | Thông tin | 6 (mapping — đã sửa) | Đã cập nhật mapping + Decision Log |
| SF-21 | `greentech/greentech-backend/src/common/filters/http-exception.filter.ts:11-67` + `common/interceptors/transform.interceptor.ts:27-46` | AllExceptionsFilter map Prisma P2025→404, P2002→409, 429, validation; output `{success,statusCode,errorCode,message,path,timestamp}`; TransformInterceptor | Thông tin | 6 (REQ-BACKEND-08/09) | Viết lại theo TypeORM (23505/22P02), đã ghi mapping |
| SF-22 | `greentech/greentech-backend/src/modules/audit-logs/` (module.ts:7,12 @Global) + `prisma/schema.prisma:47-61` (`AuditLog @@map("audit_logs")`) + `audit-logs.service.ts:13-47` | AuditLogsModule @Global verified | Thông tin | 6, REQ-BACKEND-08 | Reference hợp lệ cho phase 5 |
| SF-23 | Greentech backend KHÔNG có `.env.example` (chỉ `.env`/`.env.local` — KHÔNG đọc nội dung) | Không thể đối chiếu env keys từ repo; giữ TBD-07 / Phase 1B | Thông tin | TBD-07 | Phase 1B xác nhận qua cPanel |

## 22. Checkpoint 11 — Tắt/Ẩn Toàn Bộ Hiển Thị Tra Cứu Vận Đơn & Nâng Cấp Báo Giá Nhanh B2B (2026-09-12)

Theo yêu cầu trực tiếp từ khách hàng ("hiện tại chưa có phần vận đơn nên ẩn giúp tôi"), toàn bộ các điểm chạm hiển thị công khai liên quan đến **Tra Cứu Vận Đơn / AWB Tracking** đã được ẩn và điều chỉnh:

1. **Header Navigation & TopBar**:
   - Gỡ link `Tra Cứu Vận Đơn` trong TopBar tiện ích bên phải.
   - Bỏ `Tra Cứu` khỏi thanh điều hướng chính (`NavWingRight`), cân đối bố cục hoàn hảo giữa cánh trái (Trang chủ, Giới thiệu, Dịch vụ, Năng lực 25ha) và cánh phải (Tin tức, Tuyển dụng, Liên hệ, nút CTA Báo giá).
   - Gỡ mục `Tra Cứu Vận Đơn` trong Mobile Drawer.
2. **Hero Slider Banner**:
   - Slide 2: Đổi nút phụ từ `Tra Cứu Vận Đơn` sang `Dịch Vụ Hải Quan` trỏ trực tiếp đến `/services/dich-vu-dai-ly-hai-quan`.
3. **Widget Trang Chủ (QuickTrackerAndQuoteWidget)**:
   - Loại bỏ hoàn toàn tab và form tra cứu mã vận đơn/biển số xe.
   - Nâng cấp thành widget chuyên biệt: **"Báo Giá Nhanh B2B & Tư Vấn Cửa Khẩu Tà Lùng 24/7"**.
   - Thiết kế dạng thẻ bo tròn cao cấp, gồm: Họ tên/Doanh nghiệp, Số điện thoại/Zalo, Email liên hệ, Dropdown chọn mặt hàng/nhu cầu, cùng nút CTA "Nhận Báo Giá".
   - Tự động gửi dữ liệu về API `POST /quotes` trên backend với validation hợp lệ và hiển thị thông báo gửi thành công tức thì.
4. **Footer**:
   - Thay thế link `Tra Cứu Vận Đơn` bằng `Yêu Cầu Báo Giá & Tư Vấn` trỏ về `/contact`.
5. **Kiểm thử & Đồng bộ Git**:
   - Build production `npm run build` thành công 100% (26/26 routes).
   - Xác minh hiển thị trực quan qua browser headless screenshot (`b2b_quick_quote_widget_1789204245571.png`).
   - Đã commit và đồng bộ lên branch `develop` & `main` của repo `frontend` và superproject.

## 23. Checkpoint 12 — Tái Cấu Trúc Header Doanh Nghiệp Chuẩn, Thống Nhất Ảnh Dịch Vụ 16:9, Bổ Sung Khối Tin Tức Trang Chủ & Tối Giản Luồng Liên Hệ (2026-09-12)

Đáp ứng đầy đủ các yêu cầu điều chỉnh giao diện trực quan từ chủ sở hữu:

1. **Tối Giản Luồng Liên Hệ & Bỏ Widget Báo Giá B2B Ngoài Trang Chủ**:
   - Khách hàng xác nhận không cần form/widget báo giá nhanh phức tạp ngoài trang chủ ("Dịch vụ / Mặt hàng quan tâm, bỏ báo giá B2B nhanh này đi, chỉ giữ lại cái liên hệ là ok r").
   - Gỡ bỏ hoàn toàn `QuickTrackerAndQuoteWidget` trên trang chủ, tập trung chuyển đổi vào nút CTA `Liên Hệ` trỏ trực tiếp về trang liên hệ chính thức `/contact` (kèm Hotline +84 865.865.600 và Zalo/Email tiện ích).
2. **Tái Thiết Kế Thanh Header Doanh Nghiệp (Khắc Phục Hoàn Toàn Lỗi Bị Kéo Dãn / "Nhễ Ra")**:
   - Loại bỏ cơ chế chia 2 cánh đối xứng trước đây (vốn đẩy menu dạt về hai rìa ngoài cùng và tạo khoảng trống lớn ở giữa).
   - Thiết lập cấu trúc chuẩn nhận diện B2B:
     - **Bên Trái**: Logo chính thức đầy đủ màu sắc thương hiệu `/images/logo-01.png` (Biểu tượng núi & xe kết hợp chữ "TÀ LÙNG QUANG MINH LOGISTICS").
     - **Ở Giữa / Phải**: Cụm menu điều hướng thống nhất, khoảng cách nhịp nhàng (Trang chủ, Giới Thiệu, Dịch Vụ kèm dropdown 6 dịch vụ, Năng Lực 25ha, Tin Tức, Tuyển Dụng).
     - **Bên Phải**: Nút CTA nổi bật xanh lá `#0c9344` với nội dung `LIÊN HỆ →`.
     - **TopBar Tiện Ích**: Căn chỉnh độ rộng 1440px đồng bộ, hiển thị địa chỉ Cửa khẩu Tà Lùng, trực chiến 24/7, hotline và bộ chuyển ngôn ngữ (VI/EN/ZH).
3. **Thống Nhất Kích Thước Ảnh Khối 6 Nhóm Dịch Vụ Theo Tỷ Lệ Chuẩn Landscape 16:9**:
   - Cập nhật khung hình thẻ dịch vụ tại `FeaturedServicesSection.tsx` sử dụng thuộc tính CSS `aspect-ratio: 16 / 9; object-fit: cover;`.
   - 6 thẻ dịch vụ (Đại lý hải quan, Kho bãi 25ha, Sang tải cơ giới, Vận tải quốc tế, Bến xe điều phối, Logistics trọn gói) đạt độ đồng đều 100%, sắc nét và chuẩn bố cục truyền thông.
4. **Bổ Sung Khối Tin Tức Ra Ngoài Trang Chủ (`LatestNewsSection`)**:
   - Đưa phân mục Tin Tức & Hoạt Động Cửa Khẩu Tà Lùng lên vị trí trang trọng ngoài trang chủ (ngay sau Quy trình làm việc 4 bước).
   - Thiết kế giao diện daylight sáng sủa, thẻ tin tức chuẩn tỷ lệ landscape 16:9, kèm huy hiệu phân loại (Chính Sách & Cửa Khẩu, Tin Tức & Sự Kiện, Nghiệp Vụ Logistics), ngày đăng và thời gian cập nhật 24/7.
   - Bổ sung dữ liệu dự phòng (fallback) từ các bài viết thực tế trên website `talunglogistics.com` để không bao giờ bị rỗng hoặc nhấp nháy khi tải.
5. **Tối Ưu Hiệu Ứng Chuyển Đổi Banner Hero Slider**:
   - Đưa khung ảnh visual banner về chuẩn 16:9 (`aspect-ratio: 16 / 9`).
   - Cấu hình `AnimatePresence initial={false}` chuyển đổi crossfade mượt mà, loại bỏ triệt để hiện tượng nhấp nháy khung xám giữa các slide.
6. **Kiểm Thử & Build Production**:
   - Next.js 15.5.9 production build (`npm run build`) hoàn thành thành công 100% với 26/26 routes (0 lỗi, 0 cảnh báo lint).
   - Kiểm tra trực quan bằng subagent headless browser đạt độ hoàn mỹ cao trên mọi phân đoạn (`header_hero_top_view_1789205979197.png`, `featured_services_section_view_1789205763528.png`, `latest_news_section_centered_1789205796582.png`).

## 24. Checkpoint 13 — Tích Hợp Tìm Kiếm Toàn Diện (Search Modal) & Nâng Tầm Thẩm Mỹ Header Đẳng Cấp Doanh Nghiệp (2026-09-12)

Khắc phục phản hồi của khách hàng ("Header giờ lại quá cơ bản, còn mất phần tìm kiếm của tôi nữa chứ"):

1. **Khôi Phục & Nâng Cấp Tính Năng Tìm Kiếm Tức Thì (Instant Search)**:
   - Tích hợp nút tìm kiếm dạng capsule hiện đại trên thanh điều hướng chính (`[ 🔍 Tìm kiếm...  ⌘K ]`) và link tra cứu nhanh trên TopBar.
   - Hỗ trợ phím tắt toàn cục `Cmd+K` / `Ctrl+K`: Người dùng bấm phím tắt ở bất cứ đâu trên trang đều mở ngay hộp tìm kiếm.
   - Tích hợp `SearchModal`: Tìm kiếm real-time với backend API, nhóm kết quả rõ ràng theo **Dịch Vụ**, **Tin Tức Thị Trường** và **Tuyển Dụng** kèm icon và điều hướng chuẩn SEO.
   - Bổ sung ô tìm kiếm trong cả Mobile Drawer trên thiết bị di động.
2. **Nâng Tầm Thẩm Mỹ Header (Rich Enterprise Aesthetics)**:
   - **Logo & Nhận Diện**: Thêm vạch phân cách tinh tế cùng khối nhận diện năng lực doanh nghiệp (*TỔ HỢP 25HA / CỬA KHẨU QUỐC TẾ TÀ LÙNG*), tạo cảm giác uy tín và quy mô lớn.
   - **TopBar Tiện Ích**: Nền chuyển sắc Midnight Navy cao cấp, bổ sung hải đăng xanh nhấp nháy phát quang (`● Trực chiến hải quan 24/7`), Hotline dạng pill tag sắc nét.
   - **Menu Điều Hướng**: Hiển thị thẻ nổi bật `25HA` bên cạnh Năng Lực 25ha, dropdown Megamenu 2 cột với icon nổi trong nền gradient cho 6 dịch vụ và chân trang tư vấn trực tiếp 24/7.
   - **Hiệu Ứng Nút CTA**: Nút `LIÊN HỆ` với dải ánh kim (shimmer animation) khi hover, bo góc chuẩn, đổ bóng phát sáng xanh lá.
   - **Hiệu Ứng Cuộn**: Mặt kính mờ (Frosted Glass) `backdrop-filter: blur(16px)` khi cuộn trang, tự động thu nhỏ độ cao từ 82px về 74px nhịp nhàng.
3. **Kiểm Thử & Xác Nhận**:
   - Production build `npm run build` hoàn thành 100% không lỗi (26/26 routes, 0 warning).
   - Kiểm tra headless browser xác nhận tương tác mở Search Modal, gõ từ khóa và hiển thị kết quả thành công rực rỡ (`rich_header_top_1789206699726.png`, `search_modal_results_1789206803139.png`).
## 25. Checkpoint 14 — Nâng Cấp Logo Header & Nhãn Thương Hiệu Chính Thức (`logo-single-header.png` & `Lable-header.png`) (2026-09-12)

Thực hiện yêu cầu của khách hàng ("Chỗ TỔ HỢP 25HA CỬA KHẨU QUỐC TẾ TÀ LÙNG cho thành cái ảnh Lable-header.png và logo header thì dùng logo-single-header.png"):

1. **Cập Nhật Tài Nguyên Đồ Họa Header**:
   - Tích hợp biểu tượng logo đơn lẻ chính thức `/images/logo-single-header.png` (kích thước gốc 1892x1521, tỷ lệ ~1.24:1) vào vị trí biểu trưng thương hiệu trên Header.
   - Thay thế văn bản `TỔ HỢP 25HA / CỬA KHẨU QUỐC TẾ TÀ LÙNG` bằng dải hình ảnh đồ họa thiết kế `/images/Lable-header.png` (kích thước gốc 2808x609, tỷ lệ ~4.6:1).
2. **Căn Chỉnh Tỷ Lệ & Bố Cục Thẩm Mỹ (Desktop & Mobile Drawer)**:
   - Tinh chỉnh `LogoWrapper` với khoảng cách `gap: 12px`, vạch hairline ngăn cách màu xám nhạt sang trọng `#E2E8F0`.
   - `logo-single-header.png` hiển thị với chiều cao chuẩn `50px` (chiều rộng tỷ lệ tương ứng ~`62px`), tối ưu độ sắc nét trên màn hình Retina (High-DPI).
   - `Lable-header.png` hiển thị với chiều cao chuẩn `38px` (chiều rộng tương ứng ~`185px`), căn giữa theo trục đứng, đồng bộ nhịp nhàng cùng biểu tượng logo và thanh menu điều hướng.
   - Thêm hiệu ứng tương tác vi mô: `hover` phóng nhẹ logo (`scale(1.04)`) và nhãn (`scale(1.02)`).
   - Đồng bộ hiển thị cặp logo và nhãn trong thanh tiêu đề `DrawerHeader` của Mobile Drawer trên thiết bị di động (chiều cao 30px).
3. **Kiểm Thử Trực Quan (Browser Verification)**:
   - Xác nhận trên môi trường thực tế tại `http://localhost:3000/`.
   - Ảnh chụp giao diện thực tế ghi nhận tải thành công HTTP 200, hình ảnh sắc nét, không bị kéo dãn méo hình, căn lề hoàn hảo bên cạnh thanh tìm kiếm `⌘K` và menu điều hướng (`header_new_logo_label_1789208303772.png`).

## 26. Checkpoint 15 — Tinh Gọn TopBar & Menu Điều Hướng ("Năng Lực" & Giữ Nút Tìm Kiếm ⌘K) (2026-09-12)

Thực hiện yêu cầu của khách hàng ("bỏ chỗ trực chiến 24/7, 25ha ở năng lực chỉ ghi năng lực thôi, chỉ giữ lại tìm kiếm bỏ tra cứu"):

1. **Tinh Gọn TopBar Tiện Ích**:
   - Loại bỏ huy hiệu `Trực chiến hải quan 24/7` và hiệu ứng xung nhịp `pulseGlow`.
   - Loại bỏ nút `Tra cứu ⌘K` ở góc phải TopBar.
   - Giữ lại đầy đủ: Địa chỉ Cửa khẩu Quốc tế Tà Lùng, Phục Hòa, Cao Bằng; Hotline `+84 865.865.600`; Bộ chuyển ngôn ngữ `LanguageSwitcher` (`🇻🇳 VI`).
2. **Chuẩn Hóa Menu "Năng Lực" (Desktop & Mobile Drawer)**:
   - Thay thế liên kết `Năng Lực 25ha [25HA]` thành chỉ **`Năng Lực`** trên thanh điều hướng chính và Mobile Drawer.
   - Loại bỏ thẻ badge phụ `25HA`.
   - Bổ sung khóa dịch đa ngôn ngữ i18n: `infrastructure: "Năng Lực"` (VI), `"Capacity"` (EN), `"企业实力"` (ZH).
3. **Giữ Nguyên Nút Tìm Kiếm Tức Thì (Instant Search ⌘K)**:
   - Duy trì ô tìm kiếm viên thuốc hiện đại `[ 🔍 Tìm kiếm...  ⌘K ]` ở thanh điều hướng chính cạnh nút `LIÊN HỆ ->`.
4. **Kiểm Thử Trực Quan & Build**:
   - `npx tsc --noEmit` đạt 0 lỗi.
   - Kiểm tra headless browser xác nhận giao diện mới tinh gọn, sắc sảo, nút tìm kiếm mở Search Modal bình thường (`header_updated_verification_1789208893946.png`, `search_modal_open_1789208913570.png`).

## 27. Checkpoint 16 — Nâng Cấp Hiệu Ứng Mở Đầu (Logo Bay Từ Trái, Nhãn Bay Từ Phải, Quét Sáng & Phụ Đề Chuẩn) (2026-09-12)

Thực hiện yêu cầu của khách hàng ("Logo single thì đi từ bên trái sang, còn label từ bên phải sang, cho thêm hiệu ứng đi chứ", "k nên để tổ hợp 25ha ở chỗ đó mà ghi là trung tâm logistics tại cửa khẩu quốc tế tà lùng"):

1. **Hiệu Ứng Hội Tụ Điện Ảnh (Cinematic Convergence)**:
   - **Logo Single (`logo-single-header.png`)**: Bay mượt mà từ bên trái vào tâm (`translateX(-140px)` -> `0`), kết hợp làm mờ và thu phóng `scale(0.82)` -> `1.0`.
   - **Nhãn Thương Hiệu (`Lable-header.png`)**: Bay từ bên phải vào tâm (`translateX(140px)` -> `0`).
   - Cả 2 phần tử gặp nhau tại tâm đồng thời với đường cong giảm tốc mượt mà `cubic-bezier(0.16, 1, 0.3, 1)`.
2. **Hiệu Ứng Quét Sáng Ánh Kim & Vầng Hào Quang**:
   - Khi 2 phần tử hội tụ tại tâm (~0.8s), một chùm tia sáng ánh kim (`lightSweep`) quét chéo qua toàn bộ khối logo từ trái sang phải.
   - Vòng tròn sóng xung kích (`ringRipple`) lan tỏa rộng cùng vầng hào quang chuyển sắc mềm mại (`haloGlow`).
   - Thanh tiến trình nạp (progress bar) 240px bo tròn với dải màu thương hiệu `#233871` -> `#0c9344` nạp đầy dần nhịp nhàng.
3. **Chuẩn Hóa Phụ Đề Thương Hiệu**:
   - Đổi phụ đề từ "Tổ hợp 25ha" thành: **`Trung tâm Logistics tại Cửa khẩu Quốc tế Tà Lùng`** với kiểu chữ dãn cách sang trọng `letter-spacing: 3px`.
   - Đồng bộ alt text của nhãn thương hiệu trên Header thành `Trung tâm Logistics tại Cửa khẩu Quốc tế Tà Lùng`.
4. **Kích Hoạt Trên Mọi Lần Tải Trang**:
   - Gỡ bỏ giới hạn `sessionStorage`, preloader luôn hiển thị sống động mỗi khi người dùng tải lại trang (F5 / reload).
5. **Kiểm Thử Trực Quan**:
   - Đã kiểm tra qua headless browser và chụp ảnh giao diện thực tế (`preloader_screen_1789209681169.png`), hiệu ứng bay từ hai phía và quét sáng vô cùng cuốn hút.

## 28. Checkpoint 17 — Nâng Cấp Tùy Chọn Icon Dịch Vụ Admin Panel & Tiêu Đề Động Linh Hoạt (2026-09-12)

Thực hiện yêu cầu của khách hàng ("phần này là cố định 6 cái à? sau này k thêm đc à? những chỗ icon này ng dùng k set đc à? chốt cách 1 đi"):

1. **Cơ Sở Dữ Liệu & Backend**:
   - Thêm cột `icon VARCHAR(50)` vào bảng `services` trong PostgreSQL.
   - Bổ sung trường `icon` vào `Service` entity (`backend/src/services/entities/service.entity.ts`).
   - Cập nhật `CreateServiceDto` và `UpdateServiceDto` hỗ trợ validation và lưu trữ `icon`.
   - Cập nhật dữ liệu mặc định ban đầu cho 6 dịch vụ cốt lõi: `shield`, `exchange`, `warehouse`, `truck`, `bus`, `global`.
2. **Trang Quản Trị Admin (`admin-panel-frontend`)**:
   - **Form Dịch Vụ (`ServiceFormDrawer.tsx`)**: Tích hợp bộ chọn `Select` **"Icon biểu trưng"** với 17 biểu tượng Logistics thông dụng (Hải quan 🛡️, Kho bãi 🏢, Xe container 🚛, Sang tải 🔄, Bến xe 🚌, Chuỗi cung ứng 🌐, Lưu CFS 📦, Cẩu hàng ⚓, Đường biển 🚢, Hàng không ✈️, Trạm cân ⚖️, C/O Form E 🏆, Cửa khẩu 🏛️, An ninh 🔒, Kho lạnh ❄️, Siêu tốc ⚡, GPS 🗺️).
   - **Bảng Danh Sách (`ServicesPage`)**: Bổ sung cột **Icon** hiển thị trực quan biểu tượng tương ứng của từng dịch vụ.
3. **Giao Diện Trang Chủ (`frontend`)**:
   - **Tiêu Đề Động**: Tự động đếm theo số lượng dịch vụ thực tế từ API (`{services.length} Nhóm Dịch Vụ Toàn Diện Tại Cửa Khẩu Tà Lùng`), không còn bị cứng nhắc số 6 khi người dùng thêm/bớt dịch vụ trong Admin.
   - **Render Icon Chuẩn**: Xây dựng tiện ích `getServiceIcon(iconKey)` liên kết chuỗi `icon` từ Backend API sang bộ icon vector chất lượng cao của `react-icons/ri` với fallback thông minh.
4. **Kiểm Thử & Đồng Bộ Git**:
   - TypeScript kiểm tra 0 lỗi trên cả `backend`, `admin-panel-frontend`, và `frontend`.
   - Kiểm tra trình duyệt thực tế ghi nhận 6 thẻ dịch vụ hiển thị đầy đủ icon riêng biệt sắc nét và tiêu đề tự động cập nhật (`featured_services_full_1789210697094.png`).

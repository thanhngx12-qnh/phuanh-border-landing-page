# HỆ THỐNG THIẾT KẾ DOANH NGHIỆP (ENTERPRISE DESIGN SYSTEM SPECIFICATION)
## DỰ ÁN: TÀ LÙNG QUANG MINH LOGISTICS
**Phiên bản:** 1.0 (Trích xuất & Chuẩn hóa từ Trang Chủ - Tháng 09/2026)  
**Tác giả:** Antigravity Architecture & Design Team  
**Mục đích:** Bản đặc tả kiến trúc UI/UX và System Design đóng vai trò là kim chỉ nam (Single Source of Truth) để đồng bộ hóa giao diện và trải nghiệm người dùng cho toàn bộ các trang con (`/gioi-thieu`, `/dich-vu`, `/nang-luc-ha-tang`, `/tin-tuc`, `/tuyen-dung`, `/lien-he`, chi tiết bài viết, v.v.).

---

## I. NGUYÊN TẮC CỐT LÕI (CORE DESIGN PRINCIPLES)

1. **B2B Logistics Authority (Vị thế Tập đoàn Logistics Biên mậu Quốc gia):**
   - Thiết kế toát lên sự vững chắc, hiện đại, quy chuẩn và minh bạch của một tổ hợp logistics biên giới hàng đầu Việt - Trung.
   - Tránh phong cách rườm rà, đồ họa kiểu tiêu dùng (B2C) giá rẻ. Mọi đường nét, thẻ card và bố cục đều phải sắc sảo, dứt khoát và mang tính công nghiệp cao.

2. **Chế độ hiển thị: STRICT LIGHT MODE (Nghiêm cấm Dark Mode):**
   - Theo quyết định cốt lõi **REQ-BRAND-03**, toàn bộ hệ thống web chỉ hoạt động trên giao diện sáng cao cấp (Crisp Clean Light Theme).
   - Nền trắng ngọc trai (`#FFFFFF`) phối hợp cùng sắc xám nhạt cao cấp (`#F8FAFC`, `#F1F5F9`) tạo chiều sâu thị giác tự nhiên.

3. **Chuẩn tỷ lệ hình ảnh nghiêm ngặt: STRICT 16:9 ASPECT RATIO:**
   - Toàn bộ media cards (Dịch vụ, Tin tức, Banner trang con, Thư viện hình ảnh) **BẮT BUỘC tuân thủ tỷ lệ chuẩn 16:9** (`aspect-ratio: 16 / 9; object-fit: cover;`).
   - Ngăn chặn triệt để tình trạng vỡ layout, ảnh méo mó hoặc không đều chiều cao giữa các thẻ cạnh nhau.

4. **Nhịp điệu khoảng cách & Khung lưới chuẩn 1440px (Rhythm & Grid):**
   - Khung container tối đa: `max-width: 1440px; margin: 0 auto; padding: 0 24px;`.
   - Nhịp điệu section: `padding: 90px 24px;` (Desktop), `padding: 60px 16px;` (Mobile).
   - Phân tách section: Đan xen giữa nền Trắng tuyết (`#FFFFFF`) và nền Slate 50 (`#F8FAFC`), ngăn cách bằng đường kẻ viền thanh mảnh `1px solid #E2E8F0`.

5. **Chuyển động vi mô chuẩn công nghệ (Micro-Interactions):**
   - Timing function: `cubic-bezier(0.16, 1, 0.3, 1)` cho tất cả hiệu ứng hover, drawer, modal.
   - Thẻ Card hover: Nâng nhẹ `transform: translateY(-6px);`, bóng đổ sâu `box-shadow: 0 16px 36px rgba(35, 56, 113, 0.12);`, ảnh zoom `scale(1.06)`.
   - Nút hành động chính: Hiệu ứng quét sáng `shimmer` góc nghiêng 45 độ ánh bạc sang trọng.

---

## II. BẢNG MÃ MÀU & COLOR TOKENS THƯƠNG HIỆU

| Token Name | Hex Code / Value | Vai trò & Quy định sử dụng |
| :--- | :--- | :--- |
| `--color-primary` | `#233871` | **Navy Blue Thương hiệu:** Tiêu đề H1-H3, nút hành động thứ cấp, viền active, icon badge mặc định, chân trang. |
| `--color-primary-dark` | `#1A2B56` / `#0F1D36` | **Deep Navy:** Nền TopBar, Footer nền sâu, nền các box công nghệ cao. |
| `--color-primary-light`| `#2F4B96` | Màu hover của các phần tử Navy Blue. |
| `--color-accent` | `#0c9344` | **Emerald Green Nhận diện:** Màu điểm nhấn thương hiệu (KHÔNG dùng `#0b9444`). Dùng cho Category Eyebrow, nút Báo giá B2B, icon check, trạng thái thành công, vạch chỉ báo menu active. |
| `--color-accent-hover`| `#097034` | Trạng thái hover cho các thành phần màu Emerald Green. |
| `--color-accent-soft` | `#ECFDF5` / `rgba(12,147,68,0.08)` | Nền pill badge, nền tag chuyên mục, viền phụ trạng thái active. |
| `--color-bg-main` | `#FFFFFF` | Nền trắng chính của toàn bộ trang và thân thẻ card. |
| `--color-bg-surface` | `#F8FAFC` | Nền phụ cho các section đan xen, bảng thông số, thẻ đặc tính nổi bật. |
| `--color-border` | `#E2E8F0` | Đường kẻ phân tách chuẩn (Hairline divider), viền thẻ card, viền input. |
| `--color-border-hover`| `#233871` | Viền thẻ card khi hover hoặc input khi focus. |
| `--color-text-title` | `#0F172A` / `#1E293B` | Màu chữ tiêu đề đậm, tương phản cực cao trên nền sáng. |
| `--color-text-body` | `#64748B` | Màu chữ nội dung mô tả, phụ đề, thời gian (Slate 500). |
| `--color-text-muted` | `#94A3B8` | Chữ gợi ý mờ, placeholder, nhãn thứ cấp. |

---

## III. TYPOGRAPHY SYSTEM (HỆ PHÔNG CHỮ & PHÂN CẤP NỘI DUNG)

### 1. Cấu hình Phông chữ
- **Tiêu đề & Nhận diện (Heading Font):** `Montserrat`, `Nexa Heavy`, sans-serif (Weights: `700`, `800`, `900`).
- **Nội dung & Dữ liệu (Body Font):** `Inter`, `SVN-Gotham Regular`, -apple-system, sans-serif (Weights: `400`, `500`, `600`, `700`).

### 2. Phân cấp cỡ chữ (Type Scales)

| Cấp bậc | Cỡ chữ / Line Height | Font Weight & Màu sắc | Vị trí sử dụng |
| :--- | :--- | :--- | :--- |
| **Eyebrow / Category** | `13px` / `1.2` | `800` • `#0c9344` • Uppercase • Spacing: `1.5px` | Nhãn phân loại trên đầu mỗi Section Title |
| **Page Hero H1 (Trang con)**| `clamp(32px, 4.5vw, 48px)` / `1.2` | `900` • `#FFFFFF` hoặc `#233871` | Tiêu đề chính trên Banner đầu các trang con |
| **Section Title H2** | `clamp(24px, 3.2vw, 38px)` / `1.25` | `900` • `#233871` | Tiêu đề các khối nội dung lớn trên trang |
| **Card Title H3** | `18px - 19px` / `1.35` | `800` • `#1E293B` (Hover: `#233871` hoặc `#0c9344`) | Tiêu đề thẻ Dịch vụ, Tin tức, Năng lực |
| **Subhead / Lead Text**| `15.5px - 16.5px` / `1.6` | `500` • `#64748B` | Đoạn tóm tắt giới thiệu ngay dưới H2 |
| **Body Text** | `14px - 15px` / `1.65` | `400` • `#334155` | Đoạn văn chi tiết bài viết, mô tả sản phẩm |
| **Meta / Caption** | `12px - 13px` / `1.4` | `600` • `#64748B` | Ngày đăng, chuyên mục, hotline, thời lượng |

---

## IV. BỐ CỤC KHUNG LƯỚI & QUY CHUẨN KHOẢNG CÁCH (LAYOUT & GRID)

```
+-------------------------------------------------------------------------+
| TopBar: Max 1440px | Địa chỉ & Hotline Pill (Left) | Ngôn ngữ 🇻🇳 (Right) |
+-------------------------------------------------------------------------+
| MainNav: Max 1440px | Logo Kép | Nav Links | [🔍 Tìm kiếm ⌘K] | [LIÊN HỆ ->]|
+-------------------------------------------------------------------------+
| Hero Banner (Trang chủ hoặc Banner trang con với Breadcrumbs)           |
+-------------------------------------------------------------------------+
| Section 1: Nền #FFFFFF  | Container: Max 1440px | Padding: 90px 24px     |
+-------------------------------------------------------------------------+
| Section 2: Nền #F8FAFC  | Container: Max 1440px | Padding: 90px 24px     |
+-------------------------------------------------------------------------+
| Footer: Nền #1A2B56     | Container: Max 1440px | 4 Cột Doanh nghiệp     |
+-------------------------------------------------------------------------+
```

### Breakpoints & Grid Rules:
- **Desktop Siêu rộng (> 1440px):** Container giới hạn `1440px`, căn giữa tự nhiên (`margin: 0 auto;`).
- **Desktop Tiêu chuẩn (1080px - 1440px):** Lưới 3 cột đồng đều: `grid-template-columns: repeat(3, 1fr); gap: 28px;`.
- **Tablet / Laptop nhỏ (680px - 1080px):** Lưới 2 cột: `grid-template-columns: repeat(2, 1fr); gap: 20px;`.
- **Mobile (< 680px):** Lưới 1 cột: `grid-template-columns: 1fr; gap: 16px;`, padding section giảm xuống `60px 16px`.

---

## V. CÁC PATTERN COMPONENT DOANH NGHIỆP CHUẨN HÓA

### 1. Component Mẫu: Sub-Page Hero Banner (Header chuẩn cho toàn bộ trang con)
Mỗi trang con (`/gioi-thieu`, `/dich-vu`, `/nang-luc-ha-tang`, `/tin-tuc`, `/tuyen-dung`, `/lien-he`) phải sở hữu Banner đồng bộ theo mẫu chuẩn sau:
- **Chiều cao:** `240px - 320px` (Desktop), `180px - 220px` (Mobile).
- **Phông nền:** Nền Deep Navy `#0F1D36` kết hợp hoa văn hình học hoặc ánh sáng xanh `#0c9344` gradient góc phải.
- **Thành phần:**
  1. **Breadcrumbs:** Tích hợp JSON-LD Schema `BreadcrumbList`, màu sắc tinh tế, icon Home `RiHome4Line`.
  2. **Category Pill:** Nhãn nhỏ chữ in hoa nổi bật trên nền bán trong suốt (ví dụ: `GIỚI THIỆU DOANH NGHIỆP`, `NĂNG LỰC TỔ HỢP`).
  3. **H1 Tiêu đề:** Chữ trắng `#FFFFFF`, font `Montserrat`, size `clamp(30px, 4vw, 44px)`.
  4. **Mô tả dẫn nhập (Lead Summary):** 1-2 câu ngắn tóm tắt giá trị trang, màu `#CBD5E1`.

### 2. Component Mẫu: Enterprise Card 16:9 (Thẻ nội dung dịch vụ, tin tức, năng lực)
Mọi card hiển thị danh sách trên website tuân thủ kiến trúc thống nhất:
- **Khung thẻ:** Nền trắng `#FFFFFF`, border `1px solid #E2E8F0`, bo góc `8px - 10px`, overflow `hidden`.
- **Khối hình ảnh (Image Box):**
  - Bắt buộc tỷ lệ `aspect-ratio: 16 / 9;`.
  - Phủ gradient đáy `rgba(15, 23, 42, 0.65)` giúp tôn nhãn chữ và icon.
  - **Floating Badge:** Nằm góc trên (`top: 12px; right: 12px;`) với màu Emerald `#0c9344` (cho Dịch vụ nổi bật) hoặc Navy Blue (cho Chuyên mục tin tức).
  - **Icon Badge:** Hộp vuông `44px x 44px` bo góc `6px`, màu Navy `#233871`, icon trắng, tự động đổi màu `#0c9344` khi hover vào card.
- **Thân thẻ (Card Body):**
  - **Tiêu đề H3:** `18px - 19px`, `font-weight: 800`, giới hạn tối đa 2 dòng.
  - **Mô tả ngắn:** Slate 500 `#64748B`, cắt chữ gọn 2 dòng `-webkit-line-clamp: 2`.
  - **Khối điểm nhấn (Bullet points):** Nền `#F8FAFC` viền `#F1F5F9` chứa 2-3 thông số kỹ thuật hoặc lợi thế then chốt có kèm icon xanh `RiCheckLine`.
- **Chân thẻ (Card Footer):**
  - Phân cách bằng đường kẻ hairline `#F1F5F9`.
  - Nút "Xem chi tiết ->" chuyển động mũi tên khi hover.
  - Nút phụ "Báo giá nhanh" hoặc nhãn thời gian đọc.

### 3. Component Mẫu: Section Header Block
Khối tiêu đề chuẩn ở đầu mỗi khu vực nội dung:
```tsx
<SectionHeader>
  <HeaderText>
    <div className="category">
      <RiSparklingFill /> NHÃN PHÂN LOẠI
    </div>
    <h2>Tiêu Đề Lớn Đoạt Chuẩn Cấp Tập Đoàn</h2>
    <p>Mô tả ngắn gọn, súc tích về giá trị và năng lực cung cấp cho doanh nghiệp.</p>
  </HeaderText>
  <ViewAllBtn href="/duong-dan">
    Xem tất cả <RiArrowRightLine />
  </ViewAllBtn>
</SectionHeader>
```

### 4. Component Mẫu: Header & Navigation
- **TopBar:** Nền gradient tối `#070F1E` -> `#172554`, chứa địa chỉ thực tế tại Cửa khẩu Quốc tế Tà Lùng, Hotline pill có khung viền `+84 865.865.600`, và Bộ chuyển đổi ngôn ngữ đa quốc gia (`LanguageSwitcher` `🇻🇳 VI`).
- **Logo kép chuẩn hóa:**
  - Logo đơn: `/images/logo-single-header.png` (chiều cao `50px`).
  - Vạch ngăn cách hairline thanh lịch: `1px solid #E2E8F0`.
  - Nhãn thương hiệu: `/images/Lable-header.png` (chiều cao `38px`).
- **Search Capsule:** Nút trigger viên thuốc `[ 🔍 Tìm kiếm... ⌘K ]` kích hoạt `SearchModal` thông minh hỗ trợ phím tắt `⌘K` / `Ctrl+K`.
- **Nút CTA Liên Hệ:** Bo góc `6px`, nền gradient xanh Emerald `#0c9344`, hiệu ứng ánh bạc quét chéo (shimmer).

### 5. Component Mẫu: Logo Preloader & Transition
- Màn hình Splash xuất hiện tự nhiên khi tải trang.
- Logo bay từ bên trái (`-140px`), Nhãn bay từ bên phải (`+140px`), hội tụ tại trung tâm kèm vòng phát sáng tỏa lan (`ringRipple`) và tia sáng quét bề mặt (`lightSweep`).
- Khẩu hiệu thương hiệu chuẩn xác: **"TRUNG TÂM LOGISTICS TẠI CỬA KHẨU QUỐC TẾ TÀ LÙNG"** (Không dùng chữ "Tổ hợp 25ha" làm slogan chung).

---

## VI. BẢN ĐỒ ÁP DỤNG HỆ THỐNG THIẾT KẾ CHO CÁC TRANG CON (PAGE FLOW BLUEPRINTS)

| Trang con | Route | Cấu trúc khối theo Design System | Trọng tâm trải nghiệm |
| :--- | :--- | :--- | :--- |
| **1. Giới Thiệu** | `/gioi-thieu` | 1. Sub-Page Hero Banner (H1 + Breadcrumbs)<br>2. Lịch sử hình thành & Tầm nhìn sứ mệnh<br>3. Bản sắc & Giá trị cốt lõi (3 Cột Cards)<br>4. Đội ngũ lãnh đạo chuyên trách<br>5. Bằng khen, Chứng nhận & Cam kết an ninh<br>6. CTA Hợp tác Doanh nghiệp | Thể hiện tầm vóc chính quy, bề dày kinh nghiệm và quan hệ thông quan Việt - Trung. |
| **2. Dịch Vụ Tổng Thể** | `/dich-vu` | 1. Sub-Page Hero Banner<br>2. Bộ lọc danh mục dịch vụ<br>3. Grid 6+ Enterprise Service Cards 16:9<br>4. Bảng so sánh gói dịch vụ & tiện ích<br>5. Quy trình 6 bước xuất nhập khẩu khép kín<br>6. B2B Quick Quote Form Drawer/Section | Tối ưu chuyển đổi, làm nổi bật tính trọn gói từ bến bãi đến thủ tục hải quan. |
| **3. Dịch Vụ Chi Tiết** | `/dich-vu/[slug]` | 1. Service Hero Banner kèm Badge nghiệp vụ<br>2. Chi tiết quy trình & Biểu phí tham khảo<br>3. Thông số kỹ thuật & Năng lực bãi/phương tiện<br>4. Thư viện hình ảnh thực tế 16:9<br>5. FAQ hỏi đáp nghiệp vụ hải quan<br>6. Form đăng ký tư vấn trực tiếp | Cung cấp tài liệu nghiệp vụ chi tiết, minh bạch chi phí cho doanh nghiệp B2B. |
| **4. Năng Lực Hạ Tầng** | `/nang-luc-ha-tang` | 1. Sub-Page Hero Banner (Tổ hợp 25ha)<br>2. Tổng quan quy hoạch phân khu 250.000m²<br>3. Hệ thống kho ngoại quan & Kho lạnh thông minh<br>4. Bãi container & Đội xe đầu kéo chuyên dụng<br>5. Trạm cân điện tử & Giám sát an ninh số 24/7<br>6. Bản đồ vệ tinh tương tác mặt bằng | Khẳng định quy mô độc quyền 25ha lớn nhất tại Cửa khẩu Quốc tế Tà Lùng. |
| **5. Tin Tức & Thị Trường**| `/tin-tuc` | 1. Sub-Page Hero Banner<br>2. Tin tiêu điểm nổi bật (Featured Hero Article 16:9)<br>3. Bộ lọc chuyên mục (Chính sách hải quan, Xuất nhập khẩu, Tin nội bộ)<br>4. Grid News Cards 16:9 phân trang chuẩn<br>5. Box đăng ký nhận bản tin thị trường biên mậu | Cung cấp thông tin cập nhật, xây dựng uy tín cố vấn pháp lý thương mại. |
| **6. Chi Tiết Tin Tức** | `/tin-tuc/[slug]` | 1. Breadcrumbs + Tiêu đề báo chí chuẩn SEO<br>2. Meta tác giả, ngày phát hành, thời gian đọc<br>3. Nội dung văn bản biên tập chuẩn Markdown Prose<br>4. Chèn ảnh minh họa 16:9 kèm chú thích<br>5. Khối bài viết liên quan & Chia sẻ mạng xã hội | Chuẩn SEO Schema Article, trải nghiệm đọc báo hiện đại, trực quan. |
| **7. Tuyển Dụng** | `/tuyen-dung` | 1. Sub-Page Hero Banner ("Đồng hành cùng Tà Lùng")<br>2. Môi trường làm việc & Chế độ đãi ngộ<br>3. Danh sách vị trí tuyển dụng (Job Cards)<br>4. Quy trình phỏng vấn & tiếp nhận<br>5. Form nộp CV trực tuyến | Thu hút nhân sự logistics chất lượng cao cho vùng kinh tế cửa khẩu. |
| **8. Liên Hệ & Chi Nhánh**| `/lien-he` | 1. Sub-Page Hero Banner<br>2. Thẻ thông tin 3 Trụ sở (Cao Bằng, Hà Nội, Lạng Sơn)<br>3. Bản đồ Google Maps tương tác thực tế<br>4. Form liên hệ doanh nghiệp (RHF + Zod validation)<br>5. Kênh hỗ trợ khẩn cấp 24/7 | Tiếp nhận đầu mối liên hệ, điều phối thông tin nhanh chóng cho đối tác. |

---

## VII. DANH MỤC KIỂM THỬ GIAO DIỆN BẮT BUỘC (ACCESSIBILITY & QUALITY CHECKLIST)

Mọi trang con trước khi nghiệm thu đều phải vượt qua Checklist sau:
- [ ] **Tỷ lệ ảnh:** 100% hình ảnh trong card và bài viết phải hiển thị đúng `16:9`, không méo ảnh.
- [ ] **Màu sắc:** Tuyệt đối không dùng `#0b9444`. Đúng mã Navy `#233871` và Emerald `#0c9344`.
- [ ] **Typography:** Toàn bộ tiêu đề H1-H3 đúng font `Montserrat`, chữ nội dung dùng `Inter`.
- [ ] **SEO Breadcrumbs:** Mọi trang con có Breadcrumbs kèm thẻ dữ liệu có cấu trúc `BreadcrumbList` JSON-LD.
- [ ] **Responsive:** Hiển thị mượt mà trên Mobile (375px), Tablet (768px), Laptop (1024px, 1280px) và Màn hình rộng (1440px+).
- [ ] **Tương tác:** Mọi nút bấm, link chuyển hướng có trạng thái hover rõ nét, phím tắt `⌘K` hoạt động thông suốt.

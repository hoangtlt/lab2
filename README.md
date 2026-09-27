# Orchid Gallery SPA – SBA301 Lab 02 & Slot 9

Dự án thực hành môn **SBA301 – Integrate Single Page Application with Spring Boot**.
Ứng dụng Single Page Application (SPA) trưng bày bộ sưu tập Hoa Lan (Orchids Gallery) với **React Bootstrap**, phân tầng kiến trúc **Service Layer**, quản lý trạng thái bất đồng bộ (**Loading / Error / Empty / Data states**), **Client-side Caching (TTL 30s + Force Reload)**, cùng tính năng **Search / Filter** tối ưu không làm phát sinh request mạng mới.

---

## 1. Yêu cầu môi trường (Requirements)
- **Node.js**: Phiên bản 20.19+ hoặc 22.12+ (Hiện tại: Node v22.23.2, npm 10.9.8).
- **Trình duyệt**: Google Chrome / Microsoft Edge với DevTools.
- **IDE**: IntelliJ IDEA hoặc Visual Studio Code.

---

## 2. Cài đặt và Khởi chạy (Install & Run)

### 2.1 Cài đặt dependencies
```bash
cd orchid-gallery-spa
npm install
```

### 2.2 Chạy môi trường Development
```bash
npm run dev
```
Truy cập tại địa chỉ: `http://localhost:5173/`

### 2.3 Build Production và Preview
```bash
npm run build
npm run preview
```

---

## 3. Cấu trúc thư mục (Project Structure)
```text
orchid-gallery-spa/
├── public/
│   ├── orchids.json                  # Dữ liệu 8 hoa lan chuẩn định dạng JSON
│   └── images/
│       └── orchid-placeholder.svg    # Ảnh vector SVG offline an toàn
├── src/
│   ├── api/
│   │   ├── apiClient.js              # Cấu hình Axios instance
│   │   ├── orchidService.js          # Service layer chính (Fetch API + Cache TTL 30s)
│   │   └── orchidService.axios.example.js # Module tham chiếu so sánh giữa Fetch và Axios
│   ├── components/
│   │   ├── NavBar.jsx                # Thanh điều hướng React Bootstrap (Home/Orchids/About)
│   │   ├── OrchidCard.jsx            # Thẻ hiển thị hoa lan (Image, Name, Category, Badge, Detail button)
│   │   ├── OrchidDetailModal.jsx     # Modal chi tiết hoa lan (null-safe rendering)
│   │   ├── Orchids.jsx               # Component container chính kết nối state và list
│   │   ├── SearchBox.jsx             # Ô tìm kiếm theo tên hoa lan (Derived state)
│   │   ├── CategoryFilter.jsx        # Bộ lọc danh mục & switch 'Chỉ hoa đặc biệt'
│   │   ├── LoadingSpinner.jsx        # Spinner hiển thị khi đang tải dữ liệu
│   │   └── ErrorMessage.jsx          # Thông báo lỗi Alert + nút Try Again
│   ├── hooks/
│   │   └── useOrchids.js             # Custom hook quản lý lifecycle, loading/error/data & reload
│   ├── shared/
│   │   └── ListOfOrchids.js          # Dữ liệu tĩnh ban đầu (Lab 02 Core baseline)
│   ├── styles/
│   │   └── app.css                   # Custom CSS bổ trợ cho Bootstrap
│   ├── App.jsx                       # Root layout component
│   └── main.jsx                      # Entry point, import Bootstrap CSS & app.css
├── package.json
├── vite.config.js
└── README.md
```

---

## 4. Các tính năng cốt lõi (Core Features)

### Lớp A – Lab 02 Core:
1. **React Bootstrap NavBar**: Tên thương hiệu *Orchid Gallery* với các mục menu responsive *Home, Orchids, About*.
2. **Danh sách Hoa Lan**: Grid 8 card hoa lan với đầy đủ: tên hoa, hình ảnh, danh mục, xếp hạng rating, badge đặc biệt (*Special* màu vàng), nút *Detail*.
3. **Modal chi tiết (OrchidDetailModal)**: Hiển thị đầy đủ thông tin chi tiết (Origin, Color, Rating, Special, Description) khi bấm nút Detail; đóng modal qua nút X hoặc nút Close.

### Lớp B – Slot 9 Extension:
1. **Kiến trúc tách biệt (Separation of Concerns)**:
   - UI Component chỉ tập trung render (`Orchids.jsx`, `OrchidCard.jsx`).
   - Hook tách biệt quản lý lifecycle và state (`useOrchids.js`).
   - Service layer tập trung xử lý data access và cache (`orchidService.js`).
2. **Bất đồng bộ (Async/Await & Fetch API)**: Lấy dữ liệu từ `/orchids.json`, kiểm tra `response.ok` chặt chẽ.
3. **State Model hoàn chỉnh**:
   - `loading`: Hiển thị `LoadingSpinner`.
   - `error`: Hiển thị `ErrorMessage` với nút *Try Again*.
   - `empty`: Hiển thị thông báo khi không có dữ liệu phù hợp.
   - `data`: Render danh sách Card.
4. **Client-side Caching (TTL = 30s)**:
   - Khi load lại trong vòng 30 giây: Sử dụng dữ liệu trong cache tức thì (Cache Hit), không gửi HTTP request dư thừa.
   - Khi hết 30 giây hoặc bấm **Reload (Bypass Cache)**: Tự động bỏ qua cache và fetch dữ liệu mới.
5. **Axios Awareness**: Có cấu hình `apiClient.js` và `orchidService.axios.example.js` để đối chiếu trực tiếp với `fetch()`.

### Lớp C – Mở rộng (Search & Filter):
- **Tìm kiếm theo tên** (SearchBox): Lọc trực tiếp tức thì qua derived state.
- **Lọc theo danh mục** (CategoryFilter): Dropdown chọn tất cả hoặc từng danh mục (Dendrobium, Cattleya, Phalaenopsis, Oncidium, Vanda).
- **Lọc hoa đặc biệt**: Toggle switch "Chỉ hoa đặc biệt (Special)".
- **Tối ưu**: Toàn bộ thao tác search/filter là **Derived State** (dữ liệu suy diễn), hoàn toàn không gọi lại API hay tạo request mạng mới.

---

## 5. So sánh Fetch API và Axios

| Tiêu chí | Fetch API (Core triển khai) | Axios (`orchidService.axios.example.js`) |
| :--- | :--- | :--- |
| **Cài đặt thư viện** | Tích hợp sẵn trong trình duyệt (Không cần npm install) | Cần cài đặt gói `axios` qua npm |
| **Xử lý JSON** | Cần gọi phương thức `await response.json()` | Tự động phân tích JSON, dữ liệu nằm trong `response.data` |
| **Xử lý lỗi HTTP (4xx/5xx)** | Không reject Promise; bắt buộc kiểm tra `response.ok` hoặc `response.status` | Tự động reject Promise khi status nằm ngoài dải 2xx |
| **Cấu hình Timeout** | Cần phối hợp với `AbortController` | Có sẵn thuộc tính `timeout: 5000` |
| **Request / Response Interceptors** | Không hỗ trợ sẵn | Hỗ trợ mạnh mẽ qua `axios.interceptors` |

---

## 6. Chính sách Cache (Cache Policy)
- **Cơ chế**: Cache lưu trong biến module `orchidCache` kèm timestamp `cacheTime`.
- **Thời gian hiệu lực (TTL)**: 30.000 ms (30 giây).
- **Kiểm tra cache (Cache Validation)**:
  ```javascript
  const validCache = orchidCache && (Date.now() - cacheTime < CACHE_DURATION);
  if (!force && validCache) return orchidCache;
  ```
- **Force Reload**: Nút `Reload` gọi `reload()` trong hook `useOrchids`, kích hoạt cờ `force = true` để bypass cache và phát request HTTP mới ngay lập tức.

---

## 7. Break-It Lab: Báo cáo thử nghiệm và xử lý lỗi có chủ đích

| Mã lỗi / Tình huống | Cách tạo lỗi | Dấu hiệu / Hiện tượng | Phân tích & Cách khắc phục |
| :--- | :--- | :--- | :--- |
| **Bug 1: 404 Not Found** | Đổi URL fetch sang `/orchidss.json` | Giao diện hiện Alert lỗi màu đỏ: *"HTTP 404: Không thể tải orchids.json"* | Kiểm tra Network tab thấy request đỏ 404. Cần đảm bảo đúng đường dẫn file tĩnh đặt trong thư mục `public/`. |
| **Bug 2: JSON Parse Error** | Thêm dấu phẩy dư thừa (trailing comma) vào `public/orchids.json` | Console báo lỗi `SyntaxError: Unexpected token ... in JSON`, UI chuyển sang Error state | File JSON chuẩn không chấp nhận trailing comma hoặc comment; cần sửa định dạng JSON chuẩn. |
| **Bug 3: Modal Crash khi orchid=null** | Bỏ optional chaining trong Modal (ví dụ gọi `orchid.orchidName` khi `orchid` là `null`) | Ứng dụng crash với lỗi `Cannot read properties of null` khi vừa mount | Dùng optional chaining `orchid?.orchidName` và bọc điều kiện kiểm tra `{orchid ? (...) : <p>Chưa chọn Orchid.</p>}`. |
| **Bug 4: Infinite Fetch Loop** | Truyền trực tiếp inline function không dùng `useCallback` vào dependency của `useEffect` | Network liên tục spam request `/orchids.json` không ngừng | Sử dụng `useCallback` bọc hàm fetch và đặt dependency array `[]` hợp lý. |
| **Bug 5: Reload không cập nhật** | Nút Reload chỉ gọi lại hàm fetch thông thường mà không truyền `force = true` | Bấm Reload nhưng Network tab không xuất hiện request mới | Trong hook và service cần truyền cờ `force: true` để vô hiệu hóa kiểm tra cache TTL. |

---

## 8. Trả lời 20 câu hỏi Human Verification Gate (Phần 12.1)

1. **Vì sao Lab 02 core nên chạy với static data trước khi chuyển sang Fetch?**  
   *Trả lời:* Giúp cô lập bài toán: xác nhận component hierarchy, props, layout React Bootstrap, và tương tác mở/đóng Modal hoạt động đúng 100% trước, tránh trường hợp lỗi giao diện bị nhầm lẫn với lỗi mạng/HTTP.
2. **Props nào đi từ Orchids xuống OrchidCard?**  
   *Trả lời:* Props `orchid` (object dữ liệu 1 bông lan) và prop callback `onDetail` (hàm nhận sự kiện khi người dùng nhấn xem chi tiết).
3. **Event nào đi từ OrchidCard lên Orchids?**  
   *Trả lời:* Sự kiện click nút Detail, gọi `onDetail(orchid)` truyền object `orchid` được click ngược lên component cha `Orchids`.
4. **selectedOrchid và show khác nhau về vai trò thế nào?**  
   *Trả lời:* `show` (boolean) quyết định trạng thái hiển thị ẩn/hiện của Modal; `selectedOrchid` (object|null) lưu dữ liệu của đối tượng hoa lan đang được chọn để render nội dung trong Modal.
5. **Nếu orchid=null, Modal cần xử lý gì?**  
   *Trả lời:* Phải xử lý null-safe bằng optional chaining (`orchid?.orchidName`) hoặc conditional rendering để tránh lỗi runtime `TypeError: Cannot read properties of null`.
6. **Promise có ba trạng thái nào?**  
   *Trả lời:* `pending` (đang chờ xử lý), `fulfilled` (hoàn thành thành công), và `rejected` (thất bại/có lỗi).
7. **await làm gì với Promise?**  
   *Trả lời:* `await` tạm dừng thực thi hàm `async` cho tới khi Promise được giải quyết (settled), trả về kết quả fulfilled hoặc ném ra exception nếu Promise bị reject.
8. **Tại sao loading phải tắt trong finally?**  
   *Trả lời:* Vì khối `finally` luôn luôn được thực thi dù request thành công (try) hay thất bại (catch), đảm bảo Spinner không bao giờ bị treo vô tận.
9. **Vì sao fetch 404 không tự vào catch chỉ vì status 404?**  
   *Trả lời:* Vì theo đặc tả của Fetch API, Promise chỉ reject khi gặp lỗi mạng vật lý (network failure, mất kết nối, DNS fail). Mã phản hồi HTTP 404 hoặc 500 vẫn được tính là một HTTP response hợp lệ từ server.
10. **response.ok dùng để làm gì?**  
    *Trả lời:* `response.ok` là thuộc tính boolean trả về `true` nếu mã trạng thái HTTP nằm trong khoảng thành công 200–299, dùng để phát hiện sớm các lỗi 4xx/5xx và chủ động throw Error.
11. **public/orchids.json được truy cập bằng URL nào?**  
    *Trả lời:* Trong Vite, các file trong thư mục `public/` được serve trực tiếp tại root URL, nên truy cập bằng `/orchids.json` (không dùng `/public/orchids.json`).
12. **Service layer giải quyết coupling nào?**  
    *Trả lời:* Giảm sự phụ thuộc chặt (decoupling) giữa giao diện người dùng (UI Components) và phương thức truy xuất dữ liệu (Fetch/Axios/Cache/Endpoints). Nếu thay đổi URL hoặc thư viện HTTP, chỉ cần sửa Service Layer.
13. **useOrchids chịu trách nhiệm gì, và không nên chịu trách nhiệm gì?**  
    *Trả lời:* Chịu trách nhiệm quản lý state React (loading, error, data) và vòng đời lifecycle của request; không nên chứa trực tiếp mã gọi mạng cấp thấp (Fetch headers, parse JSON) hay render giao diện HTML/JSX.
14. **Cache hit khác cache miss thế nào?**  
    *Trả lời:* `Cache hit` xảy ra khi dữ liệu yêu cầu đã có trong cache và vẫn còn trong hạn TTL (trả về ngay lập tức, không tốn network request); `Cache miss` xảy ra khi chưa có cache hoặc cache đã hết hạn (phải gửi request mạng mới đến server).
15. **TTL 30 giây có nghĩa gì?**  
    *Trả lời:* Time-To-Live = 30s nghĩa là dữ liệu được lưu tạm trong bộ nhớ có hiệu lực trong 30 giây kể từ thời điểm fetch thành công. Sau 30 giây, cache bị coi là cũ (stale) và request tiếp theo sẽ fetch lại.
16. **Force Reload khác normal load thế nào?**  
    *Trả lời:* `Normal load` sẽ ưu tiên kiểm tra cache hợp lệ trước; `Force Reload` chủ động bỏ qua (bypass) hoặc xóa cache hiện tại để ép buộc ứng dụng thực hiện HTTP request mới lấy dữ liệu mới nhất từ server.
17. **Dùng DevTools chứng minh request thực sự xảy ra bằng cách nào?**  
    *Trả lời:* Mở tab `Network`, lọc request `Fetch/XHR`, quan sát thấy dòng `orchids.json` với Status Code 200, Method GET, Headers và Response payload.
18. **Axios trả dữ liệu ở đâu?**  
    *Trả lời:* Axios tự động giải mã dữ liệu trả về và đặt toàn bộ payload vào thuộc tính `response.data`.
19. **Search/filter local có nên gọi API mới không? Vì sao?**  
    *Trả lời:* Không nên; vì toàn bộ 8 hoa lan đã được tải về client, việc lọc trên mảng có sẵn (derived state) mang lại trải nghiệm tức thì (0ms latency) và giảm tải không cần thiết cho server.
20. **Nếu thay JSON file bằng Spring Boot API, component nào lý tưởng không cần đổi?**  
    *Trả lời:* Các UI component (`NavBar`, `OrchidCard`, `OrchidDetailModal`, `SearchBox`, `CategoryFilter`, `LoadingSpinner`, `ErrorMessage`) và hook `useOrchids` hoàn toàn không cần thay đổi; chỉ cần cập nhật URL endpoint bên trong `orchidService.js`.

---

## 9. AI Verification Log

| Prompt | Đề xuất từ AI | Evidence đã kiểm chứng | Quyết định / Tinh chỉnh |
| :--- | :--- | :--- | :--- |
| Thiết kế kiến trúc phân lớp cho SBA301 Lab 02 | Tách `api/orchidService.js`, `hooks/useOrchids.js` và UI components | Cấu trúc thư mục sạch, import đúng, build production không có cảnh báo vòng lặp | Phê duyệt kiến trúc; giữ đúng chuẩn tách lớp của syllabus. |
| Xây dựng cơ chế Client-side Cache TTL 30s | Sử dụng biến module `orchidCache` và so sánh `Date.now() - cacheTime < 30000` | Kiểm tra qua Browser DevTools: tải lại không sinh request mới; bấm Reload (force=true) sinh request mới | Phê duyệt và áp dụng chuẩn xác. |
| Tích hợp bộ lọc Search & Category | Tính toán danh sách `visibleOrchids` trực tiếp qua derived state (`useMemo`) | Browser subagent kiểm tra: lọc Ceasar ra 1 card, lọc Cattleya ra 1 card, bật Special ra 5 cards | Phê duyệt giải pháp derived state, tránh dùng useEffect gây re-render dư thừa. |

---

## 10. Checklist đánh giá hoàn thành (SBA301 Checklist)
- [x] Cài đặt thành công React Bootstrap, Bootstrap CSS và Axios.
- [x] Tạo thanh điều hướng NavBar responsive với React Bootstrap.
- [x] Khởi tạo dữ liệu OrchidsData đầy đủ 8 bản ghi trong ListOfOrchids.js.
- [x] Render danh sách hoa lan bằng hệ thống thẻ Card linh hoạt.
- [x] Modal xem chi tiết an toàn hiển thị đúng hoa lan được chọn.
- [x] Phân tầng Service Layer (`orchidService.js`) và Custom Hook (`useOrchids.js`).
- [x] Tích hợp Fetch API gọi dữ liệu từ `/orchids.json` với kiểm tra `response.ok`.
- [x] Đầy đủ 4 trạng thái: Loading, Error (có nút Try Again), Empty, Data.
- [x] Cache client với TTL 30 giây kèm nút Force Reload.
- [x] Tạo file ví dụ so sánh với Axios (`apiClient.js`, `orchidService.axios.example.js`).
- [x] Tính năng mở rộng: Tìm kiếm theo từ khóa, lọc theo danh mục, switch lọc hoa đặc biệt.
- [x] Kiểm tra thành công kịch bản Break-It Lab có chủ đích.
- [x] Build production thành công không có lỗi (`npm run build`).

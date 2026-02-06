# TÀI LIỆU THIẾT KẾ API - HỆ THỐNG E-COMMERCE

**Dự án:** E-Commerce Platform - Laptop & Electronics Store  
**Phiên bản:** 1.0.0  
**Ngày tạo:** 06/02/2026  
**Người viết:** Software Architect Team  

---

## MỤC LỤC

1. [Tổng quan thiết kế API](#6-thiết-kế-api-api-design)
2. [Nguyên tắc thiết kế API](#62-nguyên-tắc-thiết-kế-api)
3. [Phân loại API theo nghiệp vụ](#63-phân-loại-api-theo-nghiệp-vụ)
4. [Danh sách API tổng hợp](#64-danh-sách-api-tổng-hợp)
5. [Mô tả chi tiết API](#65-mô-tả-chi-tiết-api)
6. [Luồng API tiêu biểu](#66-luồng-api-tiêu-biểu)
7. [Xử lý lỗi & Exception](#67-xử-lý-lỗi--exception)
8. [Tóm tắt](#68-tóm-tắt-phần-thiết-kế-api)

---

## 6. THIẾT KẾ API (API Design)

### 6.1. Tổng quan thiết kế API

#### 6.1.1. Mục tiêu thiết kế API

Hệ thống E-Commerce được thiết kế với các mục tiêu API sau:

| **Mục tiêu** | **Mô tả** | **Ưu tiên** |
|-------------|-----------|-------------|
| **Tính nhất quán** | Đảm bảo cấu trúc API đồng nhất, dễ sử dụng | Cao |
| **Khả năng mở rộng** | Hỗ trợ thêm tính năng mới không ảnh hưởng API cũ | Cao |
| **Bảo mật** | Xác thực và phân quyền rõ ràng | Rất cao |
| **Hiệu năng** | Tối ưu thời gian phản hồi, hỗ trợ phân trang | Cao |
| **Dễ bảo trì** | Code rõ ràng, tài liệu đầy đủ | Trung bình |

#### 6.1.2. Vai trò của API trong hệ thống

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT LAYER                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Web Browser │  │ Mobile App   │  │  Admin Panel │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
└─────────┼──────────────────┼──────────────────┼──────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                    ┌────────▼────────┐
                    │   API GATEWAY   │
                    │  (Spring MVC)   │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐     ┌─────▼─────┐     ┌─────▼─────┐
    │  User API │     │Product API│     │ Cart API  │
    └─────┬─────┘     └─────┬─────┘     └─────┬─────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                    ┌────────▼────────┐
                    │  SERVICE LAYER  │
                    │  (Business Logic)│
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ REPOSITORY LAYER│
                    │   (Data Access) │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │ ORACLE DATABASE │
                    └─────────────────┘
```

**Vai trò chính:**
- **Tách biệt Frontend - Backend**: API làm cầu nối giữa giao diện và logic nghiệp vụ
- **Tái sử dụng**: Một API có thể phục vụ nhiều client (Web, Mobile, Admin)
- **Bảo mật tập trung**: Xác thực và phân quyền được xử lý tại API layer
- **Dễ kiểm thử**: API có thể test độc lập với giao diện

#### 6.1.3. Phạm vi các API được cung cấp

| **Nhóm API** | **Mô tả** | **Số lượng Endpoint** |
|-------------|-----------|----------------------|
| **Authentication** | Đăng nhập, đăng ký, quên mật khẩu | 6 |
| **User Management** | Quản lý thông tin người dùng | 4 |
| **Product Management** | CRUD sản phẩm, tìm kiếm, lọc | 12 |
| **Category Management** | Quản lý danh mục sản phẩm | 5 |
| **Cart Management** | Giỏ hàng (thêm, xóa, cập nhật) | 3 |
| **Order Management** | Đặt hàng, theo dõi đơn hàng | 8 |
| **Review Management** | Đánh giá sản phẩm | 3 |
| **Banner Management** | Quản lý banner quảng cáo | 5 |
| **Hot Deal Management** | Quản lý khuyến mãi | 5 |
| **Customer Management** | Quản lý khách hàng (Admin) | 6 |
| **TỔNG CỘNG** | | **57 Endpoints** |

---

### 6.2. Nguyên tắc thiết kế API

#### 6.2.1. Thiết kế theo RESTful API

Hệ thống tuân thủ các nguyên tắc REST:

| **Nguyên tắc** | **Áp dụng trong hệ thống** |
|---------------|---------------------------|
| **Client-Server** | Tách biệt rõ ràng giữa Frontend (Thymeleaf) và Backend (Spring Boot) |
| **Stateless** | Mỗi request chứa đầy đủ thông tin, không lưu state trên server (trừ session authentication) |
| **Cacheable** | Sử dụng HTTP headers để cache static resources |
| **Uniform Interface** | Sử dụng HTTP methods chuẩn, URL có cấu trúc rõ ràng |
| **Layered System** | Controller → Service → Repository → Database |

**Lưu ý đặc biệt:**
- Hệ thống sử dụng **Session-based Authentication** thay vì JWT do yêu cầu bảo mật cao
- Có 2 SecurityFilterChain riêng biệt cho Admin và User

#### 6.2.2. Quy ước đặt tên Endpoint

**Cấu trúc URL chuẩn:**
```
{base_url}/{version}/{resource}/{id}/{sub-resource}
```

**Ví dụ thực tế:**
```
http://localhost:8080/admin/product/123
http://localhost:8080/api/cart/add/456
http://localhost:8080/user/orders
```

**Quy tắc đặt tên:**

| **Quy tắc** | **Đúng** | **Sai** |
|------------|---------|---------|
| Sử dụng danh từ số nhiều | `/products` | `/product` |
| Chữ thường, dấu gạch ngang | `/hot-deals` | `/HotDeals` |
| Không dùng động từ trong URL | `/products/123` | `/getProduct/123` |
| Phân cấp rõ ràng | `/products/123/reviews` | `/product-reviews/123` |
| Sử dụng query params cho filter | `/products?brand=Dell&sort=price` | `/products/Dell/sortByPrice` |

**Áp dụng trong source code:**
```java
// ✅ Đúng
@GetMapping("/admin/product")
@GetMapping("/admin/edit-product/{id}")
@PostMapping("/cart/add/{id}")

// ❌ Sai (không có trong code)
@GetMapping("/admin/getProduct")
@PostMapping("/admin/createNewProduct")
```

#### 6.2.3. Sử dụng HTTP Method

##### **GET - Lấy dữ liệu**

**Đặc điểm:**
- Không thay đổi dữ liệu trên server
- Có thể cache
- Idempotent (gọi nhiều lần cho kết quả giống nhau)

**Ví dụ trong source code:**
```java
// Lấy danh sách sản phẩm
@GetMapping("/admin/product")
public String index(Model model, @Param("keyword") String keyword,
                    @RequestParam(name = "pageNo", defaultValue = "1") Integer pageNo)

// Lấy chi tiết sản phẩm
@GetMapping("/api/admin/product/{id}")
@ResponseBody
public ResponseEntity<?> getProductById(@PathVariable("id") Long id)

// Lấy danh sách đơn hàng
@GetMapping("/admin/order")
public String showOrders(@RequestParam(value = "status", required = false) Integer status,
                         @RequestParam(value = "keyword", required = false) String keyword)
```

##### **POST - Tạo mới hoặc xử lý dữ liệu**

**Đặc điểm:**
- Tạo resource mới
- Không idempotent
- Có thể thay đổi state

**Ví dụ trong source code:**
```java
// Thêm sản phẩm vào giỏ hàng
@PostMapping("/cart/add/{id}")
public ResponseEntity<?> addToCart(@PathVariable Long id, HttpSession session)

// Tạo sản phẩm mới
@PostMapping("/admin/add-product")
public String save(@ModelAttribute("product") Product product,
                   @RequestParam("imageFile") MultipartFile file)

// Đăng ký người dùng
@PostMapping("/user/register")
public String handleRegister(@RequestParam("username") String username,
                             @RequestParam("email") String email,
                             @RequestParam("password") String password)
```

##### **PUT - Cập nhật toàn bộ**

**Lưu ý:** Source code không sử dụng PUT method, thay vào đó dùng POST cho cả create và update.

##### **DELETE - Xóa dữ liệu**

**Ví dụ trong source code:**
```java
// Xóa sản phẩm
@GetMapping("/admin/delete-product/{id}")
public String delete(@PathVariable("id") Long id, RedirectAttributes redirect)

// Xóa danh mục
@GetMapping("/admin/delete-category/{id}")
public String delete(@PathVariable("id") Integer id, RedirectAttributes redirect)
```

**⚠️ Lưu ý:** Hệ thống sử dụng `@GetMapping` cho DELETE thay vì `@DeleteMapping` (không theo chuẩn REST thuần túy).

#### 6.2.4. Sử dụng HTTP Status Code

##### **2xx – Success**

| **Code** | **Ý nghĩa** | **Sử dụng trong hệ thống** |
|---------|-----------|---------------------------|
| **200 OK** | Request thành công | Trả về danh sách, chi tiết resource |
| **201 Created** | Tạo mới thành công | (Không sử dụng rõ ràng) |
| **204 No Content** | Xóa thành công, không trả về body | (Không sử dụng) |

**Ví dụ:**
```java
@PostMapping("/cart/add/{id}")
public ResponseEntity<?> addToCart(@PathVariable Long id, HttpSession session) {
    // ...
    return ResponseEntity.ok(response); // 200 OK
}
```

##### **4xx – Client Error**

| **Code** | **Ý nghĩa** | **Sử dụng trong hệ thống** |
|---------|-----------|---------------------------|
| **400 Bad Request** | Dữ liệu đầu vào không hợp lệ | Validation error |
| **401 Unauthorized** | Chưa đăng nhập | Spring Security tự động xử lý |
| **403 Forbidden** | Không có quyền truy cập | Spring Security tự động xử lý |
| **404 Not Found** | Không tìm thấy resource | Custom exception |

**Ví dụ:**
```java
@GetMapping("/api/admin/product/{id}")
@ResponseBody
public ResponseEntity<?> getProductById(@PathVariable("id") Long id) {
    Optional<Product> productOpt = productService.findById(id);
    if (productOpt.isPresent()) {
        return ResponseEntity.ok(new ProductDTO(productOpt.get()));
    } else {
        return ResponseEntity.status(404)
            .body("{\"message\": \"Không tìm thấy sản phẩm\"}"); // 404
    }
}
```

##### **5xx – Server Error**

| **Code** | **Ý nghĩa** | **Xử lý** |
|---------|-----------|----------|
| **500 Internal Server Error** | Lỗi server | Exception handler |
| **503 Service Unavailable** | Service tạm thời không khả dụng | Database connection error |

#### 6.2.5. Định dạng dữ liệu trao đổi

##### **JSON Format**

Hệ thống sử dụng JSON làm định dạng chính cho API responses.

**Cấu trúc Response chuẩn:**
```json
{
  "status": "success",
  "data": {
    "id": 123,
    "name": "Dell XPS 15",
    "price": 25000000
  },
  "message": "Lấy dữ liệu thành công"
}
```

**Ví dụ thực tế từ source code:**
```java
// Cart API Response
@PostMapping("/cart/add/{id}")
public ResponseEntity<?> addToCart(@PathVariable Long id, HttpSession session) {
    // ...
    Map<String, Object> response = new HashMap<>();
    response.put("totalItems", cart.getTotalItems());
    return ResponseEntity.ok(response);
}
```

**Response JSON:**
```json
{
  "totalItems": 3
}
```

##### **Encoding**

- **Character Encoding:** UTF-8
- **Content-Type:** `application/json; charset=UTF-8`
- **Accept:** `application/json`

**Cấu hình trong application.properties:**
```properties
spring.thymeleaf.encoding=UTF-8
```

#### 6.2.6. Versioning API

**Chiến lược hiện tại:**
- Hệ thống **chưa áp dụng versioning** rõ ràng
- Tất cả API đều ở version mặc định (v1 ngầm định)

**Đề xuất cho tương lai:**
```
/api/v1/products
/api/v2/products
```

**Hoặc sử dụng Header:**
```
Accept: application/vnd.ecommerce.v1+json
```

#### 6.2.7. Bảo mật API (Authentication & Authorization)

##### **Authentication - Xác thực**

Hệ thống sử dụng **Spring Security** với 2 cơ chế riêng biệt:

| **Loại** | **Endpoint** | **Phương thức** | **Session Key** |
|---------|-------------|----------------|----------------|
| **Admin** | `/admin/**`, `/logon` | Form-based Login | `SPRING_SECURITY_CONTEXT_ADMIN` |
| **User** | `/user/**`, `/` | Form-based Login | `SPRING_SECURITY_CONTEXT_USER` |

**Cấu hình Security:**
```java
@Bean
@Order(1)
public SecurityFilterChain adminSecurityFilterChain(HttpSecurity http) {
    http.securityMatcher("/admin/**", "/logon", "/signup")
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/logon", "/signup").permitAll()
            .anyRequest().authenticated())
        .formLogin(form -> form
            .loginPage("/logon")
            .defaultSuccessUrl("/admin", true))
        .securityContext(security -> 
            security.securityContextRepository(adminRepo));
    return http.build();
}
```

##### **Authorization - Phân quyền**

| **Role** | **Quyền truy cập** |
|---------|-------------------|
| **ROLE_ADMIN** | Toàn bộ `/admin/**` |
| **ROLE_USER** | `/user/profile`, `/user/orders`, `/orders/**` |
| **Anonymous** | `/`, `/products`, `/categories`, `/deals`, `/user/login`, `/user/register` |

**Ví dụ phân quyền:**
```java
@GetMapping("/admin/dashboard")
public String dashboard(Model model) {
    // Chỉ ROLE_ADMIN mới truy cập được
}

@GetMapping("/user/profile")
public String showProfilePage(Model model, Principal principal) {
    // Yêu cầu authenticated user
}
```

##### **Password Encryption**

```java
@Bean
public BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

// Sử dụng
String encodedPassword = passwordEncoder.encode(password);
```

---

### 6.3. Phân loại API theo nghiệp vụ

#### 6.3.1. API User

**Mục đích:** Quản lý tài khoản người dùng, xác thực, phân quyền

**Các chức năng chính:**
- Đăng ký tài khoản mới
- Đăng nhập (Admin & User)
- Quên mật khẩu / Đặt lại mật khẩu
- Xem và cập nhật thông tin cá nhân
- Quản lý khách hàng (Admin only)

**Endpoints:**
```
POST   /user/register          - Đăng ký user mới
POST   /user/login             - Đăng nhập user
POST   /logon                  - Đăng nhập admin
GET    /user/profile           - Xem thông tin cá nhân
POST   /user/forgot_password   - Quên mật khẩu
POST   /reset-password         - Đặt lại mật khẩu
```

#### 6.3.2. API Product

**Mục đích:** Quản lý sản phẩm (Laptop, Smartphone, Camera, Accessories)

**Các chức năng chính:**
- CRUD sản phẩm (Admin)
- Tìm kiếm sản phẩm theo tên, category
- Lọc theo brand, giá
- Phân trang
- Xem chi tiết sản phẩm
- Sản phẩm liên quan

**Endpoints:**
```
GET    /admin/product                    - Danh sách sản phẩm (Admin)
GET    /admin/add-product                - Form thêm sản phẩm
POST   /admin/add-product                - Tạo sản phẩm mới
GET    /admin/edit-product/{id}          - Form sửa sản phẩm
POST   /admin/edit-product               - Cập nhật sản phẩm
GET    /admin/delete-product/{id}        - Xóa sản phẩm
GET    /api/admin/product/{id}           - Lấy chi tiết sản phẩm (JSON)
GET    /laptops                          - Danh sách laptop
GET    /smartphones                      - Danh sách smartphone
GET    /cameras                          - Danh sách camera
GET    /accessories                      - Danh sách phụ kiện
GET    /product/quickview/{id}           - Xem nhanh sản phẩm
```

#### 6.3.3. API Cart

**Mục đích:** Quản lý giỏ hàng (Session-based)

**Các chức năng chính:**
- Thêm sản phẩm vào giỏ
- Cập nhật số lượng
- Xóa sản phẩm khỏi giỏ
- Tính tổng tiền

**Endpoints:**
```
POST   /cart/add/{id}          - Thêm sản phẩm vào giỏ
GET    /cart                   - Xem giỏ hàng
POST   /cart/update/{id}       - Cập nhật số lượng
DELETE /cart/remove/{id}       - Xóa sản phẩm
```

**Cấu trúc Cart trong Session:**
```java
public class Cart {
    private Map<Long, Integer> items = new HashMap<>();
    
    public void addItem(Product product) {
        Long id = product.getId();
        items.put(id, items.getOrDefault(id, 0) + 1);
    }
    
    public int getTotalItems() {
        return items.values().stream().mapToInt(i -> i).sum();
    }
}
```

#### 6.3.4. API Order

**Mục đích:** Quản lý đơn hàng

**Các chức năng chính:**
- Tạo đơn hàng từ giỏ hàng
- Xem lịch sử đơn hàng
- Cập nhật trạng thái đơn (Admin)
- Hủy đơn hàng
- Xác nhận đã nhận hàng

**Endpoints:**
```
GET    /admin/order                      - Danh sách đơn hàng (Admin)
GET    /admin/order/{id}                 - Chi tiết đơn hàng
GET    /admin/order/confirm/{id}         - Xác nhận đơn hàng
GET    /admin/order/cancel/{id}          - Hủy đơn hàng
GET    /admin/order/updateStatus/{id}/{status} - Cập nhật trạng thái
GET    /user/orders                      - Lịch sử đơn hàng (User)
GET    /orders/received/{id}             - Xác nhận đã nhận
GET    /orders/cancel/{id}               - Hủy đơn (User)
```

**Trạng thái đơn hàng:**
```java
1 - Chờ xác nhận
2 - Đã xác nhận
3 - Đang chuyển
4 - Đang giao
5 - Đã hủy
6 - Thành công
7 - Yêu cầu hủy
```

#### 6.3.5. API Payment (nếu có)

**Lưu ý:** Hệ thống hiện tại **chưa implement** API Payment.

**Đề xuất cho tương lai:**
```
POST   /api/payment/create     - Tạo giao dịch thanh toán
POST   /api/payment/vnpay      - Thanh toán qua VNPay
POST   /api/payment/momo       - Thanh toán qua Momo
GET    /api/payment/callback   - Callback từ payment gateway
GET    /api/payment/status/{id} - Kiểm tra trạng thái thanh toán
```

---

### 6.4. Danh sách API tổng hợp

#### 6.4.1. Bảng danh sách Endpoint

##### **Authentication & User Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Đăng ký User | `/user/register` | POST | Tạo tài khoản user mới | Public |
| Đăng nhập User | `/user/login` | POST | Xác thực user | Public |
| Đăng nhập Admin | `/logon` | POST | Xác thực admin | Public |
| Đăng ký Admin | `/signup` | POST | Tạo tài khoản admin | Public |
| Quên mật khẩu | `/user/forgot_password` | POST | Gửi email reset password | Public |
| Reset mật khẩu | `/reset-password` | POST | Đặt lại mật khẩu mới | Public |
| Xem profile | `/user/profile` | GET | Thông tin cá nhân | User |
| Đăng xuất User | `/user/logout` | POST | Đăng xuất user | User |
| Đăng xuất Admin | `/admin-logout` | POST | Đăng xuất admin | Admin |

##### **Product Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách sản phẩm (Admin) | `/admin/product` | GET | Quản lý sản phẩm | Admin |
| Thêm sản phẩm | `/admin/add-product` | POST | Tạo sản phẩm mới | Admin |
| Sửa sản phẩm | `/admin/edit-product` | POST | Cập nhật sản phẩm | Admin |
| Xóa sản phẩm | `/admin/delete-product/{id}` | GET | Xóa sản phẩm | Admin |
| Chi tiết sản phẩm (JSON) | `/api/admin/product/{id}` | GET | Lấy thông tin sản phẩm | Admin |
| Danh sách Laptop | `/laptops` | GET | Hiển thị laptop | Public |
| Danh sách Smartphone | `/smartphones` | GET | Hiển thị smartphone | Public |
| Danh sách Camera | `/cameras` | GET | Hiển thị camera | Public |
| Danh sách Accessories | `/accessories` | GET | Hiển thị phụ kiện | Public |
| Quick View | `/product/quickview/{id}` | GET | Xem nhanh sản phẩm | Public |
| Lọc Laptop | `/laptops/filter` | POST | Lọc theo brand | Public |
| Lọc Smartphone | `/smartphones/filter` | POST | Lọc theo brand | Public |

##### **Category Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách danh mục | `/admin/category` | GET | Quản lý danh mục | Admin |
| Thêm danh mục | `/admin/add-category` | POST | Tạo danh mục mới | Admin |
| Sửa danh mục | `/admin/edit-category` | POST | Cập nhật danh mục | Admin |
| Xóa danh mục | `/admin/delete-category/{id}` | GET | Xóa danh mục | Admin |
| Xem danh mục (User) | `/categories` | GET | Danh sách danh mục | Public |

##### **Cart Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Thêm vào giỏ | `/cart/add/{id}` | POST | Thêm sản phẩm vào giỏ | Public |
| Xem giỏ hàng | `/cart` | GET | Hiển thị giỏ hàng | Public |
| Cập nhật giỏ | `/cart/update/{id}` | POST | Cập nhật số lượng | Public |

##### **Order Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách đơn hàng (Admin) | `/admin/order` | GET | Quản lý đơn hàng | Admin |
| Chi tiết đơn hàng | `/admin/order/{id}` | GET | Xem chi tiết đơn | Admin |
| Xác nhận đơn | `/admin/order/confirm/{id}` | GET | Duyệt đơn hàng | Admin |
| Hủy đơn (Admin) | `/admin/order/cancel/{id}` | GET | Hủy đơn hàng | Admin |
| Cập nhật trạng thái | `/admin/order/updateStatus/{id}/{status}` | GET | Thay đổi trạng thái | Admin |
| Lịch sử đơn hàng (User) | `/user/orders` | GET | Xem đơn hàng của mình | User |
| Xác nhận đã nhận | `/orders/received/{id}` | GET | Đánh dấu đã nhận hàng | User |
| Hủy đơn (User) | `/orders/cancel/{id}` | GET | Hủy đơn chưa xác nhận | User |

##### **Review Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Gửi đánh giá | `/laptops/review` | POST | Đánh giá sản phẩm | Public |
| Danh sách review (Admin) | `/admin/reviews` | GET | Quản lý đánh giá | Admin |

##### **Banner Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách banner | `/admin/banner` | GET | Quản lý banner | Admin |
| Thêm banner | `/admin/add-banner` | POST | Tạo banner mới | Admin |
| Sửa banner | `/admin/edit-banner` | POST | Cập nhật banner | Admin |
| Xóa banner | `/admin/delete-banner/{id}` | GET | Xóa banner | Admin |

##### **Hot Deal Management**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách hot deal | `/admin/hotdeal` | GET | Quản lý khuyến mãi | Admin |
| Thêm hot deal | `/admin/add-hotdeal` | POST | Tạo khuyến mãi | Admin |
| Sửa hot deal | `/admin/edit-hotdeal/{id}` | POST | Cập nhật khuyến mãi | Admin |
| Xóa hot deal | `/admin/delete-hotdeal/{id}` | GET | Xóa khuyến mãi | Admin |
| Xem hot deal (User) | `/deals` | GET | Danh sách khuyến mãi | Public |

##### **Customer Management (Admin)**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Danh sách khách hàng | `/admin/customer` | GET | Quản lý khách hàng | Admin |
| Đơn hàng của khách | `/admin/customers/orders/{id}` | GET | Xem đơn hàng | Admin |
| Form khóa tài khoản | `/admin/customers/lock/{id}` | GET | Hiển thị form khóa | Admin |
| Khóa tài khoản | `/admin/customers/lock` | POST | Khóa tài khoản khách | Admin |
| Form mở khóa | `/admin/customers/unlock/{id}` | GET | Hiển thị form mở khóa | Admin |
| Mở khóa tài khoản | `/admin/customers/unlock` | POST | Mở khóa tài khoản | Admin |

##### **Dashboard & Statistics**

| **Tên API** | **Endpoint** | **HTTP Method** | **Mô tả** | **Quyền truy cập** |
|------------|-------------|----------------|-----------|-------------------|
| Dashboard | `/admin/dashboard` | GET | Thống kê tổng quan | Admin |
| Trang chủ | `/` | GET | Trang chủ website | Public |

---

### 6.5. Mô tả chi tiết API

#### 6.5.1. API User

##### **Mô tả nghiệp vụ**

API User quản lý toàn bộ vòng đời tài khoản người dùng:
- **Đăng ký:** Tạo tài khoản mới với validation
- **Đăng nhập:** Xác thực bằng username/password, tạo session
- **Quên mật khẩu:** Gửi email reset link
- **Quản lý profile:** Xem và cập nhật thông tin cá nhân

##### **Danh sách Endpoint**

###### **1. Đăng ký User**

**Endpoint:** `POST /user/register`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `username` | String | Có | Tên đăng nhập (unique) |
| `email` | String | Có | Email (unique, format email) |
| `password` | String | Có | Mật khẩu (min 6 ký tự) |
| `confirmPassword` | String | Có | Xác nhận mật khẩu |

**Response Body:**

```
Redirect: /user/login?registerSuccess=true
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 302 | Redirect thành công |
| 400 | Validation error (password không khớp, username/email đã tồn tại) |

**Ví dụ Request (Form Data):**
```http
POST /user/register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=john_doe&email=john@example.com&password=123456&confirmPassword=123456
```

**Ví dụ Response:**
```
HTTP/1.1 302 Found
Location: /user/login?registerSuccess=true
```

**Business Logic:**
```java
@PostMapping("/user/register")
public String handleRegister(
    @RequestParam("username") String username,
    @RequestParam("email") String email,
    @RequestParam("password") String password,
    @RequestParam("confirmPassword") String confirmPassword,
    RedirectAttributes redirect) {
    
    // Validation
    if (!password.equals(confirmPassword)) {
        redirect.addAttribute("error", "Mật khẩu không khớp");
        return "redirect:/user/register";
    }
    
    if (userService.findByUserName(username) != null) {
        redirect.addAttribute("error", "Tên đăng nhập đã tồn tại");
        return "redirect:/user/register";
    }
    
    // Tạo User
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setPassword(passwordEncoder.encode(password));
    user.setRole("ROLE_USER");
    user.setEnabled(true);
    
    // Tạo Customer (1-1 relationship)
    Customer customer = new Customer();
    customer.setName(username);
    customer.setEmail(email);
    customer.setCreated(new Date());
    customer.setUser(user);
    user.setCustomer(customer);
    
    userService.save(user);
    return "redirect:/user/login?registerSuccess=true";
}
```

###### **2. Đăng nhập User**

**Endpoint:** `POST /user/login`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `username` | String | Có | Tên đăng nhập hoặc email |
| `password` | String | Có | Mật khẩu |

**Response Body:**

```
Redirect: /?loginSuccess=true
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 302 | Đăng nhập thành công |
| 401 | Sai username/password |
| 403 | Tài khoản bị khóa |

**Ví dụ Request:**
```http
POST /user/login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=john_doe&password=123456
```

**Ví dụ Response (Success):**
```
HTTP/1.1 302 Found
Location: /?loginSuccess=true
Set-Cookie: JSESSIONID=ABC123...
```

**Ví dụ Response (Error):**
```
HTTP/1.1 302 Found
Location: /user/login?error=true
```

**Custom Authentication Failure Handler:**
```java
@Component
public class CustomAuthenticationFailureHandler 
    implements AuthenticationFailureHandler {
    
    @Override
    public void onAuthenticationFailure(
        HttpServletRequest request,
        HttpServletResponse response,
        AuthenticationException exception) throws IOException {
        
        String errorMessage = "Đăng nhập thất bại. Vui lòng thử lại.";
        
        if (exception instanceof DisabledException) {
            errorMessage = "Tài khoản đã bị khóa. Vui lòng liên hệ quản trị viên.";
        } else if (exception instanceof BadCredentialsException) {
            errorMessage = "Sai tên đăng nhập hoặc mật khẩu.";
        }
        
        request.getSession().setAttribute("loginError", true);
        request.getSession().setAttribute("errorMessage", errorMessage);
        response.sendRedirect("/user/login?error=true");
    }
}
```

###### **3. Quên mật khẩu**

**Endpoint:** `POST /user/forgot_password`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `email` | String | Có | Email đã đăng ký |

**Response Body:**

```html
Redirect: /user/forgot_password (with flash message)
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 302 | Đã gửi email (hoặc email không tồn tại) |

**Ví dụ Request:**
```http
POST /user/forgot_password HTTP/1.1
Content-Type: application/x-www-form-urlencoded

email=john@example.com
```

**Business Logic:**
```java
@PostMapping("/user/forgot_password")
public String handleForgotPassword(
    @RequestParam("email") String email,
    RedirectAttributes redirect) {
    
    User user = userService.findByEmail(email);
    if (user != null) {
        String resetLink = "http://localhost:8080/reset-password?email=" + email;
        emailService.sendEmail(email, "Reset Password Link", 
            "Click link để đổi mật khẩu: " + resetLink);
        redirect.addFlashAttribute("message", "Đã gửi email. Vui lòng kiểm tra hộp thư.");
    } else {
        redirect.addFlashAttribute("message", "Email không tồn tại.");
    }
    return "redirect:/user/forgot_password";
}
```

###### **4. Xem Profile**

**Endpoint:** `GET /user/profile`

**Request Parameters:** Không có (lấy từ Principal)

**Response Body:**

```html
Thymeleaf template với thông tin user
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thành công |
| 401 | Chưa đăng nhập |

**Ví dụ Request:**
```http
GET /user/profile HTTP/1.1
Cookie: JSESSIONID=ABC123...
```

**Business Logic:**
```java
@GetMapping("/user/profile")
public String showProfilePage(Model model, Principal principal) {
    User user = userService.findByUserName(principal.getName());
    if (user == null || user.getCustomer() == null) {
        return "redirect:/";
    }
    
    model.addAttribute("user", user);
    model.addAttribute("currentPage", "profile");
    return "user/profile";
}
```

---

#### 6.5.2. API Product

##### **Quản lý laptop & vật dụng**

###### **1. Lấy danh sách sản phẩm (Admin)**

**Endpoint:** `GET /admin/product`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** | **Mặc định** |
|------------|---------|-------------|-----------|-------------|
| `keyword` | String | Không | Từ khóa tìm kiếm | null |
| `pageNo` | Integer | Không | Số trang | 1 |

**Response Body:**

```html
Thymeleaf template với danh sách sản phẩm
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thành công |
| 401 | Chưa đăng nhập |
| 403 | Không có quyền admin |

**Ví dụ Request:**
```http
GET /admin/product?keyword=Dell&pageNo=1 HTTP/1.1
Cookie: JSESSIONID=ABC123...
```

**Business Logic:**
```java
@GetMapping("admin/product")
public String index(Model model, 
                    @Param("keyword") String keyword,
                    @RequestParam(name = "pageNo", defaultValue = "1") Integer pageNo) {
    
    Page<Product> listProduct;
    
    if (keyword != null && !keyword.isEmpty()) {
        listProduct = productService.searchByNameOrCategory(keyword, pageNo);
        model.addAttribute("keyword", keyword);
    } else {
        listProduct = productService.getAll(pageNo);
    }
    
    model.addAttribute("totalPage", listProduct.getTotalPages());
    model.addAttribute("currentPage", pageNo);
    model.addAttribute("listProduct", listProduct);
    return "admin/product/index";
}
```

###### **2. Lấy chi tiết sản phẩm (JSON API)**

**Endpoint:** `GET /api/admin/product/{id}`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `id` | Long | Có | ID sản phẩm |

**Response Body:**

```json
{
  "id": 123,
  "name": "Dell XPS 15",
  "price": 25000000.0,
  "image": "laptop.png",
  "description": "Laptop cao cấp...",
  "categoryName": "Laptops"
}
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thành công |
| 404 | Không tìm thấy sản phẩm |

**Ví dụ Request:**
```http
GET /api/admin/product/123 HTTP/1.1
Accept: application/json
```

**Ví dụ Response (Success):**
```json
{
  "id": 123,
  "name": "Dell XPS 15",
  "price": 25000000.0,
  "image": "laptop.png",
  "description": "Laptop cao cấp cho dân đồ họa",
  "categoryName": "Laptops"
}
```

**Ví dụ Response (Error):**
```json
{
  "message": "Không tìm thấy sản phẩm"
}
```

**Business Logic:**
```java
@GetMapping("/api/admin/product/{id}")
@ResponseBody
public ResponseEntity<?> getProductById(@PathVariable("id") Long id) {
    Optional<Product> productOpt = productService.findById(id);
    if (productOpt.isPresent()) {
        Product p = productOpt.get();
        return ResponseEntity.ok(new ProductDTO(p));
    } else {
        return ResponseEntity.status(404)
            .body("{\"message\": \"Không tìm thấy sản phẩm\"}");
    }
}
```

**ProductDTO:**
```java
public class ProductDTO {
    private Long id;
    private String name;
    private Double price;
    private String image;
    private String description;
    private String categoryName;
    
    public ProductDTO(Product p) {
        this.id = p.getId();
        this.name = p.getName();
        this.price = p.getPrice();
        this.image = p.getImage();
        this.description = p.getDescription();
        this.categoryName = (p.getCategory() != null) 
            ? p.getCategory().getName() : "";
    }
}
```

##### **Tìm kiếm – lọc sản phẩm**

###### **3. Danh sách Laptop với filter**

**Endpoint:** `GET /laptops`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** | **Mặc định** |
|------------|---------|-------------|-----------|-------------|
| `brands` | List<String> | Không | Danh sách brand (Dell, HP, Asus...) | null |
| `page` | Integer | Không | Số trang | 1 |
| `sort` | String | Không | Sắp xếp (name, price) | name |
| `show` | Integer | Không | Số sản phẩm/trang | 6 |

**Response Body:**

```html
Thymeleaf template với danh sách laptop
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thành công |

**Ví dụ Request:**
```http
GET /laptops?brands=Dell&brands=HP&page=1&sort=price&show=12 HTTP/1.1
```

**Business Logic:**
```java
@GetMapping("/laptops")
public String showLaptop(
    @RequestParam(name = "brands", required = false) List<String> brands,
    @RequestParam(name = "page", defaultValue = "1") int page,
    @RequestParam(name = "sort", defaultValue = "name") String sortField,
    @RequestParam(name = "show", defaultValue = "6") int pageSize,
    Model model) {
    
    if (page < 1) page = 1;
    int pageIndex = page - 1;
    
    Sort sort = switch (sortField.toLowerCase()) {
        case "price" -> Sort.by("price").ascending();
        case "name" -> Sort.by("name").ascending();
        default -> Sort.by("name").ascending();
    };
    
    Page<Product> laptopPage;
    Pageable pageable = PageRequest.of(pageIndex, pageSize, sort);
    
    if (brands != null && !brands.isEmpty()) {
        laptopPage = productService.findLaptopsByBrandsWithPageable(brands, pageable);
        model.addAttribute("brands", brands);
    } else {
        laptopPage = productService.findLaptopsWithPageable(pageable);
    }
    
    model.addAttribute("products", laptopPage.getContent());
    model.addAttribute("totalPages", laptopPage.getTotalPages());
    model.addAttribute("currentPage", page);
    model.addAttribute("topSelling", productService.findTop3Latest());
    model.addAttribute("selectedSort", sortField);
    model.addAttribute("selectedShow", pageSize);
    
    return "laptops";
}
```

**Repository Query:**
```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    Page<Product> findByCategory_NameIgnoreCase(String categoryName, Pageable pageable);
    
    Page<Product> findByCategory_NameIgnoreCaseAndBrandIn(
        String categoryName, 
        List<String> brands, 
        Pageable pageable
    );
}
```

##### **Ví dụ Request / Response**

**Request với nhiều filter:**
```http
GET /laptops?brands=Dell&brands=Asus&page=2&sort=price&show=12 HTTP/1.1
Accept: text/html
```

**Response (HTML Fragment):**
```html
<div class="product-list">
  <div class="product-item">
    <img src="/uploads/laptop.png" alt="Dell XPS 15">
    <h3>Dell XPS 15</h3>
    <p class="price">25.000.000 ₫</p>
    <button class="add-to-cart" data-id="123">Thêm vào giỏ</button>
  </div>
  <!-- More products... -->
</div>

<div class="pagination">
  <a href="/laptops?page=1">1</a>
  <a href="/laptops?page=2" class="active">2</a>
  <a href="/laptops?page=3">3</a>
</div>
```

---

#### 6.5.3. API Cart

##### **Thêm / cập nhật / xóa sản phẩm**

###### **1. Thêm sản phẩm vào giỏ hàng**

**Endpoint:** `POST /cart/add/{id}`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `id` | Long | Có | ID sản phẩm (path variable) |

**Response Body:**

```json
{
  "totalItems": 3
}
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thêm thành công |
| 404 | Không tìm thấy sản phẩm |

**Ví dụ Request:**
```http
POST /cart/add/123 HTTP/1.1
Content-Type: application/json
Cookie: JSESSIONID=ABC123...
```

**Ví dụ Response:**
```json
{
  "totalItems": 3
}
```

**Business Logic:**
```java
@RestController
@RequestMapping("/cart")
public class CartController {
    
    @Autowired
    private ProductService productService;
    
    @PostMapping("/add/{id}")
    public ResponseEntity<?> addToCart(
        @PathVariable Long id, 
        HttpSession session) {
        
        Cart cart = (Cart) session.getAttribute("cart");
        if (cart == null) {
            cart = new Cart();
        }
        
        Optional<Product> productOpt = productService.findById(id);
        if (productOpt.isPresent()) {
            Product product = productOpt.get();
            cart.addItem(product);
            session.setAttribute("cart", cart);
        }
        
        Map<String, Object> response = new HashMap<>();
        response.put("totalItems", cart.getTotalItems());
        
        return ResponseEntity.ok(response);
    }
}
```

**Cart Model:**
```java
public class Cart {
    private Map<Long, Integer> items = new HashMap<>();
    
    public void addItem(Product product) {
        Long id = product.getId();
        items.put(id, items.getOrDefault(id, 0) + 1);
    }
    
    public int getTotalItems() {
        return items.values().stream().mapToInt(i -> i).sum();
    }
    
    public Map<Long, Integer> getItems() {
        return items;
    }
}
```

##### **Tính tổng tiền**

**CartItem Model:**
```java
public class CartItem {
    private Long id;
    private String name;
    private String image;
    private Float price;
    private int quantity;
    
    public double getTotalPrice() {
        return price * quantity;
    }
}
```

**Tính tổng giỏ hàng:**
```java
public double calculateCartTotal(Cart cart, ProductService productService) {
    double total = 0.0;
    for (Map.Entry<Long, Integer> entry : cart.getItems().entrySet()) {
        Product product = productService.findById(entry.getKey()).orElse(null);
        if (product != null) {
            total += product.getPrice() * entry.getValue();
        }
    }
    return total;
}
```

##### **Ví dụ JSON**

**Request thêm sản phẩm:**
```json
POST /cart/add/123
```

**Response:**
```json
{
  "totalItems": 3,
  "message": "Đã thêm sản phẩm vào giỏ hàng"
}
```

**Cấu trúc Cart trong Session:**
```json
{
  "items": {
    "123": 2,
    "456": 1,
    "789": 3
  }
}
```

**Giải thích:**
- Key: Product ID
- Value: Số lượng

---

#### 6.5.4. API Order

##### **Tạo đơn hàng**

###### **1. Tạo đơn hàng mới**

**Endpoint:** `POST /orders/create`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `customerId` | Long | Có | ID khách hàng |
| `orderNote` | String | Không | Ghi chú đơn hàng |
| `items` | List<OrderItemDTO> | Có | Danh sách sản phẩm |

**Request Body (JSON):**
```json
{
  "customerId": 123,
  "orderNote": "Giao hàng giờ hành chính",
  "items": [
    {
      "productId": 456,
      "quantity": 2,
      "price": 25000000.0
    },
    {
      "productId": 789,
      "quantity": 1,
      "price": 15000000.0
    }
  ]
}
```

**Response Body:**
```json
{
  "orderId": 1001,
  "status": "Chờ xác nhận",
  "totalAmount": 65000000.0,
  "created": "2026-02-06T10:30:00"
}
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 201 | Tạo đơn thành công |
| 400 | Dữ liệu không hợp lệ |
| 404 | Không tìm thấy customer/product |

**Business Logic (Đề xuất):**
```java
@PostMapping("/orders/create")
public ResponseEntity<?> createOrder(
    @RequestBody CreateOrderRequest request,
    Principal principal) {
    
    // Lấy customer từ user đang đăng nhập
    User user = userService.findByUserName(principal.getName());
    Customer customer = user.getCustomer();
    
    // Tạo Order
    Orders order = new Orders();
    order.setCustomer(customer);
    order.setOrder_note(request.getOrderNote());
    order.setCreated(LocalDateTime.now());
    order.setStatus(1); // Chờ xác nhận
    
    // Tạo OrderDetails
    List<OrderDetail> details = new ArrayList<>();
    double totalAmount = 0.0;
    
    for (OrderItemDTO item : request.getItems()) {
        Product product = productService.findById(item.getProductId())
            .orElseThrow(() -> new NotFoundException("Product not found"));
        
        OrderDetail detail = new OrderDetail();
        detail.setOrder(order);
        detail.setProduct(product);
        detail.setQuantity(item.getQuantity());
        detail.setPrice(product.getPrice());
        
        details.add(detail);
        totalAmount += product.getPrice() * item.getQuantity();
    }
    
    order.setOrderDetails(details);
    Orders savedOrder = orderService.save(order);
    
    // Response
    Map<String, Object> response = new HashMap<>();
    response.put("orderId", savedOrder.getId());
    response.put("status", "Chờ xác nhận");
    response.put("totalAmount", totalAmount);
    response.put("created", savedOrder.getCreated());
    
    return ResponseEntity.status(201).body(response);
}
```

##### **Cập nhật trạng thái**

###### **2. Cập nhật trạng thái đơn hàng**

**Endpoint:** `GET /admin/order/updateStatus/{id}/{status}`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `id` | Long | Có | ID đơn hàng |
| `status` | Integer | Có | Trạng thái mới (1-7) |

**Response Body:**

```
Redirect: /admin/order (with flash message)
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 302 | Cập nhật thành công |
| 400 | Trạng thái không hợp lệ |
| 404 | Không tìm thấy đơn hàng |

**Ví dụ Request:**
```http
GET /admin/order/updateStatus/1001/2 HTTP/1.1
Cookie: JSESSIONID=ABC123...
```

**Business Logic:**
```java
@GetMapping("/updateStatus/{id}/{status}")
public String updateStatus(
    @PathVariable Long id, 
    @PathVariable int status, 
    RedirectAttributes ra) {
    
    if (status < 1 || status > 7) {
        ra.addFlashAttribute("error", "Trạng thái không hợp lệ.");
        return "redirect:/admin/order";
    }
    
    orderService.updateStatus(id, status);
    ra.addFlashAttribute("message", "Cập nhật trạng thái thành công.");
    return "redirect:/admin/order";
}
```

**Service Implementation:**
```java
@Override
public void updateStatus(Long id, int status) {
    Orders order = orderRepository.findById(id).orElse(null);
    if (order != null) {
        order.setStatus(status);
        orderRepository.save(order);
    } else {
        throw new RuntimeException("Không tìm thấy đơn hàng với ID: " + id);
    }
}
```

**Bảng trạng thái:**

| **Status** | **Tên trạng thái** | **Mô tả** |
|-----------|-------------------|-----------|
| 1 | Chờ xác nhận | Đơn hàng mới tạo |
| 2 | Đã xác nhận | Admin đã duyệt |
| 3 | Đang chuyển | Đang vận chuyển |
| 4 | Đang giao | Shipper đang giao |
| 5 | Đã hủy | Đơn bị hủy |
| 6 | Thành công | Giao hàng thành công |
| 7 | Yêu cầu hủy | Khách yêu cầu hủy |

##### **Truy vấn lịch sử đơn hàng**

###### **3. Lịch sử đơn hàng của User**

**Endpoint:** `GET /user/orders`

**Request Parameters:**

| **Tham số** | **Kiểu** | **Bắt buộc** | **Mô tả** |
|------------|---------|-------------|-----------|
| `status` | Integer | Không | Lọc theo trạng thái |

**Response Body:**

```html
Thymeleaf template với danh sách đơn hàng
```

**Status Code:**

| **Code** | **Mô tả** |
|---------|-----------|
| 200 | Thành công |
| 401 | Chưa đăng nhập |

**Ví dụ Request:**
```http
GET /user/orders?status=1 HTTP/1.1
Cookie: JSESSIONID=ABC123...
```

**Business Logic:**
```java
@GetMapping("/user/orders")
public String viewUserOrders(
    Model model,
    @RequestParam(value = "status", required = false) Integer status,
    Principal principal) {
    
    User user = userService.findByUserName(principal.getName());
    if (user == null || user.getCustomer() == null) {
        return "redirect:/";
    }
    
    List<Orders> orders = (status == null) 
        ? orderService.getOrderByUser(user.getCustomer())
        : orderService.getOrdersByUserAndStatus(user.getCustomer(), status);
    
    List<OrderViewModel> viewModels = orders.stream()
        .map(OrderViewModel::new)
        .toList();
    
    model.addAttribute("orders", viewModels);
    model.addAttribute("currentStatus", status);
    model.addAttribute("currentPage", "orders");
    return "user/orders";
}
```

**OrderViewModel:**
```java
public class OrderViewModel {
    private final Orders order;
    private final double totalPrice;
    
    public OrderViewModel(Orders order) {
        this.order = order;
        this.totalPrice = order.getOrderDetails().stream()
            .mapToDouble(d -> d.getPrice() * d.getQuantity())
            .sum();
    }
    
    public String getFormattedTotalPrice() {
        NumberFormat formatter = NumberFormat.getCurrencyInstance(new Locale("vi", "VN"));
        return formatter.format(totalPrice);
    }
    
    public String getStatusText() {
        return switch (order.getStatus()) {
            case 1 -> "Chờ xác nhận";
            case 2 -> "Đã xác nhận";
            case 3 -> "Đang chuyển";
            case 4 -> "Đang giao";
            case 5 -> "Đã huỷ";
            case 6 -> "Thành công";
            case 7 -> "Yêu cầu huỷ";
            default -> "Không rõ";
        };
    }
}
```

---

### 6.6. Luồng API tiêu biểu

#### 6.6.1. Luồng đăng nhập người dùng

```
┌─────────┐                ┌─────────┐                ┌─────────┐                ┌─────────┐
│ Browser │                │  Spring │                │ Security│                │Database │
│         │                │   MVC   │                │ Service │                │         │
└────┬────┘                └────┬────┘                └────┬────┘                └────┬────┘
     │                          │                          │                          │
     │  1. GET /user/login      │                          │                          │
     ├─────────────────────────>│                          │                          │
     │                          │                          │                          │
     │  2. Return login form    │                          │                          │
     │<─────────────────────────┤                          │                          │
     │                          │                          │                          │
     │  3. POST /user/login     │                          │                          │
     │     (username, password) │                          │                          │
     ├─────────────────────────>│                          │                          │
     │                          │                          │                          │
     │                          │  4. loadUserByUsername() │                          │
     │                          ├─────────────────────────>│                          │
     │                          │                          │                          │
     │                          │                          │  5. SELECT * FROM users  │
     │                          │                          ├─────────────────────────>│
     │                          │                          │                          │
     │                          │                          │  6. User data            │
     │                          │                          │<─────────────────────────┤
     │                          │                          │                          │
     │                          │  7. UserDetails          │                          │
     │                          │<─────────────────────────┤                          │
     │                          │                          │                          │
     │                          │  8. Verify password      │                          │
     │                          │     (BCrypt)             │                          │
     │                          ├──────────┐               │                          │
     │                          │          │               │                          │
     │                          │<─────────┘               │                          │
     │                          │                          │                          │
     │  9. 302 Redirect         │                          │                          │
     │     Set-Cookie: JSESSIONID                          │                          │
     │<─────────────────────────┤                          │                          │
     │                          │                          │                          │
     │  10. GET /?loginSuccess=true                        │                          │
     ├─────────────────────────>│                          │                          │
     │                          │                          │                          │
     │  11. Home page           │                          │                          │
     │<─────────────────────────┤                          │                          │
     │                          │                          │                          │
```

**Chi tiết từng bước:**

1. **User truy cập trang login**
2. **Server trả về form đăng nhập**
3. **User submit form với username & password**
4. **Spring Security gọi CustomUserDetailService.loadUserByUsername()**
5. **Query database để lấy thông tin user**
6. **Trả về User entity**
7. **Tạo CustomUserDetails object**
8. **Verify password bằng BCrypt**
9. **Tạo session và redirect về trang chủ**
10. **Browser request trang chủ với session cookie**
11. **Hiển thị trang chủ với trạng thái đã đăng nhập**

**Code Implementation:**

```java
// Step 4-7: CustomUserDetailService
@Override
public UserDetails loadUserByUsername(String username) 
    throws UsernameNotFoundException {
    User user = userService.findByUserName(username);
    if (user == null) {
        throw new UsernameNotFoundException("Tên đăng nhập không tồn tại");
    }
    
    Collection<GrantedAuthority> authorities = new HashSet<>();
    authorities.add(new SimpleGrantedAuthority("ROLE_" + user.getRole()));
    
    return new CustomUserDetails(user, authorities);
}

// Step 8: Password verification (automatic by Spring Security)
DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
provider.setPasswordEncoder(passwordEncoder); // BCrypt
```

#### 6.6.2. Luồng thêm sản phẩm vào giỏ hàng

```
┌─────────┐          ┌─────────┐          ┌─────────┐          ┌─────────┐
│ Browser │          │   Cart  │          │ Product │          │Database │
│         │          │   API   │          │ Service │          │         │
└────┬────┘          └────┬────┘          └────┬────┘          └────┬────┘
     │                    │                    │                    │
     │  1. Click "Add to Cart"                │                    │
     │     (Product ID: 123)                  │                    │
     ├───────────────────>│                    │                    │
     │                    │                    │                    │
     │  2. POST /cart/add/123                 │                    │
     ├───────────────────>│                    │                    │
     │                    │                    │                    │
     │                    │  3. Get cart from session              │
     │                    ├──────────┐         │                    │
     │                    │          │         │                    │
     │                    │<─────────┘         │                    │
     │                    │                    │                    │
     │                    │  4. findById(123)  │                    │
     │                    ├───────────────────>│                    │
     │                    │                    │                    │
     │                    │                    │  5. SELECT * FROM products
     │                    │                    ├───────────────────>│
     │                    │                    │                    │
     │                    │                    │  6. Product data   │
     │                    │                    │<───────────────────┤
     │                    │                    │                    │
     │                    │  7. Product object │                    │
     │                    │<───────────────────┤                    │
     │                    │                    │                    │
     │                    │  8. cart.addItem(product)              │
     │                    ├──────────┐         │                    │
     │                    │          │         │                    │
     │                    │<─────────┘         │                    │
     │                    │                    │                    │
     │                    │  9. Save cart to session               │
     │                    ├──────────┐         │                    │
     │                    │          │         │                    │
     │                    │<─────────┘         │                    │
     │                    │                    │                    │
     │  10. JSON Response │                    │                    │
     │      {"totalItems": 3}                  │                    │
     │<───────────────────┤                    │                    │
     │                    │                    │                    │
     │  11. Update cart badge                  │                    │
     │      (3 items)     │                    │                    │
     ├──────────┐         │                    │                    │
     │          │         │                    │                    │
     │<─────────┘         │                    │                    │
     │                    │                    │                    │
```

**Code Implementation:**

```java
// Step 2-10: CartController
@PostMapping("/cart/add/{id}")
public ResponseEntity<?> addToCart(@PathVariable Long id, HttpSession session) {
    // Step 3: Get cart from session
    Cart cart = (Cart) session.getAttribute("cart");
    if (cart == null) {
        cart = new Cart();
    }
    
    // Step 4-7: Find product
    Optional<Product> productOpt = productService.findById(id);
    if (productOpt.isPresent()) {
        Product product = productOpt.get();
        
        // Step 8: Add to cart
        cart.addItem(product);
        
        // Step 9: Save to session
        session.setAttribute("cart", cart);
    }
    
    // Step 10: Return response
    Map<String, Object> response = new HashMap<>();
    response.put("totalItems", cart.getTotalItems());
    return ResponseEntity.ok(response);
}

// Step 11: Frontend JavaScript
function addToCart(productId) {
    fetch(`/cart/add/${productId}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        }
    })
    .then(response => response.json())
    .then(data => {
        // Update cart badge
        document.getElementById('cart-count').textContent = data.totalItems;
        alert('Đã thêm vào giỏ hàng!');
    });
}
```

#### 6.6.3. Luồng đặt hàng và thanh toán

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ Browser │     │  Order  │     │ Product │     │  Email  │     │Database │
│         │     │   API   │     │ Service │     │ Service │     │         │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │               │
     │  1. View Cart │               │               │               │
     ├──────────────>│               │               │               │
     │               │               │               │               │
     │  2. Click "Checkout"          │               │               │
     ├──────────────>│               │               │               │
     │               │               │               │               │
     │  3. POST /orders/create       │               │               │
     │     (cart items, customer info)               │               │
     ├──────────────>│               │               │               │
     │               │               │               │               │
     │               │  4. Validate cart items       │               │
     │               ├──────────────>│               │               │
     │               │               │               │               │
     │               │  5. Check stock               │               │
     │               │               ├──────────────────────────────>│
     │               │               │               │               │
     │               │               │  6. Stock data│               │
     │               │               │<──────────────────────────────┤
     │               │               │               │               │
     │               │  7. Products OK               │               │
     │               │<──────────────┤               │               │
     │               │               │               │               │
     │               │  8. Create Order entity       │               │
     │               ├──────────┐    │               │               │
     │               │          │    │               │               │
     │               │<─────────┘    │               │               │
     │               │               │               │               │
     │               │  9. Create OrderDetails       │               │
     │               ├──────────┐    │               │               │
     │               │          │    │               │               │
     │               │<─────────┘    │               │               │
     │               │               │               │               │
     │               │  10. BEGIN TRANSACTION        │               │
     │               ├──────────────────────────────────────────────>│
     │               │               │               │               │
     │               │  11. INSERT INTO orders       │               │
     │               ├──────────────────────────────────────────────>│
     │               │               │               │               │
     │               │  12. INSERT INTO order_details│               │
     │               ├──────────────────────────────────────────────>│
     │               │               │               │               │
     │               │  13. UPDATE inventory         │               │
     │               ├──────────────────────────────────────────────>│
     │               │               │               │               │
     │               │  14. COMMIT   │               │               │
     │               ├──────────────────────────────────────────────>│
     │               │               │               │               │
     │               │  15. Send confirmation email  │               │
     │               ├──────────────────────────────>│               │
     │               │               │               │               │
     │               │               │               │  16. Send email
     │               │               │               ├──────────┐    │
     │               │               │               │          │    │
     │               │               │               │<─────────┘    │
     │               │               │               │               │
     │               │  17. Clear cart from session  │               │
     │               ├──────────┐    │               │               │
     │               │          │    │               │               │
     │               │<─────────┘    │               │               │
     │               │               │               │               │
     │  18. 302 Redirect             │               │               │
     │      /user/orders?success=true│               │               │
     │<──────────────┤               │               │               │
     │               │               │               │               │
     │  19. Show success message     │               │               │
     │<──────────────┤               │               │               │
     │               │               │               │               │
```

**Code Implementation (Đề xuất):**

```java
@PostMapping("/orders/create")
@Transactional
public String createOrder(
    @RequestParam("orderNote") String orderNote,
    HttpSession session,
    Principal principal,
    RedirectAttributes ra) {
    
    // Step 3: Get cart and user
    Cart cart = (Cart) session.getAttribute("cart");
    if (cart == null || cart.getItems().isEmpty()) {
        ra.addFlashAttribute("error", "Giỏ hàng trống");
        return "redirect:/cart";
    }
    
    User user = userService.findByUserName(principal.getName());
    Customer customer = user.getCustomer();
    
    // Step 8: Create Order
    Orders order = new Orders();
    order.setCustomer(customer);
    order.setOrder_note(orderNote);
    order.setCreated(LocalDateTime.now());
    order.setStatus(1); // Chờ xác nhận
    
    // Step 9: Create OrderDetails
    List<OrderDetail> details = new ArrayList<>();
    double totalAmount = 0.0;
    
    for (Map.Entry<Long, Integer> entry : cart.getItems().entrySet()) {
        // Step 4-7: Validate product
        Product product = productService.findById(entry.getKey())
            .orElseThrow(() -> new NotFoundException("Product not found"));
        
        OrderDetail detail = new OrderDetail();
        detail.setOrder(order);
        detail.setProduct(product);
        detail.setQuantity(entry.getValue());
        detail.setPrice(product.getPrice());
        
        details.add(detail);
        totalAmount += product.getPrice() * entry.getValue();
    }
    
    order.setOrderDetails(details);
    
    // Step 10-14: Save to database (Transaction)
    Orders savedOrder = orderService.save(order);
    
    // Step 15-16: Send email
    emailService.sendEmail(
        customer.getEmail(),
        "Xác nhận đơn hàng #" + savedOrder.getId(),
        "Đơn hàng của bạn đã được tạo thành công. Tổng tiền: " + totalAmount
    );
    
    // Step 17: Clear cart
    session.removeAttribute("cart");
    
    // Step 18-19: Redirect
    ra.addFlashAttribute("success", "Đặt hàng thành công!");
    return "redirect:/user/orders?success=true";
}
```

---

### 6.7. Xử lý lỗi & Exception

#### 6.7.1. Các lỗi phổ biến

| **Loại lỗi** | **HTTP Code** | **Nguyên nhân** | **Xử lý** |
|--------------|--------------|----------------|-----------|
| **Validation Error** | 400 | Dữ liệu đầu vào không hợp lệ | Trả về thông báo lỗi cụ thể |
| **Authentication Error** | 401 | Chưa đăng nhập | Redirect về trang login |
| **Authorization Error** | 403 | Không có quyền truy cập | Hiển thị trang 403 |
| **Not Found** | 404 | Resource không tồn tại | Trả về thông báo "Không tìm thấy" |
| **Duplicate Entry** | 409 | Username/Email đã tồn tại | Thông báo trùng lặp |
| **Server Error** | 500 | Lỗi server (DB, logic) | Log error, trả về thông báo chung |

#### 6.7.2. Cấu trúc Response lỗi

**Cấu trúc chuẩn:**
```json
{
  "status": "error",
  "code": 400,
  "message": "Dữ liệu không hợp lệ",
  "errors": [
    {
      "field": "email",
      "message": "Email đã tồn tại"
    },
    {
      "field": "password",
      "message": "Mật khẩu phải có ít nhất 6 ký tự"
    }
  ],
  "timestamp": "2026-02-06T10:30:00"
}
```

**Global Exception Handler (Đề xuất):**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<?> handleNotFoundException(NotFoundException ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("status", "error");
        error.put("code", 404);
        error.put("message", ex.getMessage());
        error.put("timestamp", LocalDateTime.now());
        
        return ResponseEntity.status(404).body(error);
    }
    
    @ExceptionHandler(DuplicateException.class)
    public ResponseEntity<?> handleDuplicateException(DuplicateException ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("status", "error");
        error.put("code", 409);
        error.put("message", ex.getMessage());
        error.put("timestamp", LocalDateTime.now());
        
        return ResponseEntity.status(409).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handleGeneralException(Exception ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("status", "error");
        error.put("code", 500);
        error.put("message", "Đã xảy ra lỗi. Vui lòng thử lại sau.");
        error.put("timestamp", LocalDateTime.now());
        
        // Log error
        ex.printStackTrace();
        
        return ResponseEntity.status(500).body(error);
    }
}
```

#### 6.7.3. Ví dụ lỗi thực tế

##### **Lỗi 1: Sản phẩm không tồn tại**

**Request:**
```http
GET /api/admin/product/999999 HTTP/1.1
```

**Response:**
```json
{
  "message": "Không tìm thấy sản phẩm"
}
```

**Code:**
```java
@GetMapping("/api/admin/product/{id}")
@ResponseBody
public ResponseEntity<?> getProductById(@PathVariable("id") Long id) {
    Optional<Product> productOpt = productService.findById(id);
    if (productOpt.isPresent()) {
        return ResponseEntity.ok(new ProductDTO(productOpt.get()));
    } else {
        return ResponseEntity.status(404)
            .body("{\"message\": \"Không tìm thấy sản phẩm\"}");
    }
}
```

##### **Lỗi 2: Username đã tồn tại**

**Request:**
```http
POST /user/register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=john_doe&email=new@example.com&password=123456&confirmPassword=123456
```

**Response:**
```
HTTP/1.1 302 Found
Location: /user/register?error=Tên đăng nhập đã tồn tại
```

**Code:**
```java
if (userService.findByUserName(username) != null) {
    redirect.addAttribute("error", "Tên đăng nhập đã tồn tại");
    return "redirect:/user/register";
}
```

##### **Lỗi 3: Tài khoản bị khóa**

**Request:**
```http
POST /user/login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=locked_user&password=123456
```

**Response:**
```
HTTP/1.1 302 Found
Location: /user/login?error=true
```

**Session Attribute:**
```
errorMessage: "Tài khoản đã bị khóa. Vui lòng liên hệ quản trị viên."
```

**Code:**
```java
@Override
public void onAuthenticationFailure(
    HttpServletRequest request,
    HttpServletResponse response,
    AuthenticationException exception) throws IOException {
    
    String errorMessage = "Đăng nhập thất bại. Vui lòng thử lại.";
    
    if (exception instanceof DisabledException) {
        errorMessage = "Tài khoản đã bị khóa. Vui lòng liên hệ quản trị viên.";
    } else if (exception instanceof BadCredentialsException) {
        errorMessage = "Sai tên đăng nhập hoặc mật khẩu.";
    }
    
    request.getSession().setAttribute("loginError", true);
    request.getSession().setAttribute("errorMessage", errorMessage);
    response.sendRedirect("/user/login?error=true");
}
```

##### **Lỗi 4: Validation Error**

**Request:**
```http
POST /user/register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=john&email=invalid-email&password=123&confirmPassword=456
```

**Response (Đề xuất):**
```json
{
  "status": "error",
  "code": 400,
  "message": "Dữ liệu không hợp lệ",
  "errors": [
    {
      "field": "email",
      "message": "Email không đúng định dạng"
    },
    {
      "field": "password",
      "message": "Mật khẩu phải có ít nhất 6 ký tự"
    },
    {
      "field": "confirmPassword",
      "message": "Mật khẩu xác nhận không khớp"
    }
  ]
}
```

---

### 6.8. Tóm tắt phần thiết kế API

#### **Điểm mạnh của hệ thống**

| **Khía cạnh** | **Đánh giá** | **Chi tiết** |
|--------------|-------------|-------------|
| **Bảo mật** | ⭐⭐⭐⭐⭐ | Spring Security với 2 SecurityFilterChain riêng biệt, BCrypt password |
| **Phân tách Admin-User** | ⭐⭐⭐⭐⭐ | Session context riêng, endpoint rõ ràng |
| **Phân trang** | ⭐⭐⭐⭐ | Hỗ trợ pagination cho hầu hết danh sách |
| **Tìm kiếm & Lọc** | ⭐⭐⭐⭐ | Filter theo brand, category, keyword |
| **Upload File** | ⭐⭐⭐⭐ | UUID naming, lưu local storage |
| **Email Service** | ⭐⭐⭐ | Gửi email reset password |

#### **Điểm cần cải thiện**

| **Vấn đề** | **Mức độ** | **Đề xuất** |
|-----------|-----------|------------|
| **Không tuân thủ REST thuần túy** | Trung bình | Sử dụng `@DeleteMapping` thay vì `@GetMapping` cho delete |
| **Thiếu API versioning** | Thấp | Thêm `/api/v1/` prefix |
| **Thiếu chuẩn hóa Response** | Trung bình | Tạo ResponseWrapper chung |
| **Thiếu Global Exception Handler** | Cao | Implement `@ControllerAdvice` |
| **Thiếu API Documentation** | Cao | Tích hợp Swagger/OpenAPI |
| **Thiếu Rate Limiting** | Trung bình | Thêm rate limiter cho API |
| **Thiếu API Payment** | Cao | Tích hợp VNPay, Momo |

#### **Tổng kết số liệu**

```
📊 THỐNG KÊ API

Tổng số Endpoints:        57
├─ Authentication:         9
├─ Product Management:    12
├─ Category Management:    5
├─ Cart Management:        3
├─ Order Management:       8
├─ Review Management:      3
├─ Banner Management:      5
├─ Hot Deal Management:    5
├─ Customer Management:    6
└─ Dashboard:              1

Phương thức HTTP:
├─ GET:                   42 (73.7%)
├─ POST:                  15 (26.3%)
├─ PUT:                    0 (0%)
└─ DELETE:                 0 (0%)

Phân quyền:
├─ Public:                25 (43.9%)
├─ User:                  12 (21.1%)
└─ Admin:                 20 (35.0%)
```

#### **Roadmap phát triển API**

**Phase 1 - Cải thiện hiện tại (1-2 tháng):**
- [ ] Implement Global Exception Handler
- [ ] Chuẩn hóa Response format
- [ ] Thêm API Documentation (Swagger)
- [ ] Sửa HTTP methods cho đúng REST

**Phase 2 - Tính năng mới (2-3 tháng):**
- [ ] API Payment (VNPay, Momo)
- [ ] API Wishlist
- [ ] API Shipping Address
- [ ] API Coupon/Discount

**Phase 3 - Tối ưu (3-4 tháng):**
- [ ] API Versioning
- [ ] Rate Limiting
- [ ] Caching (Redis)
- [ ] API Analytics

**Phase 4 - Mở rộng (4-6 tháng):**
- [ ] GraphQL API
- [ ] WebSocket (Real-time notifications)
- [ ] Mobile API optimization
- [ ] Microservices architecture

---

## PHỤ LỤC

### A. Danh sách Entity và Relationship

```
User (1) ←→ (1) Customer
Customer (1) ←→ (N) Orders
Orders (1) ←→ (N) OrderDetail
Product (1) ←→ (N) OrderDetail
Category (1) ←→ (N) Product
Product (1) ←→ (N) Review
Customer (1) ←→ (N) Review
Product (1) ←→ (N) HotDeal
```

### B. Cấu trúc Database chính

**Bảng users:**
```sql
CREATE TABLE users (
    id NUMBER PRIMARY KEY,
    username VARCHAR2(50) UNIQUE,
    email VARCHAR2(100) UNIQUE,
    password VARCHAR2(255),
    role VARCHAR2(20),
    enabled NUMBER(1),
    fullname VARCHAR2(100)
);
```

**Bảng products:**
```sql
CREATE TABLE products (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(200),
    price NUMBER(10,2),
    image VARCHAR2(255),
    description CLOB,
    brand VARCHAR2(50),
    status NUMBER(1),
    category_id NUMBER,
    created DATE,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

### C. Environment Variables

```properties
# Database
spring.datasource.url=jdbc:oracle:thin:@//localhost:1521/orcld
spring.datasource.username=system
spring.datasource.password=1234567a@

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Thymeleaf
spring.thymeleaf.cache=false
spring.thymeleaf.encoding=UTF-8

# File Upload
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

---

**KẾT THÚC TÀI LIỆU**

*Tài liệu này được tạo tự động từ source code và có thể được cập nhật theo phiên bản mới của hệ thống.*

**Liên hệ:**
- Email: dev@ecommerce.com
- Slack: #api-support
- Wiki: https://wiki.ecommerce.com/api

**Lịch sử phiên bản:**
- v1.0.0 (06/02/2026): Phiên bản đầu tiên

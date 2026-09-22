# Hardware Store Inventory Management System

> **สถานะ:** 🟡 อยู่ในขั้นตอนวางแผน (Planning Phase) — ยังไม่เริ่ม implement

ระบบจัดการสินค้าและคลังสินค้าสำหรับร้านอุปกรณ์การช่าง ครอบคลุมตั้งแต่การจัดการสินค้า/หมวดหมู่/ซัพพลายเออร์ การรับสินค้าเข้าคลัง การขายสินค้า การตรวจสอบ Stock ไปจนถึงการแจ้งเตือนสินค้าใกล้หมด

Concept หลัก:
```
Product → Inventory → Purchase/Sale → Stock Movement
```

---

## 1. ฟีเจอร์หลัก (Core Features)

- จัดการข้อมูลสินค้า (Product)
- จัดการหมวดหมู่สินค้า (Category)
- จัดการ Supplier
- รับสินค้าเข้าคลัง (Purchase Order)
- ขาย/ตัดสินค้าออกจากคลัง (Sales Order)
- ตรวจสอบจำนวน Stock + ประวัติการเคลื่อนไหว (Stock Movement)
- แจ้งเตือนสินค้าใกล้หมด (Low Stock Notification)
- จัดการลูกค้า (Customer)
- ดูประวัติการซื้อขาย
- รายงาน Stock / Dashboard
- Authentication/Role (optional — เพิ่มทีหลังถ้ามีเวลา)

---

## 2. Tech Stack (แผน)

| ส่วน | เทคโนโลยี |
|---|---|
| Backend | Spring Boot (REST API) |
| Frontend | React |
| Database | PostgreSQL / MySQL |
| ORM | Spring Data JPA |
| Docs API | Swagger / OpenAPI |
| Testing | JUnit + Mockito |
| Deployment | Docker / docker-compose (+ GitHub Actions ถ้ามีเวลา) |

---

## 3. Database Design (ร่าง)

วางแผนไว้ประมาณ 9–10 ตาราง ครอบคลุมทั้งความสัมพันธ์แบบ 1:1 และ 1:N

**ตารางหลัก:** `categories`, `products`, `inventory_stocks`, `suppliers`, `customers`, `sales_orders`, `sales_order_items`, `purchase_orders`, `purchase_items`, `stock_movements`

**ความสัมพันธ์สำคัญ:**
- `Product` 1:1 `InventoryStock`
- `Category` 1:N `Product`
- `Customer` 1:N `SalesOrder` 1:N `SalesOrderItem`
- `Supplier` 1:N `PurchaseOrder` 1:N `PurchaseItem`

รายละเอียดเต็มดูที่ `doc/data-dictionary.md` (ยังไม่ได้เขียน)

---

## 4. Architecture (แผน)

```
React Frontend
      │  REST/JSON
      ▼
Controller → Service (Strategy / State / Transaction) → Repository → DB
                       │
                  Observer/Event → Low Stock Notification
```

Layered Architecture: `controller → service → repository → domain`
พร้อม `dto/`, `mapper/`, `strategy/`, `state/`, `event/`, `exception/` แยกความรับผิดชอบชัดเจน

---

## 5. Design Patterns ที่วางแผนใช้ (≥3 ตัว)

| Pattern | ใช้กับ | เหตุผล |
|---|---|---|
| **Strategy** | การคำนวณส่วนลด (`DiscountStrategy`) | เพิ่ม/แก้เงื่อนไขส่วนลดได้โดยไม่แตะ business logic เดิม (OCP) |
| **State** | สถานะ `SalesOrder` (PENDING → CONFIRMED → COMPLETED / CANCELLED) | ลด if/else ของ status ในโค้ด |
| **Observer** | แจ้งเตือน Low Stock (`StockChangedEvent`) | Decouple ระหว่าง Stock update กับ Notification |

---

## 6. SOLID Principles (แผนการอธิบาย)

- **S** — แยก Controller / Service / Repository / Mapper ชัดเจน
- **O** — `DiscountStrategy` เพิ่ม strategy ใหม่ได้โดยไม่แก้ `OrderService`
- **L** — ทุก implementation ของ `DiscountStrategy` / `OrderState` แทนกันได้
- **I** — แยก Service ตามโดเมน (Product/Order/Inventory/...) แทนที่จะรวมเป็น interface ใหญ่
- **D** — `OrderService` พึ่งพา interface `DiscountStrategy` ผ่าน constructor injection

---

## 7. โครงสร้าง Repository (แผน)

```
Hardware_Store_Inventory_Management_System/
├── README.md
├── code/
│   ├── backend/        # Spring Boot
│   └── frontend/       # React
├── test/
│   ├── backend/
│   └── frontend/
├── doc/
│   ├── diagrams/        # use-case, ER, sequence, state, component, deployment
│   ├── solid-analysis.md
│   ├── design-patterns.md
│   ├── data-dictionary.md
│   ├── api-documentation.md
│   └── slide/
└── img/
```

---

## 8. REST API (ร่างเบื้องต้น)

ตัวอย่าง endpoint หลัก (เวอร์ชันเต็มดูที่ `doc/api-documentation.md`):

```
GET/POST/PUT/DELETE  /api/v1/products
GET/POST/PUT/DELETE  /api/v1/categories
GET/POST/PUT/DELETE  /api/v1/customers
POST                 /api/v1/orders          # ขายสินค้า → ลด stock
POST                 /api/v1/purchases       # รับสินค้าเข้า → เพิ่ม stock
GET                  /api/v1/inventory
GET                  /api/v1/inventory/low-stock
GET                  /api/v1/products/{id}/stock-movements
```

รองรับ pagination/sorting (`?page=&size=&sort=`) และ validation ผ่าน `@Valid` + Global Exception Handler

---

## 9. Scope

| ระดับ | รายการ |
|---|---|
| **Must Have** | Product, Category, Inventory, Supplier, Customer, Sales Order, Purchase Order, Stock Movement, REST API, React UI, Swagger, Unit Test, Docker |
| **Should Have** | Low Stock Notification, Dashboard, Search, Pagination/Sorting, Order State, Discount Strategy |
| **Optional** | Authentication/Role, Reports, CSV Export, Many-to-Many, CI/CD, Email Notification |

> ถ้าเวลาจำกัด: ตัด Optional ก่อน — **ห้ามตัด** Testing / Design Pattern / Diagram / Deployment

---

## 10. Members

| No. | Name | Student ID | Section | Branch | Responsibility |
|---|---|---|---|---|---|
| 1 | นายพิสิษฐ์ ทรัพย์อุดมโชติ | 673380285-2 | 01 |  |  |
| 2 | นายพีรพล แก้วเจริญสันติสุข | 673380287-8 | 01 |  |  |
| 3 | นายพีรพัฒน์ แท่นประยุทร | 673380288-6 | 01 |  |  |


**Branch strategy:**
```
main ← develop ← membername_รหัสนักศึกษา_01
```

---

## 11. แผนงานถัดไป (Next Steps)

1. ล็อก Database / ER Diagram
2. ออกแบบ Domain Model
3. กำหนด API Contract (request/response DTO)
4. Package / Class Diagram
5. แบ่งงาน 5 คน แล้วเริ่ม implement

---

## 12. เอกสารที่เกี่ยวข้อง (ยังไม่ได้เขียน)

- `doc/solid-analysis.md`
- `doc/design-patterns.md`
- `doc/data-dictionary.md`
- `doc/api-documentation.md`
- `doc/diagrams/` (use-case, ER, sequence ×3, state, component, deployment)
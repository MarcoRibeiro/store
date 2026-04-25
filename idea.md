# 🛍️ Reconditioned Tech E-commerce — Product Specification

## 1. Overview

This project consists of building an e-commerce platform for selling reconditioned technology products (e.g., smartphones, laptops, consoles).

The goal is to launch a functional MVP quickly that allows:
- listing and selling products
- managing catalog and orders
- validating the business model
- enabling incremental evolution

---

## 2. Goals

### Primary Goals
- Enable online sales of reconditioned tech products
- Clearly communicate product condition (grading system)
- Keep MVP simple and fast to launch
- Ensure the system is extensible

### Non-Goals (MVP)
- Full ERP integration (e.g., Primavera)
- Advanced marketing tools
- Complex inventory management
- Multi-language / multi-currency support

---

## 3. Actors

### Customer
- browses products
- views product details
- places orders
- views order history

### Admin
- manages products
- manages images
- manages categories
- manages orders

---

## 4. Core Domains

- Products
- Categories
- Orders
- Users
- Media (Images)
- Payments (manual for MVP)

---

## 5. Functional Requirements

## 5.1 Products

### Product Attributes
- id
- title
- description
- category_id
- price (includes shipping)
- grade (A, B, C)
- serial_number
- stock_status (available / sold)
- created_at

### Product Media
- multiple images per product
- ordered list
- one primary image

---

## 5.2 Categories

- id
- name
- active (boolean)

---

## 5.3 Product Listing

### Requirements
- filter by category
- sort by:
  - price
  - newest

### Display
- primary image
- title
- price
- grade

---

## 5.4 Product Detail Page

### Must Include
- title
- description
- price
- grade
- serial number
- image gallery
- availability

### Behaviour
- image zoom / preview
- clear explanation of product grading

---

## 5.5 Cart & Checkout

### Cart
- add/remove product
- list items

### Checkout
- collect:
  - name
  - email
  - phone
  - address
  - postal code
  - optional tax ID (NIF)

- confirm order

---

## 5.6 Payments (MVP)

### Methods
- MB Way
- Bank Transfer

### Notes
- manual validation allowed
- admin updates order status after payment confirmation

---

## 5.7 Orders

### Order Attributes
- id
- customer data (snapshot)
- items
- total_price
- payment_method
- status
- created_at

### Status
- pending
- payment_pending
- paid
- shipped
- completed
- cancelled

---

## 5.8 User Accounts

### Customer
- register / login
- view orders
- update personal data

### Authentication
- email/password OR social login (Google)

---

## 5.9 Admin Panel

### Product Management
- create / edit / delete products
- manage images (upload, order)
- assign category
- set grade
- set serial number
- mark as featured

### Category Management
- create / edit / delete categories

### Order Management
- view orders
- update order status

---

## 5.10 Media Management

### Requirements
- multi-image upload
- ordering
- preview
- deletion

### Storage
- external object storage (S3-compatible)

---

## 6. Non-Functional Requirements

## 6.1 Performance
- fast page load
- optimized images
- CDN for media delivery

## 6.2 Security
- HTTPS required
- hashed passwords
- input validation
- role-based access control

## 6.3 Availability
- single-node setup acceptable for MVP
- regular backups required

## 6.4 Scalability
- modular architecture
- separation of services (API, database, storage)

---

## 7. Infrastructure (MVP)

### Recommended Setup
- VPS (e.g., Hetzner or equivalent)
- PostgreSQL database
- Object storage (S3-compatible)
- CDN for images

---

## 8. GDPR Compliance

### Requirements
- privacy policy
- cookie consent banner
- minimal data collection
- secure data storage

### User Rights (Future)
- data export
- account deletion

---

## 9. Pricing Model

- product price includes shipping
- fixed shipping cost embedded in price

---

## 10. Design Guidelines

### Style
- clean and modern
- product-focused UI

### Colors
- blue
- white
- black

---

## 11. Future Enhancements

- payment gateway integration
- search and advanced filters
- product reviews
- promotions and coupons
- ERP integration (Primavera)
- shipping integrations
- analytics and reporting

---

## 12. Open Questions

- final category structure
- MB Way payment flow (manual vs API)
- invoicing flow (manual vs automated)
- shipping provider
- return policy

---

## 13. Success Criteria (MVP)

- products can be created and managed
- users can place orders
- admin can process orders
- system is stable and usable
- ready for iteration and scaling

# Storefront MVP Design

Date: 2026-04-26
Topic: Reconditioned tech e-commerce storefront MVP
Status: Drafted from approved brainstorming session

## 1. Goal

Design the first customer-facing slice of a reconditioned tech e-commerce platform.

This MVP storefront should help cautious buyers trust the store quickly, browse inventory easily, and place an order using manual payment methods without requiring an account.

## 2. Scope

Included in this slice:

- Homepage
- Catalog / listing page
- Product detail page
- Cart
- Checkout
- Order confirmation page
- Optional login/signup pages
- Store-level reputation badge slot for third-party ratings

Explicitly out of scope for this slice:

- Admin panel
- ERP integration
- Automated payment gateway integration
- Product review system
- Multi-language support
- Multi-currency support
- International shipping
- Advanced promotions and coupons

## 3. Product Direction

The storefront follows a trust-first commerce approach.

The homepage should not behave like a generic product grid. Its primary job is to reduce hesitation for buyers shopping for refurbished devices. The experience should lead with reassurance and value, then move users into discovery and purchase.

Brand tone:

- Trustworthy and modern
- Price-conscious without feeling cheap
- Clean and product-focused

## 4. Primary User Journey

Primary flow:

`Homepage -> Catalog -> Product Detail -> Cart -> Checkout -> Confirmation`

Supporting flow:

`Homepage/Catalog/Product Detail -> Optional Login/Signup`

Guest checkout remains the default purchase path. Accounts are available, but they must not block browsing or purchase. In this slice, login/signup exists only as a lightweight convenience feature; a full account dashboard and order history remain deferred.

## 5. Page Architecture

### Homepage

Purpose:

- Establish trust fast
- Explain the value proposition
- Lead users into featured inventory and the catalog

Recommended sections:

- Hero with core value proposition
- Trust/value highlights such as tested devices, warranty/returns messaging, and transparent grading
- Store reputation block with generic third-party rating slot
- Featured or recent products
- Short grading explainer teaser with link to fuller explanation
- Clear catalog entry point

### Catalog

Purpose:

- Help users discover inventory quickly with practical filters

Controls:

- Search box
- Category filter
- Grade filter
- Price range filter
- Availability filter
- Sort by newest and price

Display:

- Product grid
- Primary image
- Title
- Price
- Grade
- Availability

### Product Detail Page

Purpose:

- Convert interest into confidence and cart adds

Required content:

- Image gallery with real product photos
- Title
- Description
- Price
- Grade
- Serial number visibility
- Availability status
- Add to cart action
- Shared grading explanation

Condition detail remains lightweight in this slice: grade plus short shared explanation, not per-product inspection reports.

### Cart

Purpose:

- Review selected items before checkout
- Revalidate stock before payment intent

Required behavior:

- Show line items and totals
- Allow removal of items
- Recheck product availability whenever the cart is loaded or checkout starts

### Checkout

Purpose:

- Capture order information with as little friction as possible

Constraints:

- Guest checkout only for the MVP purchase flow
- Portugal-only shipping
- Manual payment methods only

Required fields:

- Name
- Email
- Phone
- Address
- Postal code
- Optional tax ID (NIF)
- Payment method

Payment methods:

- MB Way
- Bank transfer

### Confirmation Page

Purpose:

- Confirm the order was created successfully
- Tell the buyer exactly how to pay

Required content:

- Order summary
- Order identifier
- Payment method selected
- MB Way or bank transfer instructions
- Message confirming the same instructions were emailed

## 6. Trust System

Trust is a first-class part of the storefront, not decorative content.

The storefront should combine:

- Internal trust signals: tested devices, transparent grading, warranty/returns messaging, real product photos, serial visibility
- External trust signals: a generic store-level reputation slot for a future Google-stars or Trustpilot-style integration

The external reputation module should be designed as a configurable block with:

- Source name
- Rating value
- Review count
- Supporting label text
- Destination URL

This keeps the first implementation provider-agnostic.

Placement guidance:

- Prominent on the homepage near the main trust/value area
- Light repetition near checkout to reinforce confidence at the payment decision point

Failure behavior:

- If the third-party reputation source is missing, unavailable, or not yet chosen, the storefront still renders normally
- The module falls back to internal trust messaging instead of blocking the page or leaving broken UI

## 7. Inventory Model

The storefront must support mixed inventory:

- One-off products for unique refurbished items
- Quantity-based products for items with small stock counts

Storefront-facing availability states:

- Available
- Low stock
- Sold out / unavailable

Behavior:

- One-off products become unavailable as soon as an order is created
- Quantity-based products reserve stock immediately when an order is created
- If an unpaid order is later cancelled by admin workflow, that reserved stock should be returned to inventory
- The storefront always displays current sale availability, not historical assumptions from when the item was first viewed

## 8. Cart And Checkout Behavior

### Stock revalidation

- Availability must be rechecked when adding to cart
- Availability must be rechecked again when loading the cart and before checkout submission
- If a product becomes unavailable, it is removed from the cart and the buyer sees a clear explanation

Chosen behavior:

- Block checkout and remove unavailable items from the cart with a clear message

### Order creation

On successful checkout submission:

1. Validate required customer and shipping fields
2. Validate Portugal-only delivery scope
3. Validate selected payment method
4. Revalidate stock
5. Create the order
6. Snapshot customer, items, and pricing data onto the order
7. Mark the order status as `payment_pending`
8. Show payment instructions on the confirmation page
9. Send the same instructions by email

## 9. Data Responsibilities

The storefront should separate responsibilities into clear units.

### Trust content

Owns:

- Hero messaging
- Value proposition blocks
- Warranty/returns/tested-device messaging
- Grading explainer teaser
- Reputation badge slot

### Catalog discovery

Owns:

- Search
- Filters
- Sort
- Availability filtering
- Empty state behavior

### Product presentation

Owns:

- Product cards
- Product detail display
- Gallery
- Grade and serial display
- Availability messaging

### Cart and checkout

Owns:

- Cart line items
- Stock revalidation
- Customer form capture
- Payment method selection
- Order creation handoff
- Confirmation messaging

### Reputation integration

Owns:

- Source metadata display
- Rating value display
- Link-out to external proof source
- Graceful fallback when unavailable

## 10. Error Handling And UX Safeguards

### Empty catalog results

- Show a friendly no-results state
- Provide a reset-filters action
- Avoid dead-end blank grids

### Reputation provider unavailable

- Render fallback trust messaging
- Do not show broken embeds or blocking errors

### Invalid checkout data

- Show inline validation messages
- Preserve entered form data where possible

### Stock conflicts

- Remove unavailable items from cart
- Explain which item changed and why checkout cannot continue until the cart is refreshed

### Unsupported shipping scope

- Prevent checkout completion for non-Portugal addresses in this MVP
- Explain that launch shipping is currently limited to Portugal

## 11. Testing Strategy For This Slice

Focus testing on the highest-risk customer journeys.

### Browsing and discovery

- Search returns expected products
- Category, grade, price, and availability filters combine correctly
- Sorting by newest and price works correctly
- Empty states render clearly

### Product detail

- Gallery displays correctly
- Grade and serial number appear correctly
- Availability and add-to-cart behavior reflect current stock

### Cart and checkout

- Unavailable items are removed during revalidation
- Checkout form validates required fields
- Portugal-only delivery rule is enforced
- Manual payment method selection is required

### Order flow

- Successful submission creates an order
- New order is marked `payment_pending`
- Confirmation page shows correct payment instructions
- Payment-instructions email is sent with matching content

### Reputation block

- Configured reputation data displays correctly
- Missing or unavailable reputation data degrades gracefully

## 12. Success Criteria For This Slice

The storefront MVP is successful when:

- Users can understand why the store is trustworthy within the first page view
- Users can browse products with practical discovery controls
- Users can add available products to the cart and complete checkout as guests
- Orders are created reliably with `payment_pending` status
- Buyers receive clear payment instructions on screen and by email
- Trust/reputation elements strengthen confidence without becoming a dependency for core flows

## 13. Deferred For Later Phases

These are intentionally deferred so the first slice remains small and shippable:

- Product reviews
- Automated payment capture
- Advanced search behavior
- Multi-country shipping
- Full customer account area and order history
- Admin workflows
- ERP integration
- Shipping-provider integration

## 14. Open Integration Decision

The design includes a store-level reputation slot, but the exact provider is intentionally undecided.

The implementation should treat this as a pluggable integration point so the store can later connect to:

- Google Business profile stars
- Trustpilot
- Another provider with equivalent store-level reputation data

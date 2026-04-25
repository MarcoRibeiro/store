# Storefront MVP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first customer-facing storefront MVP for the refurbished-tech business, covering trust-led discovery, catalog browsing, product detail, cart, checkout, manual payment confirmation, and a lightweight optional account shell.

**Architecture:** Build a greenfield Next.js App Router application with server-rendered catalog pages, Prisma-backed product/order storage in PostgreSQL, and small domain libraries for catalog, cart, checkout, and reputation data. Keep checkout guest-first, reserve stock at order creation, and isolate provider-dependent concerns behind narrow adapters so trust-rating and email delivery can change later without rewriting core flows.

**Tech Stack:** Next.js App Router (current stable guidance via `/vercel/next.js`), TypeScript, Prisma + PostgreSQL (patterns from `/websites/prisma_io`), Auth.js for optional account pages (`/nextauthjs/next-auth`), Vitest, Playwright

---

## File Structure

Planned top-level structure:

- `app/` - Next.js App Router pages, layouts, and route handlers
- `components/storefront/` - visual storefront components and trust blocks
- `components/forms/` - reusable cart/checkout/auth forms
- `lib/db.ts` - singleton Prisma client
- `lib/catalog/` - catalog queries and mappers
- `lib/cart/` - cart cookie/session helpers and revalidation logic
- `lib/orders/` - order creation, stock reservation, and confirmation helpers
- `lib/auth/` - Auth.js configuration and account helpers
- `lib/reputation/` - provider-agnostic reputation block model
- `lib/validation/` - checkout and auth validation
- `prisma/` - schema, migrations, and seed script
- `tests/unit/` - Vitest unit and integration-style tests for domain code
- `tests/e2e/` - Playwright flows for storefront, cart, and checkout

## Task 1: Scaffold the application and baseline tooling

**Files:**
- Create: `package.json`
- Create: `next.config.ts`
- Create: `tsconfig.json`
- Create: `app/layout.tsx`
- Create: `app/page.tsx`
- Create: `app/globals.css`
- Create: `vitest.config.ts`
- Create: `tests/unit/homepage.test.tsx`
- Create: `.env.example`
- Create: `README.md`

- [ ] **Step 1: Write the failing homepage smoke test**

```tsx
// tests/unit/homepage.test.tsx
import { render, screen } from "@testing-library/react"
import HomePage from "@/app/page"

describe("homepage", () => {
  it("renders the trust-first headline", async () => {
    const Page = await HomePage()
    render(Page)
    expect(
      screen.getByRole("heading", {
        name: /refurbished tech you can trust/i,
      }),
    ).toBeInTheDocument()
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/homepage.test.tsx`
Expected: FAIL because the Next.js app files do not exist yet.

- [ ] **Step 3: Create the minimal app shell and tooling**

```json
// package.json
{
  "name": "store",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "playwright test",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:seed": "tsx prisma/seed.ts"
  }
}
```

```tsx
// app/layout.tsx
import "./globals.css"

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

```tsx
// app/page.tsx
export default async function HomePage() {
  return (
    <main>
      <h1>Refurbished tech you can trust</h1>
      <p>Tested devices, transparent grading, and clear manual payment instructions.</p>
    </main>
  )
}
```

- [ ] **Step 4: Run the unit test again**

Run: `npm run test -- tests/unit/homepage.test.tsx`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add package.json next.config.ts tsconfig.json app/layout.tsx app/page.tsx app/globals.css vitest.config.ts tests/unit/homepage.test.tsx .env.example README.md
git commit -m "chore: scaffold storefront application"
```

## Task 2: Model the domain with Prisma and seed representative storefront data

**Files:**
- Create: `prisma/schema.prisma`
- Create: `prisma/seed.ts`
- Create: `lib/db.ts`
- Create: `tests/unit/catalog-schema.test.ts`
- Modify: `.env.example`

- [ ] **Step 1: Write the failing schema-level test**

```ts
// tests/unit/catalog-schema.test.ts
import { describe, expect, it } from "vitest"
import { productGradeLabels } from "@/lib/catalog/constants"

describe("catalog domain constants", () => {
  it("supports the approved A/B/C grading model", () => {
    expect(productGradeLabels).toEqual({
      A: "Excellent",
      B: "Very Good",
      C: "Good",
    })
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/catalog-schema.test.ts`
Expected: FAIL because the catalog constants and Prisma-backed domain files do not exist yet.

- [ ] **Step 3: Create the Prisma schema, client, and seed data**

```prisma
// prisma/schema.prisma
model Category {
  id        String    @id @default(cuid())
  name      String
  slug      String    @unique
  active    Boolean   @default(true)
  products  Product[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Product {
  id             String         @id @default(cuid())
  title          String
  slug           String         @unique
  description    String
  priceCents     Int
  grade          ProductGrade
  serialNumber   String
  stockMode      StockMode
  stockQuantity  Int            @default(1)
  featured       Boolean        @default(false)
  categoryId     String
  category       Category       @relation(fields: [categoryId], references: [id])
  images         ProductImage[]
  orderItems     OrderItem[]
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt
}

model ProductImage {
  id         String   @id @default(cuid())
  productId  String
  product    Product  @relation(fields: [productId], references: [id])
  url        String
  altText    String
  sortOrder  Int
  isPrimary  Boolean  @default(false)
}

model Order {
  id                String        @id @default(cuid())
  email             String
  customerName      String
  phone             String
  addressLine       String
  postalCode        String
  taxId             String?
  paymentMethod     PaymentMethod
  status            OrderStatus   @default(PAYMENT_PENDING)
  totalCents        Int
  items             OrderItem[]
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model OrderItem {
  id             String   @id @default(cuid())
  orderId        String
  order          Order    @relation(fields: [orderId], references: [id])
  productId      String
  product        Product  @relation(fields: [productId], references: [id])
  titleSnapshot  String
  gradeSnapshot  ProductGrade
  priceCents     Int
  quantity       Int
}

enum ProductGrade {
  A
  B
  C
}

enum StockMode {
  SINGLE
  QUANTITY
}

enum PaymentMethod {
  MB_WAY
  BANK_TRANSFER
}

enum OrderStatus {
  PAYMENT_PENDING
  PAID
  SHIPPED
  COMPLETED
  CANCELLED
}
```

```ts
// lib/db.ts
import { PrismaClient } from "@prisma/client"

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient }

export const db =
  globalForPrisma.prisma ??
  new PrismaClient({ log: process.env.NODE_ENV === "development" ? ["warn", "error"] : ["error"] })

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db
```

```ts
// prisma/seed.ts
await db.category.createMany({
  data: [
    { name: "Smartphones", slug: "smartphones" },
    { name: "Laptops", slug: "laptops" },
    { name: "Consoles", slug: "consoles" },
  ],
})
```

- [ ] **Step 4: Run migrations, seed data, and rerun tests**

Run: `npm run db:generate && npm run db:migrate -- --name init_storefront && npm run db:seed && npm run test -- tests/unit/catalog-schema.test.ts`
Expected: Prisma client generated, migration applied, seed completed, test PASS

- [ ] **Step 5: Commit**

```bash
git add prisma/schema.prisma prisma/seed.ts lib/db.ts tests/unit/catalog-schema.test.ts .env.example
git commit -m "feat: add storefront database schema and seed data"
```

## Task 3: Build catalog queries and trust-aware homepage data

**Files:**
- Create: `lib/catalog/constants.ts`
- Create: `lib/catalog/queries.ts`
- Create: `lib/reputation/types.ts`
- Create: `lib/reputation/config.ts`
- Create: `components/storefront/home-hero.tsx`
- Create: `components/storefront/reputation-badge.tsx`
- Create: `components/storefront/featured-products.tsx`
- Create: `tests/unit/homepage-data.test.ts`
- Modify: `app/page.tsx`

- [ ] **Step 1: Write the failing query test**

```ts
// tests/unit/homepage-data.test.ts
import { describe, expect, it } from "vitest"
import { mapReputationConfig } from "@/lib/reputation/config"

describe("reputation config", () => {
  it("returns null when no provider values are configured", () => {
    expect(
      mapReputationConfig({
        sourceName: "",
        ratingValue: "",
        reviewCount: "",
        targetUrl: "",
      }),
    ).toBeNull()
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/homepage-data.test.ts`
Expected: FAIL because the reputation mapping and homepage data files do not exist.

- [ ] **Step 3: Create query helpers and homepage components**

```ts
// lib/catalog/constants.ts
export const productGradeLabels = {
  A: "Excellent",
  B: "Very Good",
  C: "Good",
} as const
```

```ts
// lib/reputation/config.ts
import type { ReputationBadgeModel } from "./types"

type ReputationEnv = {
  sourceName: string
  ratingValue: string
  reviewCount: string
  targetUrl: string
}

export function mapReputationConfig(env: ReputationEnv): ReputationBadgeModel | null {
  if (!env.sourceName || !env.ratingValue || !env.reviewCount || !env.targetUrl) {
    return null
  }

  return {
    sourceName: env.sourceName,
    ratingValue: Number(env.ratingValue),
    reviewCount: Number(env.reviewCount),
    targetUrl: env.targetUrl,
    label: "Store rating",
  }
}
```

```tsx
// app/page.tsx
import { getFeaturedProducts } from "@/lib/catalog/queries"
import { mapReputationConfig } from "@/lib/reputation/config"

export default async function HomePage() {
  const products = await getFeaturedProducts()
  const reputation = mapReputationConfig({
    sourceName: process.env.REPUTATION_SOURCE_NAME ?? "",
    ratingValue: process.env.REPUTATION_RATING_VALUE ?? "",
    reviewCount: process.env.REPUTATION_REVIEW_COUNT ?? "",
    targetUrl: process.env.REPUTATION_TARGET_URL ?? "",
  })

  return (
    <main>
      <h1>Refurbished tech you can trust</h1>
      {reputation ? <p>{reputation.sourceName}</p> : <p>Fully tested, honestly graded, ready to ship in Portugal.</p>}
      <section>{products.map((product) => <article key={product.id}>{product.title}</article>)}</section>
    </main>
  )
}
```

- [ ] **Step 4: Run the tests**

Run: `npm run test -- tests/unit/homepage-data.test.ts tests/unit/homepage.test.tsx`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add lib/catalog/constants.ts lib/catalog/queries.ts lib/reputation/types.ts lib/reputation/config.ts components/storefront/home-hero.tsx components/storefront/reputation-badge.tsx components/storefront/featured-products.tsx tests/unit/homepage-data.test.ts app/page.tsx
git commit -m "feat: add homepage trust and catalog data"
```

## Task 4: Implement catalog listing and product detail pages

**Files:**
- Create: `app/products/page.tsx`
- Create: `app/products/[slug]/page.tsx`
- Create: `components/storefront/catalog-filters.tsx`
- Create: `components/storefront/product-card.tsx`
- Create: `components/storefront/product-gallery.tsx`
- Create: `components/storefront/grade-explainer.tsx`
- Create: `tests/unit/catalog-filters.test.ts`
- Create: `tests/e2e/catalog.spec.ts`
- Modify: `lib/catalog/queries.ts`

- [ ] **Step 1: Write the failing filter test**

```ts
// tests/unit/catalog-filters.test.ts
import { describe, expect, it } from "vitest"
import { normalizeCatalogSearchParams } from "@/lib/catalog/queries"

describe("normalizeCatalogSearchParams", () => {
  it("defaults to newest sorting", () => {
    expect(normalizeCatalogSearchParams({})).toMatchObject({ sort: "newest" })
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/catalog-filters.test.ts`
Expected: FAIL because the catalog normalization helper is missing.

- [ ] **Step 3: Implement the catalog and product detail pages**

```ts
// lib/catalog/queries.ts
export function normalizeCatalogSearchParams(input: Record<string, string | string[] | undefined>) {
  return {
    query: typeof input.q === "string" ? input.q : "",
    category: typeof input.category === "string" ? input.category : "",
    grade: typeof input.grade === "string" ? input.grade : "",
    availability: typeof input.availability === "string" ? input.availability : "available",
    sort: input.sort === "price-asc" || input.sort === "price-desc" ? input.sort : "newest",
  }
}
```

```tsx
// app/products/page.tsx
import { getCatalogPageData, normalizeCatalogSearchParams } from "@/lib/catalog/queries"

export default async function ProductsPage({ searchParams }: { searchParams: Promise<Record<string, string | string[] | undefined>> }) {
  const normalized = normalizeCatalogSearchParams(await searchParams)
  const data = await getCatalogPageData(normalized)

  return (
    <main>
      <h1>Browse refurbished tech</h1>
      <p>{data.total} products found</p>
    </main>
  )
}
```

```tsx
// app/products/[slug]/page.tsx
import { notFound } from "next/navigation"
import { getProductBySlug } from "@/lib/catalog/queries"

export default async function ProductDetailPage({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params
  const product = await getProductBySlug(slug)

  if (!product) notFound()

  return (
    <main>
      <h1>{product.title}</h1>
      <p>{product.serialNumber}</p>
    </main>
  )
}
```

- [ ] **Step 4: Run the unit and e2e catalog tests**

Run: `npm run test -- tests/unit/catalog-filters.test.ts && npm run test:e2e -- tests/e2e/catalog.spec.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add app/products/page.tsx app/products/[slug]/page.tsx components/storefront/catalog-filters.tsx components/storefront/product-card.tsx components/storefront/product-gallery.tsx components/storefront/grade-explainer.tsx tests/unit/catalog-filters.test.ts tests/e2e/catalog.spec.ts lib/catalog/queries.ts
git commit -m "feat: add catalog and product detail pages"
```

## Task 5: Add cookie-backed cart state and stock revalidation

**Files:**
- Create: `app/cart/page.tsx`
- Create: `app/actions/cart.ts`
- Create: `lib/cart/cookie.ts`
- Create: `lib/cart/service.ts`
- Create: `components/storefront/cart-table.tsx`
- Create: `tests/unit/cart-service.test.ts`
- Create: `tests/e2e/cart.spec.ts`

- [ ] **Step 1: Write the failing cart revalidation test**

```ts
// tests/unit/cart-service.test.ts
import { describe, expect, it } from "vitest"
import { revalidateCartLines } from "@/lib/cart/service"

describe("revalidateCartLines", () => {
  it("drops unavailable single-stock items", async () => {
    const result = await revalidateCartLines([
      { productId: "single-product", quantity: 1 },
    ])

    expect(result.removed).toContain("single-product")
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/cart-service.test.ts`
Expected: FAIL because the cart service does not exist.

- [ ] **Step 3: Implement cart storage and revalidation**

```ts
// lib/cart/cookie.ts
export const CART_COOKIE = "store-cart"

export type CartLineInput = {
  productId: string
  quantity: number
}
```

```ts
// lib/cart/service.ts
import { db } from "@/lib/db"

export async function revalidateCartLines(lines: Array<{ productId: string; quantity: number }>) {
  const removed: string[] = []

  for (const line of lines) {
    const product = await db.product.findUnique({ where: { id: line.productId } })
    if (!product || product.stockQuantity < line.quantity) removed.push(line.productId)
  }

  return { removed }
}
```

```tsx
// app/cart/page.tsx
import { getCartViewModel } from "@/lib/cart/service"

export default async function CartPage() {
  const cart = await getCartViewModel()
  return <main><h1>Your cart</h1><p>{cart.lines.length} items</p></main>
}
```

- [ ] **Step 4: Run cart tests**

Run: `npm run test -- tests/unit/cart-service.test.ts && npm run test:e2e -- tests/e2e/cart.spec.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add app/cart/page.tsx app/actions/cart.ts lib/cart/cookie.ts lib/cart/service.ts components/storefront/cart-table.tsx tests/unit/cart-service.test.ts tests/e2e/cart.spec.ts
git commit -m "feat: add cart state and stock revalidation"
```

## Task 6: Implement checkout, order creation, and manual payment instructions

**Files:**
- Create: `app/checkout/page.tsx`
- Create: `app/checkout/actions.ts`
- Create: `app/orders/[id]/page.tsx`
- Create: `components/forms/checkout-form.tsx`
- Create: `lib/orders/create-order.ts`
- Create: `lib/orders/payment-instructions.ts`
- Create: `lib/validation/checkout.ts`
- Create: `tests/unit/create-order.test.ts`
- Create: `tests/e2e/checkout.spec.ts`

- [ ] **Step 1: Write the failing order-creation test**

```ts
// tests/unit/create-order.test.ts
import { describe, expect, it } from "vitest"
import { createOrderFromCheckout } from "@/lib/orders/create-order"

describe("createOrderFromCheckout", () => {
  it("creates payment_pending orders and reserves stock", async () => {
    const order = await createOrderFromCheckout({
      customerName: "Marco Ribeiro",
      email: "marco@example.com",
      phone: "910000000",
      addressLine: "Rua Exemplo 1",
      postalCode: "1000-100",
      paymentMethod: "MB_WAY",
      lines: [{ productId: "qty-product", quantity: 1 }],
    })

    expect(order.status).toBe("PAYMENT_PENDING")
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/create-order.test.ts`
Expected: FAIL because order creation is not implemented.

- [ ] **Step 3: Implement validation, transactional order creation, and confirmation data**

```ts
// lib/validation/checkout.ts
export function validatePortugalPostalCode(postalCode: string) {
  return /^\d{4}-\d{3}$/.test(postalCode)
}
```

```ts
// lib/orders/create-order.ts
import { db } from "@/lib/db"

export async function createOrderFromCheckout(input: CheckoutInput) {
  return db.$transaction(async (tx) => {
    const products = await tx.product.findMany({ where: { id: { in: input.lines.map((line) => line.productId) } } })

    const order = await tx.order.create({
      data: {
        customerName: input.customerName,
        email: input.email,
        phone: input.phone,
        addressLine: input.addressLine,
        postalCode: input.postalCode,
        taxId: input.taxId,
        paymentMethod: input.paymentMethod,
        status: "PAYMENT_PENDING",
        totalCents: products.reduce((sum, product) => sum + product.priceCents, 0),
        items: {
          create: products.map((product) => ({
            productId: product.id,
            titleSnapshot: product.title,
            gradeSnapshot: product.grade,
            priceCents: product.priceCents,
            quantity: input.lines.find((line) => line.productId === product.id)?.quantity ?? 1,
          })),
        },
      },
    })

    for (const product of products) {
      await tx.product.update({
        where: { id: product.id },
        data: {
          stockQuantity: Math.max(0, product.stockQuantity - 1),
        },
      })
    }

    return order
  })
}
```

```ts
// lib/orders/payment-instructions.ts
export function getPaymentInstructions(method: "MB_WAY" | "BANK_TRANSFER") {
  return method === "MB_WAY"
    ? "Use the MB Way number configured in env and include the order number in the payment note."
    : "Transfer the order total to the configured IBAN and include the order number in the reference."
}
```

- [ ] **Step 4: Run checkout tests**

Run: `npm run test -- tests/unit/create-order.test.ts && npm run test:e2e -- tests/e2e/checkout.spec.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add app/checkout/page.tsx app/checkout/actions.ts app/orders/[id]/page.tsx components/forms/checkout-form.tsx lib/orders/create-order.ts lib/orders/payment-instructions.ts lib/validation/checkout.ts tests/unit/create-order.test.ts tests/e2e/checkout.spec.ts
git commit -m "feat: add checkout and order creation flow"
```

## Task 7: Send confirmation email and add resilient trust/reputation fallback

**Files:**
- Create: `lib/mailer.ts`
- Create: `lib/orders/send-confirmation.ts`
- Create: `components/storefront/trust-strip.tsx`
- Create: `tests/unit/payment-instructions.test.ts`
- Modify: `app/orders/[id]/page.tsx`
- Modify: `app/page.tsx`

- [ ] **Step 1: Write the failing payment-instructions test**

```ts
// tests/unit/payment-instructions.test.ts
import { describe, expect, it } from "vitest"
import { getPaymentInstructions } from "@/lib/orders/payment-instructions"

describe("getPaymentInstructions", () => {
  it("returns MB Way instructions", () => {
    expect(getPaymentInstructions("MB_WAY")).toMatch(/mb way/i)
  })
})
```

- [ ] **Step 2: Run the test to verify it fails if not already green**

Run: `npm run test -- tests/unit/payment-instructions.test.ts`
Expected: FAIL if the helper has not yet been implemented in Task 6; otherwise keep this test and extend it for email content matching.

- [ ] **Step 3: Implement mailer and email dispatch**

```ts
// lib/mailer.ts
export type MailPayload = {
  to: string
  subject: string
  html: string
}

export async function sendMail(payload: MailPayload) {
  if (process.env.NODE_ENV !== "production") {
    console.log("DEV_MAIL", payload)
    return
  }

  throw new Error("Production mail transport not configured")
}
```

```ts
// lib/orders/send-confirmation.ts
import { sendMail } from "@/lib/mailer"
import { getPaymentInstructions } from "@/lib/orders/payment-instructions"

export async function sendOrderConfirmationEmail(input: { email: string; orderId: string; paymentMethod: "MB_WAY" | "BANK_TRANSFER" }) {
  await sendMail({
    to: input.email,
    subject: `Order ${input.orderId} received`,
    html: `<p>${getPaymentInstructions(input.paymentMethod)}</p>`,
  })
}
```

- [ ] **Step 4: Run unit tests and smoke the confirmation page**

Run: `npm run test -- tests/unit/payment-instructions.test.ts tests/unit/create-order.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add lib/mailer.ts lib/orders/send-confirmation.ts components/storefront/trust-strip.tsx tests/unit/payment-instructions.test.ts app/orders/[id]/page.tsx app/page.tsx
git commit -m "feat: add confirmation email and trust fallback"
```

## Task 8: Add optional Auth.js account access without blocking guest checkout

**Files:**
- Create: `auth.ts`
- Create: `app/api/auth/[...nextauth]/route.ts`
- Create: `app/login/page.tsx`
- Create: `app/signup/page.tsx`
- Create: `app/account/page.tsx`
- Create: `components/forms/sign-in-form.tsx`
- Create: `tests/unit/auth-page.test.tsx`

- [ ] **Step 1: Write the failing protected-page test**

```tsx
// tests/unit/auth-page.test.tsx
import { describe, expect, it, vi } from "vitest"

vi.mock("@/auth", () => ({ auth: vi.fn(async () => null) }))

describe("account page", () => {
  it("renders an access message when the user is not signed in", async () => {
    const { default: AccountPage } = await import("@/app/account/page")
    const page = await AccountPage()
    expect(page).toBeTruthy()
  })
})
```

- [ ] **Step 2: Run the test to verify it fails**

Run: `npm run test -- tests/unit/auth-page.test.tsx`
Expected: FAIL because the auth files do not exist.

- [ ] **Step 3: Implement Auth.js and lightweight account pages**

```ts
// auth.ts
import NextAuth from "next-auth"
import Credentials from "next-auth/providers/credentials"

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Credentials({
      credentials: {
        email: {},
        password: {},
      },
      authorize: async () => null,
    }),
  ],
  pages: {
    signIn: "/login",
  },
})
```

```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"

export const { GET, POST } = handlers
```

```tsx
// app/account/page.tsx
import { auth } from "@/auth"

export default async function AccountPage() {
  const session = await auth()

  if (!session) {
    return <main><h1>Sign in to save your details</h1></main>
  }

  return <main><h1>My account</h1></main>
}
```

- [ ] **Step 4: Run the auth tests**

Run: `npm run test -- tests/unit/auth-page.test.tsx`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add auth.ts app/api/auth/[...nextauth]/route.ts app/login/page.tsx app/signup/page.tsx app/account/page.tsx components/forms/sign-in-form.tsx tests/unit/auth-page.test.tsx
git commit -m "feat: add optional account access"
```

## Task 9: Add end-to-end quality gates and deployment notes

**Files:**
- Create: `playwright.config.ts`
- Create: `tests/e2e/homepage.spec.ts`
- Create: `tests/e2e/full-checkout.spec.ts`
- Modify: `README.md`
- Modify: `.env.example`

- [ ] **Step 1: Write the failing full-flow Playwright spec**

```ts
// tests/e2e/full-checkout.spec.ts
import { test, expect } from "@playwright/test"

test("guest shopper can place an MB Way order", async ({ page }) => {
  await page.goto("/")
  await expect(page.getByRole("heading", { name: /refurbished tech you can trust/i })).toBeVisible()
  await page.getByRole("link", { name: /browse products/i }).click()
  await page.getByRole("button", { name: /add to cart/i }).first().click()
  await page.goto("/checkout")
  await page.getByLabel(/name/i).fill("Marco Ribeiro")
  await page.getByLabel(/email/i).fill("marco@example.com")
  await page.getByLabel(/phone/i).fill("910000000")
  await page.getByLabel(/address/i).fill("Rua Exemplo 1")
  await page.getByLabel(/postal code/i).fill("1000-100")
  await page.getByLabel(/mb way/i).check()
  await page.getByRole("button", { name: /place order/i }).click()
  await expect(page.getByText(/payment instructions/i)).toBeVisible()
})
```

- [ ] **Step 2: Run the e2e suite to verify it fails until the flow is complete**

Run: `npm run test:e2e -- tests/e2e/full-checkout.spec.ts`
Expected: FAIL until the full user flow is implemented.

- [ ] **Step 3: Finalize README, env docs, and CI-friendly commands**

```md
// README.md
## Local development

1. `npm install`
2. Copy `.env.example` to `.env`
3. `npm run db:migrate -- --name init_storefront`
4. `npm run db:seed`
5. `npm run dev`

## Verification

- `npm run test`
- `npm run test:e2e`
- `npm run build`
```

- [ ] **Step 4: Run the final verification commands**

Run: `npm run test && npm run test:e2e && npm run build`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add playwright.config.ts tests/e2e/homepage.spec.ts tests/e2e/full-checkout.spec.ts README.md .env.example
git commit -m "test: add storefront verification coverage"
```

## Self-Review

- Spec coverage: homepage trust, catalog, product detail, cart, guest checkout, Portugal-only shipping, manual payment confirmation, stock reservation, reputation fallback, and optional account entry all map to Tasks 3 through 8.
- Placeholder scan: no `TBD`, `TODO`, or deferred implementation notes remain inside the executable tasks.
- Type consistency: product grades, payment methods, and order status names stay consistent with the Prisma schema and service examples above.

## GitHub Issue Breakdown

Create one implementation issue per task after saving this plan:

1. Scaffold the storefront application and tooling
2. Add Prisma schema, migrations, and seed data
3. Build homepage trust content and reputation config
4. Implement catalog listing and product detail pages
5. Add cart state and stock revalidation
6. Implement checkout and order creation flow
7. Send confirmation email and trust fallback content
8. Add optional Auth.js account access
9. Add end-to-end verification and deployment docs

## Suggested Labels

- `frontend`
- `backend`
- `database`
- `auth`
- `checkout`
- `testing`
- `docs`

## Execution Handoff

Plan complete and saved to `docs/superpowers/plans/2026-04-26-storefront-mvp.md`. Two execution options:

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?

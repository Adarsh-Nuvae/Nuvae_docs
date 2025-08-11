# Denials API Guide (super simple + emojis) 🚀

This is your friendly map of everything under `src/pages/api/denials`. We'll keep it visual, plain-English, and add Prisma pointers so it's easy to understand and maintain.

---

## 📖 Legend

- 🧭 **What it does**
- 🧪 **Input** (query/body)
- 🗃️ **Prisma in action**
- 📤 **Output**
- 🔐 **Auth**
- 💡 **Tips**

---

## 🏗️ Big Picture (How the API Fits Together)

- The **UI calls** these endpoints to list, filter, and change denial statuses
- You can **act on one denial** (edit/review) or **many at once** (bulk review/resolve)
- **Status journey:** `OPEN` → `INPROGRESS` → `CLOSED` (reviewed) → `RESOLVED` (final)

---

## 📁 Endpoints Directory

- **GET** `/api/denials` ➜ list denials
- **PUT** `/api/denials/[denialId]/edit` ➜ mark INPROGRESS
- **PUT** `/api/denials/[denialId]/review` ➜ mark CLOSED (+ update files)
- `/api/denials/[denialId]/reasons` (CRUD) ➜ manage reasons
- **GET** `/api/denials/bulk-review` ➜ list for bulk review + counters
- **PUT** `/api/denials/bulk-review` ➜ bulk close (CLOSED)
- **PUT** `/api/denials/bulk-review/complete` ➜ move CLOSED → RESOLVED
- **GET** `/api/denials/bulk-review/summary` ➜ totals + validation for download
- **POST** `/api/denials/by-ids` ➜ fetch by ID list
- **GET** `/api/denials/payers` ➜ payer list
- **GET** `/api/denials/payers/[id]` ➜ one payer
- **GET** `/api/denials/payers/[id]/reason-codes` ➜ reason codes
- **GET** `/api/denials/payer-plans` ➜ unique plan names from denials

---

## 🔗 API Endpoints Details

### 1️⃣ **GET** `/api/denials` (list)

- 🧭 **What:** Returns a page of denials with filters (payer, date, status, amount, financial class)
- 🧪 **Input (query):** `pageNo`, `pageSize`, `claimId`, `payerId=1,2`, `startDate`, `endDate`, `status`, `amountOperator+amountValue`, `financialClassId=3,4`
- 🗃️ **Prisma:** `count()`, `findMany({ where, skip, take, include: { patient.resource, financialClass }, orderBy: createdAt desc })`
- 📤 **Output:** `{ denials: DenialDto[], totalClaimsCount, pageNo, pageSize }`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** "CLOSED" filter also includes RESOLVED behind the scenes

---

### 2️⃣ **PUT** `/api/denials/[denialId]/edit`

- 🧭 **What:** Move a denial to INPROGRESS
- 🧪 **Input:** path param `denialId`
- 🗃️ **Prisma:** `findUnique({ id })`, `update({ id, data: { status: INPROGRESS, updatedAt }})`
- 📤 **Output:** updated denial JSON
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Use this when someone starts editing or working a denial

---

### 3️⃣ **PUT** `/api/denials/[denialId]/review`

- 🧭 **What:** Mark a denial as CLOSED (reviewed). Optionally mark related files CLOSED too
- 🧪 **Input (body):** `{ files: ["PDR_FORM", "APPEAL_LETTER"] }`
- 🗃️ **Prisma:**
  - `denial.update({ status: CLOSED })`
  - if `PDR_FORM` ➜ `pDRForm.updateMany({ denialId, denialStatus: CLOSED })`
  - if `APPEAL_LETTER` ➜ `appeal.updateMany({ denialId, denialStatus: CLOSED })`
- 📤 **Output:** updated denial JSON
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** 400 if already CLOSED, 404 if not found

---

### 4️⃣ `/api/denials/[denialId]/reasons` (POST/GET/PUT/DELETE)

- 🧭 **What:** Manage Claim Rejection Reasons for a denial
- 🧪 **Input:**
  - **POST body:** `{ denialId, content, accuracy }`
  - **GET list:** from path `[denialId]`
  - **GET single:** add `?id=123`
  - **PUT/DELETE:** `?id=123`
- 🗃️ **Prisma:** `claimRejectionReason.create/findMany/findUnique/update/delete` (with includes for nested docs/ICD)
- 📤 **Output:** reason(s) JSON
- 🔐 **Auth:** (not wrapped with withAPIAuth in code shown — consider adding it if needed)
- 💡 **Tip:** The path includes `[denialId]`, but single fetch uses `?id=` — slightly unusual

---

### 5️⃣ **GET** `/api/denials/bulk-review`

- 🧭 **What:** Returns a page of denials for bulk review and useful counters
- 🧪 **Input (query):** same filter style as list; plus `pageNo/pageSize`
- 🗃️ **Prisma:** multiple `count()` calls, then `findMany` with ordering by `denialStatusOrder.order asc`, `createdAt desc`
- 📤 **Output:** `{ data, totalCount, totalReviewedClaims, totalEditedClaims, totalPendingReview, pageNo, pageSize }`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Excludes RESOLVED so you focus on items still in play

---

### 6️⃣ **PUT** `/api/denials/bulk-review`

- 🧭 **What:** Bulk set many denials to CLOSED. Optionally close related PDR/Appeal files
- 🧪 **Input (body):** 
  ```json
  {
    payerId?: number[], 
    reasonCodes?: number[], 
    paidDate?: { startDate, endDate },
    denialIds?: number[], 
    files?: ["PDR_FORM"|"APPEAL_LETTER"], 
    appealAmount?, 
    includeAllStatus?, 
    financialClassId?
  }
  ```
- 🗃️ **Prisma:**
  - `denial.updateMany({ where: { ...filters, status: { not: RESOLVED }}, data: { status: CLOSED, updatedAt }})`
  - `pDRForm.updateMany(...)`, `appeal.updateMany(...)` if files provided
- 📤 **Output:** `{ message, updatedCount }`
- 🔐 **Auth:** `edit_denials`
- 💡 **Tip:** Try GET bulk-review first to preview what you'll close

---

### 7️⃣ **PUT** `/api/denials/bulk-review/complete`

- 🧭 **What:** Move previously CLOSED denials to RESOLVED (final step after downloads)
- 🧪 **Input (body):** same filter ideas as bulk-review
- 🗃️ **Prisma:** `denial.updateMany({ where: { ...filters, status: CLOSED }, data: { status: RESOLVED, updatedAt }})`
- 📤 **Output:** `{ message, updatedCount }`
- 🔐 **Auth:** (currently none set in code; consider requiring `edit_denials`)
- 💡 **Tip:** Use after download-confirm so reports match reality

---

### 8️⃣ **GET** `/api/denials/bulk-review/summary`

- 🧭 **What:** Gives totals for amounts and a validation checklist for PDR forms
- 🧪 **Input (query):** same filters as bulk review
- 🗃️ **Prisma:**
  - `findMany({ where: { ...filters, status: CLOSED }, include: { appeal: { pdrForms: true }}})`
  - `count({ where: { ...filters, status: { not: RESOLVED }}})`
- 📤 **Output:** 
  ```json
  {
    totalClaimAmount, 
    totalPaymentAmount, 
    estimatedRecovery,
    count, 
    totalCount, 
    validationResults: [{ denialId, missingFields[] }]
  }
  ```
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Great for telling users what's missing before finalizing

---

### 9️⃣ **POST** `/api/denials/by-ids`

- 🧭 **What:** Fetch a group of denials by ID list
- 🧪 **Input (body):** `{ denialIds: number[] }`
- 🗃️ **Prisma:** `denial.findMany({ where: { id: { in: denialIds }}, orderBy: createdAt desc })`
- 📤 **Output:** `{ denials: DenialDto[] }`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Use for ad-hoc selections

---

### 🔟 **GET** `/api/denials/payers`

- 🧭 **What:** List payers for dropdowns
- 🧪 **Input:** none
- 🗃️ **Prisma:** `denialPayer.findMany({ select: { id, displayName }, orderBy: displayName asc })`
- 📤 **Output:** `[{ id, name }]`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Mapped to `{ id, name }` on output

---

### 1️⃣1️⃣ **GET** `/api/denials/payers/[id]`

- 🧭 **What:** Fetch one payer
- 🧪 **Input:** path `id`
- 🗃️ **Prisma:** `denialPayer.findUnique({ where: { id }, select: { id, name }})`
- 📤 **Output:** `{ id, name }`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** 400 for bad ID, 404 if not found

---

### 1️⃣2️⃣ **GET** `/api/denials/payers/[id]/reason-codes`

- 🧭 **What:** Reason codes for one or more payers (comma-separated IDs)
- 🧪 **Input:** path `id` like "1,2,3"
- 🗃️ **Prisma:** `denialPlanReasonCodeMapping.findMany(...)` then deduplicate
- 📤 **Output:** `[{ id, code, name }]`
- 🔐 **Auth:** `view_denials`
- 💡 **Tip:** Supports multiple payers in one call

---

### 1️⃣3️⃣ **GET** `/api/denials/payer-plans`

- 🧭 **What:** Unique plan names directly from denials
- 🧪 **Input:** none
- 🗃️ **Prisma:** `denial.findMany({ where: { planName: { not: null }}, distinct: ["planName"], select: { planName }, orderBy: planName asc })`
- 📤 **Output:** `[{ name: planName }]`
- 🔐 **Auth:** (no wrapper; harmless read)
- 💡 **Tip:** Used to build plan filters quickly

---

## 🎯 Prisma Quick Tips

- **Filtering nested relations:** `where: { denialPlan: { denialPayerId: { in: [...] }}}`
- **Pagination:** `skip = (pageNo - 1) * pageSize`, `take = pageSize`
- **Ordering by multiple:** `orderBy: [{ denialStatusOrder: { order: "asc" }}, { createdAt: "desc" }]`
- **Bulk updates:** `updateMany({ where, data })`
- **Counts** are cheap and useful for UI summaries

---

## 🔄 Example Flows (Visual)

### **Close one denial** ✅
1. **PUT** `/api/denials/123/review` with `{ files: ["PDR_FORM"] }`
2. **Denial** → `CLOSED`, **PDR form** → `CLOSED`

### **Bulk review many** 🔁
1. **GET** `/api/denials/bulk-review?payerId=2&startDate=2024-01-01&endDate=2024-12-31`
2. **PUT** `/api/denials/bulk-review` with same filters + files
3. **GET** `/api/denials/bulk-review/summary` (see totals + validation)
4. **PUT** `/api/denials/bulk-review/complete` (move `CLOSED` → `RESOLVED`)

---

**That's it!** If you want, I can add side-by-side Django-style pseudo code to each endpoint for super-fast mapping.

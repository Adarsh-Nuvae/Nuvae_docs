# Denials UI Components Guide (with emojis!) ✨

This guide explains every file in `src/components/denial`, how it works, and how it supports the Denials list page at `src/pages/denials/index.tsx`. It's friendly, simple, and packed with emojis so it's easy for anyone to follow.

---

## 🔗 Quick Links to Files

- **Barrel:** [`src/components/denial/index.ts`](../src/components/denial/index.ts)
- **Main list and actions:** [`src/components/denial/denials-queue.tsx`](../src/components/denial/denials-queue.tsx)
- **Filter bar:** [`src/components/denial/denial-filter.tsx`](../src/components/denial/denial-filter.tsx)
- **Payer dropdown:** [`src/components/denial/denial-payer-dropdown.tsx`](../src/components/denial/denial-payer-dropdown.tsx)
- **Financial class dropdown:** [`src/components/denial/financial-class-dropdown.tsx`](../src/components/denial/financial-class-dropdown.tsx)
- **File selection (checkboxes):** [`src/components/denial/file-selection.tsx`](../src/components/denial/file-selection.tsx)
- **File selection modal:** [`src/components/denial/file-selection-modal.tsx`](../src/components/denial/file-selection-modal.tsx)
- **Bulk review modal:** [`src/components/denial/bulk-review-modal.tsx`](../src/components/denial/bulk-review-modal.tsx)
- **Bulk review tabs (destination):** [`src/components/denial/review-tabs.tsx`](../src/components/denial/review-tabs.tsx)
- **Bulk review progress:** [`src/components/denial/review-summary.tsx`](../src/components/denial/review-summary.tsx)
- **Bulk review download confirm:** [`src/components/denial/download-confirm-modal.tsx`](../src/components/denial/download-confirm-modal.tsx)
- **Success message:** [`src/components/denial/success-message.tsx`](../src/components/denial/success-message.tsx)
- **Denial details (composed):** [`src/components/denial/denial-details-view.tsx`](../src/components/denial/denial-details-view.tsx)
- **Denial details (claim):** [`src/components/denial/denial-details-claim-view.tsx`](../src/components/denial/denial-details-claim-view.tsx)
- **Denial details (financial):** [`src/components/denial/denial-details-financial-summary-view.tsx`](../src/components/denial/denial-details-financial-summary-view.tsx)
- **Denial details (services):** [`src/components/denial/denial-details-service-view.tsx`](../src/components/denial/denial-details-service-view.tsx)
- **EOB viewer:** [`src/components/denial/denial-eob-view.tsx`](../src/components/denial/denial-eob-view.tsx)
- **Appeal tab content:** [`src/components/denial/appeal-tab-content.tsx`](../src/components/denial/appeal-tab-content.tsx)
- **Edit appeal dialog:** [`src/components/denial/edit-appeal-dialog.tsx`](../src/components/denial/edit-appeal-dialog.tsx)

---

## 🏛️ How the Denials Page Works (Big Picture)

- **Show** a table of denials with filters and pagination
- **Let you pick** one or many rows
- **Export files** for selected rows
- **Start** a bulk review flow
- **Then take you** to focused screens for reviewing and editing

---

## 🎭 Main Characters on the Page

- **DenialsView (page):** `src/pages/denials/index.tsx`
- **DenialsQueue (list UI):** `src/components/denial/denials-queue.tsx`
- **DenialFilter (toolbar):** `src/components/denial/denial-filter.tsx`
- **BulkReviewModal (setup):** `src/components/denial/bulk-review-modal.tsx`
- **FileSelectionModal (choose files):** `src/components/denial/file-selection-modal.tsx`

---

## 💬 How They Talk to Each Other

1. The page renders **DenialsQueue**
2. **DenialsQueue** renders **DenialFilter**, the table, export menu, and a "Review Selected" flow using **FileSelectionModal**
3. Clicking "Bulk Review Claims" on the page opens **BulkReviewModal**; when you click Start, it navigates to `/denials/bulk?...`

---

## 📁 File-by-File Tour (with emojis)

### 1️⃣ **Barrel export** — `index.ts`

- **What:** Re-exports many components from this folder
- **Why:** Makes imports easy, like `import { DenialsQueue } from "@/components/denial"`
- **Helps the page:** Cleaner imports on `src/pages/denials/index.tsx`

---

### 2️⃣ **DenialsQueue** — `denials-queue.tsx` 🧮

- **Think:** The big table and actions
- **Shows:** Title, filter bar, table with checkboxes, View Details, View Appeal, export menu, pagination
- **Actions:**
  - **Bulk Review Claims button** → tells the page to open BulkReviewModal
  - **Review Selected** → opens FileSelectionModal (pick PDR/Appeal) then calls back with selected rows and files
  - **Export menu** → downloads ZIPs for selected rows
- **Loads data** via `DenialsClient.getDenials` with filters
- **Helps the page:** It's the main UI for listing and selecting denials

---

### 3️⃣ **DenialFilter** — `denial-filter.tsx` 🧰

- **Think:** The toolbar for filters
- **Shows:** Search Claim ID 🔎, Appeal Amount, Payer Plan, Financial Class, Status, EOB Date range, Apply/Clear
- **Calls** `onApplyFilters` with the current filter state
- **Helps the page:** Drives which denials appear in the list

---

### 4️⃣ **DenialPayerPlanDropdown** — `denial-payer-dropdown.tsx` 🧾

- **Think:** A dropdown to pick payer plans
- **Features:** Search, multi-select, Select All, click-outside to close
- **Used by:** DenialFilter and BulkReviewModal
- **Helps the page:** Lets users narrow to specific payers quickly

---

### 5️⃣ **FinancialClassDropdown** — `financial-class-dropdown.tsx` 🏷️

- **Think:** A dropdown to pick financial classes
- **Features:** Search, multi-select, Select All; loads classes from `/api/financial-class`
- **Used by:** DenialFilter and BulkReviewModal
- **Helps the page:** Adds precise business-category filtering

---

### 6️⃣ **FileSelection** — `file-selection.tsx` ☑️

- **Simple checkboxes** for which files to include: PDR Form and Appeal Letter
- **Used by:** FileSelectionModal, BulkReviewModal
- **Helps the page:** Captures which documents matter for a review/export

---

### 7️⃣ **FileSelectionModal** — `file-selection-modal.tsx` 🗂️

- **A small pop-up** that wraps FileSelection
- **Confirm** → returns selected file types and closes
- **Used by:** DenialsQueue's "Review Selected" flow
- **Helps the page:** Lets the user confirm PDR/Appeal inclusion

---

### 8️⃣ **BulkReviewModal** — `bulk-review-modal.tsx` 🚦

- **One-stop setup** for bulk review filters: Payers, Reason Codes, Date range, Files (PDR/Appeal), Appeal Amount, Financial Classes
- **Fetches reason codes** based on selected payer(s)
- **Start Review** → sends these filters back so the page can navigate to `/denials/bulk?...`
- **Helps the page:** Cleanly gathers all info to start a bulk review

---

### 9️⃣ **ReviewTabs** — `review-tabs.tsx` 🧑‍⚖️

- **This powers** the Bulk Review screen (destination after setup)
- **Shows** patient/claim header, progress counts, and a main panel with tabs:
  - PDR Form, Appeal Letter, EOB (depending on what files you selected and the denial source)
- **Buttons:** Approve, Skip, Edit, Next/Previous
- **Uses** ReviewSummary on the left
- **Helps the page:** Not used on the list; it's where the real reviewing happens after you leave the list

---

### 🔟 **ReviewSummary** — `review-summary.tsx` 📊

- **A compact panel** with:
  - Overall progress bar
  - Total/Reviewed/Edited/Pending counts
- **Helps the page:** Part of bulk review UX, not the list

---

### 1️⃣1️⃣ **DownloadConfirmModal** — `download-confirm-modal.tsx` 📥✅

- **After bulk approvals,** confirms downloading selected files
- **Shows** validation results (missing fields), plus a review summary with estimated recovery
- **Clicking Download** can trigger resolving the denials
- **Helps the page:** End-of-flow confirmation in bulk review

---

### 1️⃣2️⃣ **SuccessMessage** — `success-message.tsx` 🎉

- **A little banner** that fades out (e.g., "Files downloaded!")
- **Helps the page:** Positive feedback after finishing tasks in bulk review

---

### 1️⃣3️⃣ **DenialDetailsView** — `denial-details-view.tsx` 🧩

- **Composes** the details page with three parts:
  - Claim view
  - Financial summary
  - Services table
- **Helps the page:** The list page's "View Details" goes to the details screen that uses this component

---

### 1️⃣4️⃣ **DenialDetailsClaimView** — `denial-details-claim-view.tsx` 🧾

- **Card showing:** Date Paid, Service Period, Claim Number, Patient Account Number, Patient Name, Health Plan, Plan Responsibility
- **Helps the page:** Part of the details screen

---

### 1️⃣5️⃣ **DenialDetailsFinancialSummaryView** — `denial-details-financial-summary-view.tsx` 💵

- **Card showing:** Billed, Allowed, Net, Interest, and Recommended (appeal) amounts
- **Helps the page:** Part of the details screen

---

### 1️⃣6️⃣ **DenialDetailsServiceView** — `denial-details-service-view.tsx` 🧑‍⚕️

- **Big table** with:
  - Service Date, Description, Billed/Paid/Recommended amounts, Adjustment Reason, Net, Interest
  - Tabs for All Services vs Appealable Services
  - Expand rows to read the Appeal Recommendation
- **Helps the page:** Part of the details screen

---

### 1️⃣7️⃣ **DenialEobView** — `denial-eob-view.tsx` 📄🖼️

- **Detects** if the source is a PDF or image and shows the right viewer
- **Used by:** Details page and bulk review tabs (if source is EOB)
- **Helps the page:** Indirectly — useful after clicking "View Details"

---

### 1️⃣8️⃣ **AppealTabContent** — `appeal-tab-content.tsx` ✍️

- **Tab content** for the appeal letter
- **Loads** letter text; can be editable; updates the first line to today's date
- **Used by:** EditAppealDialog, ReviewTabs
- **Helps the page:** Shows letter content where needed

---

### 1️⃣9️⃣ **EditAppealDialog** — `edit-appeal-dialog.tsx` 🧑‍💻

- **Full-screen editor** with tabs: PDR Form, Appeal Letter, EOB
- **Save Letter, Regenerate Letter**
- **Used by:** Bulk review when you click Edit
- **Helps the page:** Deep editing when needed

---

## 🚀 User Journey with Emojis

**Filter the list** 🔎 → **see denials** 📋 → **select some** ✅ → **choose files** 🗂️ → **start bulk review** 🚀 → **approve/edit** ✍️ → **download** 📦 → **success** 🎉


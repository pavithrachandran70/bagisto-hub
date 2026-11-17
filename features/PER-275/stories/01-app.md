# Story PER-275 — Show the channel on a customer's storefront orders

## Metadata

```yaml
id: 275
type: story
epic: (none — backlog item)
title: Show the channel on a customer's storefront orders
status: planned
track: quick
hitl: false
assignee: (unassigned)
created: 2025-01-16
updated: 2025-01-16
```

## Description

A customer who shops more than one channel cannot tell which store an order came from. Admin order lists already show the channel. The storefront account does not: `packages/Webkul/Shop/src/Resources/views/customers/account/orders` never renders the channel.

Show the channel name on the customer's order list and on the order detail page, using the channel stored on the order.

## Requirements

### Background

Admin order lists already show the channel. The storefront views under `packages/Webkul/Shop/src/Resources/views/customers/account/orders` do not render it, though the order model already stores `channel` (verify field name and relationship in order model or schema).

### Acceptance Criteria

#### AC1: Order list shows channel name

**Given** a customer views their order list at the storefront account  
**When** the page renders  
**Then** each order displays its channel name beside or below the order identifier

#### AC2: Order detail shows channel name

**Given** a customer views a single order's detail page  
**When** the page renders  
**Then** the same channel name appears on the detail view

#### AC3: Single-channel store behavior

**Given** a store configured with only one channel  
**When** a customer views their orders  
**Then** that channel's name still appears with no empty or missing label

#### AC4: Localized labels

**Given** the storefront is running in any locale Bagisto ships  
**When** the channel name or its label is displayed  
**Then** the label (e.g., "Channel:", "Store:") is translated into that locale

## Technical Specification

### Architecture

**Type:** UI-only change  
**Scope:** Customer account order views in Shop package  
**Contract impact:** None — uses existing order model `channel` field

### Implementation Plan

#### Files to Modify

1. **Order list view** — `packages/Webkul/Shop/src/Resources/views/customers/account/orders/index.blade.php` (verify exact filename)
   - Add channel name display in the order row template
   - Use existing order-channel relationship accessor (verify in order model)
   - Follow existing Blade patterns in this view for rendering order fields

2. **Order detail view** — `packages/Webkul/Shop/src/Resources/views/customers/account/orders/view.blade.php` (verify exact filename)
   - Add channel name display in the order header or summary section
   - Use same relationship accessor as list view
   - **Handle null channel gracefully:** When both `channel_name` column and `channel` relationship are null, display a fallback value (e.g., the default channel name or a translated placeholder) to satisfy AC3 — no empty or missing label should appear

3. **Translation files** — Add label key to each shipped locale's customer account translations:
   - Base locale: `packages/Webkul/Shop/src/Resources/lang/en/app.php` (or appropriate customer section file)
   - All other shipped locales under `packages/Webkul/Shop/src/Resources/lang/<locale>/`
   - Key example: `'channel' => 'Channel'` or `'order-channel' => 'Store'`

4. **Order list data source** (if applicable) — Controller or DataGrid serving the order list:
   - **Ensure channel name is always populated:** When the `channel_name` column is null, fall back to loading the channel relationship and using its `name` attribute, so the view always receives a non-null channel name
   - Verify the data-fetching logic (e.g., in `packages/Webkul/Shop/src/Http/Controllers/Customers/OrderController.php` or a DataGrid class) to confirm how orders are loaded and whether eager-loading or a computed accessor is needed

#### Technical Notes

- **Channel field:** The order model already has a `channel` relationship (verify exact name and type in order model — likely `BelongsTo` relationship to `Webkul\Core\Models\Channel`)
- **Display location:** Place channel name logically near order number/date in list view, and in the order header or summary block in detail view
- **No field name prescription:** Developer chooses appropriate Blade syntax (`{{ }}` vs `{!! !!}`) and relationship accessor (e.g., `$order->channel->name`) based on existing patterns in these views
- **AC3 compliance — no empty labels:** When both `channel_name` column and the `channel` relationship are null (edge case), the implementation must display a fallback value (e.g., the name of the application's default channel, or a translated string like "Default Store") rather than rendering an empty string or null after the "Channel:" label
- **Locale coverage:** Verify shipped locales by checking `packages/Webkul/Shop/src/Resources/lang/` directory structure; add translation key to every locale found

### Dependencies

**Blockers:** None — order model `channel` field already exists

**Validation requirements:**
- Order model must have `channel` relationship defined (verify in order model)
- Views `index.blade.php` and `view.blade.php` must exist under `packages/Webkul/Shop/src/Resources/views/customers/account/orders/` (verify paths)
- Shipped locales directory structure under `packages/Webkul/Shop/src/Resources/lang/` (verify list)

## Tests

### Test 1: Multi-channel order list displays channel names (AC1)

**Purpose:** Verify that the order list view shows the channel name for each order when multiple channels exist.

**Setup:**
- Create two channels in admin (e.g., "Main Store", "Wholesale")
- Place at least one order on each channel for the same customer account
- Ensure orders have valid `channel` relationship or `channel_name` populated

**Execution:**
1. Log in as the customer on the storefront
2. Navigate to the customer account orders page (`/customer/account/orders` or equivalent route)
3. Observe the order list display

**Expected Result:**
- Each order row displays its associated channel name
- Channel name appears beside or below the order identifier
- No empty or null values where channel name should appear
- Both orders from different channels show their respective channel names correctly

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/OrderListChannelDisplayTest.php`

---

### Test 2: Order detail page displays channel name (AC2)

**Purpose:** Verify that the order detail view shows the channel name consistently with the list view.

**Setup:**
- Use the same multi-channel orders from Test 1
- Ensure at least one order has a valid channel relationship

**Execution:**
1. Log in as the customer on the storefront
2. Navigate to the customer account orders page
3. Click to view the detail page of an order
4. Observe the order header or summary section

**Expected Result:**
- The channel name appears on the detail view
- The channel name matches what was shown in the order list
- Display uses the same translation key and format as the list view
- No empty or null values in the channel display area

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/OrderDetailChannelDisplayTest.php`

---

### Test 3: Single-channel store shows channel name (AC3)

**Purpose:** Verify that a store with only one channel still displays the channel name with no empty labels.

**Setup:**
- Configure the store to have only one active channel
- Place at least one order as a customer on that single channel
- Ensure the order has a valid channel relationship

**Execution:**
1. Log in as the customer on the storefront
2. Navigate to the customer account orders page
3. View both the order list and detail page

**Expected Result:**
- The single channel's name appears on both list and detail views
- No empty label (e.g., "Channel: " with nothing after it)
- Channel name is visible and properly formatted
- If `channel_name` is null but relationship exists, fallback mechanism provides the name

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/SingleChannelOrderDisplayTest.php`

---

### Test 4: Channel labels are translated in all shipped locales (AC4)

**Purpose:** Verify that the channel label (e.g., "Channel:", "Store:") is translated correctly in every locale Bagisto ships.

**Setup:**
- Identify all shipped locales by scanning `packages/Webkul/Shop/src/Resources/lang/` directory
- Add the channel label translation key to each locale file
- Place at least one order for a customer account
- Prepare to test in multiple locales (at minimum: en, ar, de, es, fr, hi_IN)

**Execution:**
1. For each shipped locale:
   - Switch the storefront to that locale
   - Log in as the customer
   - Navigate to the customer account orders page
   - View both order list and detail page
   - Observe the channel label text

**Expected Result:**
- The channel label (e.g., "Channel:", "Store:", or equivalent) appears in the correct language for each locale
- The channel name itself is not translated (it's data), but the label is
- No English fallback text appears in non-English locales for the label
- All shipped locales have the translation key defined and render correctly

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/OrderChannelLocalizationTest.php`

---

### Edge Case Test: Null channel fallback (AC3 edge case)

**Purpose:** Verify that when both `channel_name` column and `channel` relationship are null, a fallback value is displayed to prevent empty labels.

**Setup:**
- Create an order record where both `channel_name` and the `channel_id` (or equivalent foreign key) are null
- This simulates a data inconsistency or migration state
- Identify the fallback mechanism (e.g., default channel name, translated placeholder like "Default Store")

**Execution:**
1. Log in as the customer on the storefront
2. Navigate to the customer account orders page
3. View the order with null channel in both list and detail views
4. Observe what text appears in the channel display area

**Expected Result:**
- A fallback channel name or placeholder text appears (e.g., "Default Store", "Main Channel", or similar)
- No empty string or null renders after the "Channel:" label
- No PHP error or exception occurs
- The fallback text is translated if it uses a translation key

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/OrderNullChannelFallbackTest.php`

---

### Edge Case Test: Long channel names do not break layout

**Purpose:** Verify that unusually long channel names do not cause layout issues in the order list or detail views.

**Setup:**
- Create a channel with a very long name (e.g., 100+ characters)
- Place an order on that channel for a customer account

**Execution:**
1. Log in as the customer on the storefront
2. Navigate to the customer account orders page
3. View the order list
4. Click to view the order detail page
5. Observe the channel name display in both views

**Expected Result:**
- The long channel name displays without breaking the page layout
- Text wraps appropriately or is truncated with ellipsis if necessary
- No horizontal scrolling or overflow issues
- Order information remains readable and accessible

**Test file:** `packages/Webkul/Shop/tests/Feature/Customers/LongChannelNameLayoutTest.php`

---

## Test File Plan

The following test files will be created under `packages/Webkul/Shop/tests/Feature/Customers/`:

1. **OrderListChannelDisplayTest.php** — Tests AC1 (multi-channel order list)
2. **OrderDetailChannelDisplayTest.php** — Tests AC2 (order detail page)
3. **SingleChannelOrderDisplayTest.php** — Tests AC3 (single-channel store)
4. **OrderChannelLocalizationTest.php** — Tests AC4 (localized labels)
5. **OrderNullChannelFallbackTest.php** — Tests AC3 edge case (null channel fallback)
6. **LongChannelNameLayoutTest.php** — Tests layout edge case (long channel names)

Each test file contains the test code once, following Pest or PHPUnit conventions based on the existing test structure in `packages/Webkul/Shop/tests/`.

## Notes

- **Focus per plan:** Exact views to modify are `packages/Webkul/Shop/src/Resources/views/customers/account/orders/index.blade.php` and `view.blade.php`; channel field already on order model; translation required for all shipped locales
- **No schema changes:** This is presentation-layer only
- **Reuse admin pattern:** Reference how admin order lists display channel for consistency in approach (not necessarily styling)
- **PR comment resolution:** Ensure the order detail view handles the case where both `channel_name` and `channel` relationship are null by displaying a non-empty fallback value (AC3). Ensure the order list data source (DataGrid or controller) populates the channel name from the relationship when the `channel_name` column is null, so the list view never displays null/empty (AC1 + AC3).

## Assumptions

- Order model `channel` field name and relationship type will be verified in order model during implementation
- Exact view filenames (`index.blade.php`, `view.blade.php`) will be confirmed from `packages/Webkul/Shop/src/Resources/views/customers/account/orders/` directory structure
- Shipped locales list will be derived from `packages/Webkul/Shop/src/Resources/lang/` directory at implementation time
- When both `channel_name` and the `channel` relationship are null, the implementation will use a fallback strategy (e.g., application default channel name or a translated constant) to satisfy AC3

## Open Questions

None — ticket scope and existing codebase context are sufficient for implementation.

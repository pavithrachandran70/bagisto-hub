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

Admin order lists already show the channel. The storefront views under `packages/Webkul/Shop/src/Resources/views/customers/account/orders` do not render it, though the order model already stores `channel` (verify field name and relationship in `app/Models/Order.php` or schema).

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
   - Use existing order-channel relationship accessor (verify in `app/Models/Order.php`)
   - Follow existing Blade patterns in this view for rendering order fields

2. **Order detail view** — `packages/Webkul/Shop/src/Resources/views/customers/account/orders/view.blade.php` (verify exact filename)
   - Add channel name display in the order header or summary section
   - Use same relationship accessor as list view

3. **Translation files** — Add label key to each shipped locale's customer account translations:
   - Base locale: `packages/Webkul/Shop/src/Resources/lang/en/app.php` (or appropriate customer section file)
   - All other shipped locales under `packages/Webkul/Shop/src/Resources/lang/<locale>/`
   - Key example: `'channel' => 'Channel'` or `'order-channel' => 'Store'`

#### Technical Notes

- **Channel field:** The order model already has a `channel` relationship (verify exact name and type in `app/Models/Order.php` — likely `BelongsTo` relationship to `Webkul\Core\Models\Channel`)
- **Display location:** Place channel name logically near order number/date in list view, and in the order header or summary block in detail view
- **No field name prescription:** Developer chooses appropriate Blade syntax (`{{ }}` vs `{!! !!}`) and relationship accessor (e.g., `$order->channel->name`) based on existing patterns in these views
- **Single-channel handling:** No special conditional logic needed — if a store has one channel, that channel's name renders normally
- **Locale coverage:** Verify shipped locales by checking `packages/Webkul/Shop/src/Resources/lang/` directory structure; add translation key to every locale found

### Dependencies

**Blockers:** None — order model `channel` field already exists

**Validation requirements:**
- Order model must have `channel` relationship defined (verify in `app/Models/Order.php`)
- Views `index.blade.php` and `view.blade.php` must exist under `packages/Webkul/Shop/src/Resources/views/customers/account/orders/` (verify paths)
- Shipped locales directory structure under `packages/Webkul/Shop/src/Resources/lang/` (verify list)

## Tests

### Verification Steps

1. **Multi-channel test:**
   - Create two channels in admin
   - Place orders on each channel for the same customer account
   - Log in as customer on storefront
   - View order list — confirm each order shows its channel name
   - Open each order's detail page — confirm channel name appears

2. **Single-channel test:**
   - Configure store with only one channel
   - Place order as customer
   - View order list and detail — confirm channel name renders (not empty)

3. **Localization test:**
   - Identify all shipped locales in `packages/Webkul/Shop/src/Resources/lang/`
   - For each locale: switch storefront locale, view orders, confirm channel label is translated

### Edge Cases

- **Missing channel:** If an order somehow lacks a channel reference, verify graceful handling (no fatal error); log warning if encountered during test
- **Long channel names:** Test with channel name exceeding typical length to verify layout does not break

## Notes

- **Focus per plan:** Exact views to modify are `packages/Webkul/Shop/src/Resources/views/customers/account/orders/index.blade.php` and `view.blade.php`; channel field already on order model; translation required for all shipped locales
- **No schema changes:** This is presentation-layer only
- **Reuse admin pattern:** Reference how admin order lists display channel for consistency in approach (not necessarily styling)

## Assumptions

- Order model `channel` field name and relationship type will be verified in `app/Models/Order.php` during implementation
- Exact view filenames (`index.blade.php`, `view.blade.php`) will be confirmed from `packages/Webkul/Shop/src/Resources/views/customers/account/orders/` directory structure
- Shipped locales list will be derived from `packages/Webkul/Shop/src/Resources/lang/` directory at implementation time

## Open Questions

None — ticket scope and existing codebase context are sufficient for implementation.

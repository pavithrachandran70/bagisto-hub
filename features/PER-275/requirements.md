# requirements.md

## Overview

Display the channel name on the customer's storefront order list and order detail pages so customers who shop multiple channels can identify which store each order came from.

## Background

Admin order lists already show the channel. The storefront views under `packages/Webkul/Shop/src/Resources/views/customers/account/orders` do not render it, though the order model already stores `channel` (verify in order model and schema).

## Acceptance Criteria

### AC1: Order list shows channel name

**Given** a customer views their order list at the storefront account  
**When** the page renders  
**Then** each order displays its channel name beside or below the order identifier

### AC2: Order detail shows channel name

**Given** a customer views a single order's detail page  
**When** the page renders  
**Then** the same channel name appears on the detail view

### AC3: Single-channel store behavior

**Given** a store configured with only one channel  
**When** a customer views their orders  
**Then** that channel's name still appears with no empty or missing label

### AC4: Localized labels

**Given** the storefront is running in any locale Bagisto ships  
**When** the channel name or its label is displayed  
**Then** the label (e.g., "Channel:", "Store:") is translated into that locale

## Verification

1. Create orders on two different channels for the same customer account.
2. View the customer's order list in the storefront account — confirm each order shows its channel name.
3. Open one order's detail page — confirm the channel name appears there too.
4. Configure a single-channel store, place an order, and verify the channel name still renders.
5. Switch locale to each of Bagisto's shipped languages — confirm the channel label is translated.

## Notes

- The order model's `channel` field (verify exact field name and relationship in `app/Models/Order.php` or schema) holds the channel reference.
- Modify the Blade views under `packages/Webkul/Shop/src/Resources/views/customers/account/orders` — the list view and the detail view.
- Translation keys go in `packages/Webkul/Shop/src/Resources/lang/<locale>/app.php` or the appropriate customer account translation file for each shipped locale.
- Do NOT prescribe which Blade syntax (e.g., `{{ }}` vs `{!! !!}`) or relationship accessor (e.g., `->channel->name`) to use — the developer chooses based on existing patterns in those views.

---
key: PER-275
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: "Title:\_Show the channel on a customer\u2019s storefront orders"
---

Description:

A customer who shops more than one channel cannot tell which store an order came from. Admin order lists already show the channel. The storefront account does not: `packages/Webkul/Shop/src/Resources/views/customers/account/orders` never renders the channel.

Show the channel name on the customer’s order list and on the order detail page, using the channel stored on the order.

Acceptance criteria

* The order list shows the channel name for each order.
* The order detail page shows the same channel name.
* A store with one channel still shows that channel, with no empty label.
* The label is translated in every locale Bagisto already ships.

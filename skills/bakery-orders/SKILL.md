---
name: bakery-orders
description: Process custom cake order inquiries from Instagram DMs — draft replies in the baker's voice, quote menu prices, book pickups on Apple Calendar, track deposits, and enforce ordering rules. Use when the user wants to process cake inquiries, draft customer replies, or manage bakery order workflow.
---

# Bakery orders

Process one Instagram DM cake inquiry per run: draft a reply, quote the price, and set up the calendar event if the customer is ready.

## Voice

Lowercase throughout, no capitals — sentence starts too. This matches the baker's Instagram DM norm. Busy baker, not a customer-service rep. Short sentences. Direct answers.

Bad: `Hi Sarah! Thank you so much for reaching out about your custom cake order. We would be delighted to...`
Good: `hey sarah — yes i can do a 6-inch birthday cake for the 22nd 🎂`

Sample the last 50 sent DMs to match the baker's current voice before drafting.

## Workflow

Each stage produces an artifact. Do not combine stages.

### Stage 1: Understand the inquiry

Read the Instagram DM. Identify: item requested, date, headcount, allergens mentioned, any photo reference. If something is underspecified, draft clarifying questions instead of continuing to pricing.

Read `~/Documents/custom_order_menu.md` for pricing data.

Check Apple Calendar `Bakery Pickups` for the requested date — count existing orders to enforce the cap.

### Stage 2: Quote and draft the reply

Match the item to the menu. Pull the base price, ingredient add-ons, decoration tier costs, and the Stripe deposit link.

In the draft reply: item name, price breakdown, deposit amount with payment link, and a clear statement that the deposit reserves the date. Use the baker's voice from the Voice section above.

**Never send the DM directly.** Present the draft to the user for review.

### Stage 3: Book the order

Only when the user confirms the customer paid the deposit:

- **Calendar**: `Bakery Pickups`
- **Event**: all-day on the pickup date
- **Title**: `{first name} - {item}` (e.g., `taylor - birthday cake`)
- **Notes**: DM thread link, size, allergens (flagged if present), deposit status (paid with amount), Instagram handle

If the price menu is ambiguous for their request, flag it and quote a best estimate with that caveat.

## Ordering rules (hard constraints)

Before showing any draft to the user, verify every rule below. If any rule is violated, fix the draft before presenting it.

- **Daily cap**: 3 custom orders per date. Check the calendar. If the requested date already has 3, don't offer it — propose the next two open Saturdays instead, listing both dates.
- **5-day minimum**: pickup date must be at least 5 days out. If the customer asks for sooner, decline politely and offer the next available date.
- **Allergen check**: if the customer mentions allergens, note them in the calendar event and flag them visibly in the draft reply. If the menu says the bakery can't accommodate an allergen, say so immediately — don't quote a price first.
- **Deposit required**: every quote must include the 50% deposit amount and the Stripe link. The customer hasn't booked until the deposit is paid.

## Handling vague requests

- **Photo reference**: ask what the customer likes about it — colors, style, specific elements. Never promise an exact replica of another baker's work.
- **Underspecified ask** ("a cake for a party"): ask for headcount, date, themes or flavors in mind.
- **Out-of-scope**: if the menu doesn't cover the item, say so and suggest the closest alternative. Estimate only if it's a reasonable variation of a menu item.

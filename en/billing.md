# Billing and Wallet

Billing in Komuta runs at the organization level: your services, managed services and other resources are metered by the hour, a statement for the previous cycle is issued on the 6th of every month, and the amount is charged to your default card automatically. This page covers what is billed, how to read and pay statements, the wallet, payment methods, the billing profile, the spending limit, and what happens when a statement goes unpaid.

---

## Billing Pages

Open **My Account** from the user menu in the top right. The **Billing** group in the left menu holds these pages:

| Page | Address | What it is for |
|---|---|---|
| **Billing Profile** | [console.komuta.io/account/billing/profile](https://console.komuta.io/account/billing/profile) | The legal identity, address, tax ID and billing emails used on your invoices. |
| **Current Spend** | [console.komuta.io/account/billing/current](https://console.komuta.io/account/billing/current) | What the current cycle has cost so far, and your build minutes. |
| **Observability Storage** | [console.komuta.io/account/billing/observability-storage](https://console.komuta.io/account/billing/observability-storage) | How much telemetry each service stores and its monthly price. |
| **Wallet** | [console.komuta.io/account/billing/wallet](https://console.komuta.io/account/billing/wallet) | Balance, adding funds, the spending limit, your account tier and coupons. |
| **Statements** | [console.komuta.io/account/billing/invoices](https://console.komuta.io/account/billing/invoices) | Issued statements, their breakdown, and payment. |
| **Payment Methods** | [console.komuta.io/account/billing/payment-methods](https://console.komuta.io/account/billing/payment-methods) | Saved cards and the default card. |

Billing pages are open to **organization admins** only. Members do not see these pages in the menu; when a statement is unpaid, the banner at the top tells them "Ask an organization admin to settle the balance — billing pages are admin-only." Admins also see the balance, the account tier and an **Add funds** button in the user menu.

> **Note:** Each organization has its own wallet, statements, cards and spending limit. If you belong to more than one organization, the billing pages show the organization that is currently selected.

---

## How You Are Billed

- **Currency:** USD. Prices exclude tax; tax is calculated from the address on your billing profile and shown as a separate line on the statement.
- **Metering:** Usage is metered by the hour. Monthly prices are based on 720 hours; a resource you create or delete mid-month is billed only for the hours it ran.
- **You see the price first:** The service wizard shows package prices, and the managed service wizard and the Stack plan show the **Estimated monthly cost**.

| What is billed | Price |
|---|---|
| Services and jobs | By the resource package you choose; the price is shown in the creation wizard. Service packages: 0.5 vCPU / 1 GB USD 9, 1 vCPU / 2 GB USD 18, 2 vCPU / 4 GB USD 36, 4 vCPU / 8 GB USD 72 (per month). A service running more than one replica is metered for each replica. |
| Managed services (PostgreSQL, Valkey, RabbitMQ) | On the plan's CPU limit, memory limit and disk: USD 12.85 per vCPU per month, USD 5.14 per GiB of memory per month, USD 0.1447 per GiB of disk per month. |
| API Gateway | USD 9 per month (USD 0.0125 per hour) plus the compute the gateway uses. |
| Build minutes | 2,000 minutes included in every billing cycle, then USD 0.008 per minute. Time waiting in the queue is not billed. |
| Bandwidth plans | Starter free (default), Growth USD 4.99, Business USD 14.99, Enterprise USD 49.99, Scale USD 149.99 (per month). The plan fee accrues hourly even without traffic. |
| Observability storage | USD 0.000342 per GB-hour (about USD 0.25 per GB per month). |
| Always on add-on | USD 3 per billing cycle + tax, for only the part of the cycle it is on. |

Every organization can run one service on the free package; that service is not billed. The free plan rules and the conditions of the **Always on** add-on are on the [Service Sleep](service-sleep.md) page. The full, current price list is on the [Pricing](https://www.komuta.io/pricing) page.

---

## Billing Cycle and Statements

A billing cycle starts on the 6th of the month at 00:00 (UTC) and ends on the 6th of the next month. Once a cycle closes, its statement is issued on the 6th.

- If there is no billable usage in the cycle, no statement is issued.
- If the total is under USD 0.50, no statement is issued; the usage carries over to the next cycle.
- Usage whose metering completes late is added to the next statement. When a statement covers usage from more than one cycle, the line description names the cycle's month (for example `[2026-09]`).

When a statement is issued, the amount is settled in this order:

1. Any coupon or campaign discount comes off.
2. The **Credit** in your wallet (coupon, campaign and bonus credit) comes off before tax.
3. Tax is added to what remains.
4. **Your funds** in the wallet are applied to the amount after tax.
5. Whatever is left is charged to your default card automatically.

A statement's **Due** date is the day it is issued; there is no separate payment term.

### The Statements page

**Account → Statements** lists one statement per row: **Invoice**, **Period**, **Status**, **Amount** and **Date**. The **Invoice** column shows the invoice number; until the invoice has been issued it reads **Not issued yet**. Your first statement appears here once your first billing cycle closes.

Click a row to open the statement:

- **Total**, **Subtotal**, **Tax**, **Due** and, once paid, the **Paid** date. Until tax has been calculated, the **Tax** field shows "—".
- The split of cost by category: **Managed Services**, **Services & Jobs**, **API Gateway**, **Network**, **Observability**.
- The resources in each category; click a resource to see its meters (**Meter**, **Qty**, **Unit Price**, **Amount**).
- A coupon or campaign discount, if any, as a separate negative line that comes off the whole statement.

The info icon next to the total is a reminder that this view is a usage breakdown: the invoice itself, with its tax, number and PDF, is issued by our payment provider and opens with **View invoice**.

### Statement statuses

| Status | Meaning |
|---|---|
| **Draft** | The statement is being prepared. You normally do not see it on issued statements. |
| **Issued** | The statement was issued and sent for collection. Your default card is charged; if that fails, the charge is retried automatically over the next few days. |
| **Paid** | The amount was collected. |
| **Uncollectible** | Retries are over and the amount could not be collected. The organization enters a grace period; you can still pay the statement (see the **Unpaid Statements** section). |
| **Canceled** | Komuta canceled the statement; there is nothing to pay. |

### Paying a statement yourself

When you open an open statement (**Issued** or **Uncollectible**), these buttons appear depending on its state:

| Button | When it appears | What it does |
|---|---|---|
| **View & pay invoice** | Once the invoice has been issued | Opens the invoice in a new tab. You can pay by card, with 3-D Secure if needed, and download the PDF. On paid statements the button is called **View invoice**. |
| **Pay from balance ({amount})** | Once the invoice has been issued and you have **Your funds** in the wallet | Re-issues the invoice with your balance applied. If the balance covers the whole amount, the message "Paid from your balance. Nothing is left to pay on this statement." appears. |
| **Pay now** | The statement is owed but no invoice has been issued yet | Issues the invoice; your balance, if any, is applied automatically. Then the message "Your invoice is ready. Use "View & pay invoice" to pay it." appears. |

> **Note:** **Pay now** does not work while the address on your billing profile is incomplete; the console takes you to the **Billing Profile** page.

> **Warning:** Funds you add to the wallet later do not settle a statement whose invoice has already been issued. Pay such a statement with **Pay from balance** or **View & pay invoice**. If you add funds while an invoice is open, the add funds window reminds you of this too.

---

## Current Spend

**Account → Current Spend** shows what the current billing cycle has cost so far, broken down by product.

- **Current Cycle Total** — the amount so far this cycle. **Billing Period** shows the start and end of the cycle, and **Tracked Until** shows the hour up to which metering is complete. The hour you are in is not included yet.
- **Build minutes** — minutes used this cycle and how many are left in, or over, the included allowance. Time is counted from the moment a build starts until it finishes; time waiting in the queue is not billed, and a cancelled build is billed only until it was cancelled. Minutes beyond the included allowance are billed on this cycle's statement.
- **Cost Breakdown** and **Past Cycles** — how this cycle's spend splits across products, and the totals of recent cycles.
- **All Usage** — every metered line item behind the total: **Product**, **Plan / Resource**, **Meter**, **Quantity**, **Unit Price**, **Amount**. Several meters of the same resource are grouped into one row; click the row to expand it.

The amount on this page is usage before tax, credit and discounts.

---

## Wallet

**Account → Wallet** brings together your balance, spending limit, account tier, coupons and every wallet movement.

### Balance: your funds and credit

The **Balance** is made of two separate pools. Click the info icon to open **Balance breakdown**.

| | **Your funds** | **Credit** |
|---|---|---|
| Where it comes from | Money you added with your card | Sign-up bonus, coupon and campaign credit |
| Expiry | Never expires | Promotional credit has an expiry date |
| How it is used on a statement | After tax | Used first, before tax is added |

- When you hold credit, the two pools are shown separately under the balance. If some credit is about to expire, a "{amount} expires {date}" tag appears.
- Promotional credit that expires unspent is removed from the balance; it shows in **Transactions** as **Promotional credit expired**.
- The first organization created at sign-up receives a USD 10 welcome credit valid for 30 days (**Sign-up bonus**).

### Adding funds

The address on the **Billing Profile** page must be complete before you can add funds.

1. On **Account → Wallet**, click **Add funds**.
2. Pick one of the preset amounts (USD 10, 25, 50, 100, 300) or type an amount in **Custom amount**. A single top-up is at least USD 5 and at most USD 1,000.
3. If you have saved cards, choose one under **Pay with**; to pay with a different card, choose **Use a new card**.
4. Click **Continue to payment**.
5. If you chose a new card, enter its details, then click **Pay {amount}**. Complete 3-D Secure if your bank asks for it.
6. When the payment completes, "**{amount} added**" and your **New balance** appear. If confirming the payment takes longer, **Payment received** appears instead; the balance shows up on its own as soon as the payment settles, and nothing more is needed from you.

No separate tax is charged on the top-up itself; the funds are applied to the amount after tax on your future statements. Top-ups also raise your account tier (see the **Spending Limit and Account Tiers** section).

If you see the error "Add a payment method first: loading funds needs a billing account, which is created when you save your first card in Account → Billing.", add a card on **Account → Payment Methods** first.

### Transactions

**Transactions** lists every movement in and out of the wallet. Filter with the **All**, **Money in** and **Money out** tabs, and sort by date, amount or expiry. Promotional credits carry a **Promo** tag. Click a row to see its **Transaction ID** and, where they exist, the **Payment ID** and **Invoice ID**; for movements tied to a statement, **View invoice** takes you to that statement.

| Movement | Meaning |
|---|---|
| **Added funds** | Money you added with your card. |
| **Sign-up bonus** | The welcome credit given at sign-up. |
| **Coupon credit** / **Campaign credit** | Credit from a coupon, promotion code or campaign. |
| **Invoice payment** | Credit applied to a statement. |
| **Balance applied to invoice** | Your funds applied to an invoice. |
| **Promotional credit expired** | Credit that expired unspent. |
| **Refund** | Money refunded to you. |
| **Manual adjustment** | A balance correction made by the Komuta team. |

---

## Payment Methods

**Account → Payment Methods** lists the cards saved for invoice payments and wallet top-ups. Statements are always charged to the **default** card.

To add a card:

1. On **Account → Billing Profile**, make sure the country, address, city and postal code are filled in. Otherwise the card form does not open and the error "Add your billing address before you save a card or load funds. …" appears.
2. Click **Add card**.
3. Enter the card details and click **Save card**. Complete 3-D Secure if your bank asks for it.

Komuta never stores your full card number; the card is kept securely with our payment provider.

- **Verification:** When a card is added, a temporary USD 1 authorization is placed and released right away; no money is taken from the card. If the authorization succeeds, the card gets the **Verified** tag, and an account on the **Starter** tier moves up to **Verified**. A card that cannot be verified is still saved.
- **Default card:** The card you add becomes the default and carries the **Default** tag. To make another card the default, click **Set Default** on that card.
- **Removing a card:** Hover over the card, click the cross icon and confirm with **Remove card**. If you remove the default card, your most recently added other card becomes the default.

> **Warning:** If you remove your only card, automatic collection stops: your next statement will go unpaid and your services can be restricted until you add a card again.

---

## Billing Profile

**Account → Billing Profile** holds the legal identity printed on your invoices and your billing emails. Keep it current so invoices are valid for accounting.

The **Legal identity** section:

| Field | Description |
|---|---|
| **Legal name** | Registered company or person name. |
| **Customer type** | **Company** or **Individual**. |
| **Tax ID** | Shown only for **Company**; when the country is `TR`, it is labeled **Tax ID (VKN)**. |
| **Tax ID type** | Shown only for **Company**; required when you enter a tax ID: **UK VAT**, **EU VAT** or the Turkish VKN / TIN type. |
| **Address line 1**, **Address line 2**, **City**, **Postal code** | The billing address. |
| **Country** | The 2-letter ISO country code (for example `TR`, `DE`, `GB`). |
| **Billing language** | Turkish or **English**; the language of your invoices. |

Click **Save Changes** to save.

**Country**, **Address line 1**, **City** and **Postal code** are required (no postal code is asked for in countries without a postal code system). Because tax is worked out from this address, while it is incomplete:

- You cannot add a card, add funds or pay a statement with **Pay now**.
- Your usage keeps being metered, but no invoice can be issued; the statement stays **Not issued yet** on the **Statements** page. The **Your billing address is missing** banner and an **Add billing address** button appear at the top, and the profile page shows **Billing is on hold — your address is incomplete**.
- A statement whose invoice cannot be issued moves the organization into a grace period 10 days after it was created (see the **Unpaid Statements** section).

If **Legal name** is empty, statements are still issued, but with a placeholder identity.

If you are a **Company**, enter your tax ID and its type. Our payment provider verifies the number; if it cannot be verified, VAT is charged even where reverse charge would apply.

### Billing emails

In **Billing emails** you can enter up to three addresses for invoice and payment notifications: a **Primary email** and two **Additional email** fields. When an organization is created, the primary address is the email of the person who created it. These addresses receive notices such as:

- A failed card charge, with a link to pay.
- Funds added to the wallet.
- A change in the organization's billing status (grace period, suspension, reactivation).
- An account tier upgrade, and a new operation that brings your estimated monthly spend to 80% of the limit or more.
- A statement that cannot be issued because the address is missing.

---

## Spending Limit and Account Tiers

Every organization has a **Monthly resource estimate limit**. When you create a new resource or grow an existing one (package, replicas, autoscaling maximum, managed service plan, bandwidth plan and so on), Komuta works out the organization's estimated monthly cost including that change. If the estimate would exceed the limit, the operation is refused with this message:

> This operation would bring your estimated monthly spend to ‹new estimate› USD, above your current spending cap of ‹limit› USD. Your cap needs to be at least ‹difference› USD higher to continue. The cap rises as you add funds to your wallet and verify a card — top up from Account → Billing.

The limit is checked only when you create or grow something. Resources that are already running are not stopped; metered usage such as build minutes can exceed the limit.

The ring on **Account → Wallet** shows the spending recorded this cycle (**Current Cycle Total**) as a share of your limit. Under it you see your tier name and a **Level up to {tier}** link that shows the next tier's limit and conditions.

### Account tiers

Every new organization starts on the **Starter** tier with a USD 20 monthly limit. A tier unlocks automatically as soon as **any one** of its conditions is met:

| Tier | Monthly limit | How to unlock | Concurrent builds (max queued) |
|---|---|---|---|
| **Starter** | USD 20 | Starting tier | 1 (5) |
| **Verified** | USD 50 | Add a verified card | 1 (10) |
| **Basic** | USD 100 | Add USD 10 in total **or** pay USD 10 in statements in total | 2 (20) |
| **Pro** | USD 300 | Add USD 50 in total **or** pay USD 50 in statements in total | 4 (30) |
| **Scale** | USD 1,000 | Add USD 300 in total **or** pay USD 300 in statements in total | 8 (50) |

- The top-up total is the sum of everything you have added; spending the balance does not lower your tier. A refunded top-up is taken off the total.
- Your billing emails get a notice when your tier goes up.
- You cannot set the limit yourself. To go beyond Scale, or to raise the limit sooner, click **Request a limit increase** on **Account → Wallet**. A [support ticket](support-tickets.md) form opens with your current limit filled in; add the limit you need and your planned usage, and send it. Our team reviews the request.

> **Note:** In an organization that is in a grace period or suspended, operations that increase spend are refused whatever the limit. See the **Unpaid Statements** section.

---

## Coupons and Promotions

You can use a coupon or promotion code at sign-up, or later in the **Coupon & promotions** card on **Account → Wallet**.

1. On **Account → Wallet**, click **Redeem a code** (or scroll to the **Coupon & promotions** card).
2. Type the code in the **Coupon or promotion code** field. The code is checked as you type, and what it gives is shown.
3. Click **Apply code**. The message "Code {code} applied" appears and the code is added to **Active offers**.

A code can give one or more of these:

- **Credit** — added to the **Credit** pool of your wallet, spent before your own funds, and usually with an expiry date. Some credits read "added once you confirm your e-mail"; that credit is added when you confirm your email address.
- **Percentage discount** — a percentage off your statements for a set period or until a date; it appears as a separate negative line on the statement.
- **Extra free services** — raises the number of services you can run on the free package.

Rules:

- Each code can be used once per organization.
- An organization can hold at most one partner coupon, and a partner coupon must be applied before the first paid invoice. Promotion codes can be added at any time.
- A code issued for new organizations only, if it was not used at sign-up, can be applied from the **Wallet** page within 30 days of sign-up, only in the organization created at sign-up, and only before the first paid statement.
- Campaigns need no code; they apply to your organization automatically and show in the list as "Applied automatically — no code needed".
- Coupon and campaign discounts do not apply to the **Always on** add-on.

If the code you entered at sign-up could not be applied, the card shows "The code {code} you entered at sign-up was not applied"; if you have another code, you can apply it from the same card.

Common rejection messages:

| Message | Meaning |
|---|---|
| We don't recognise that coupon code. | The code is mistyped or does not exist. |
| Coupon codes are 6–32 characters: letters, digits and dashes. | The code's format is invalid. |
| This coupon has expired. / This coupon is not valid yet. | The code is outside its validity dates. |
| This coupon has reached its redemption limit. | The code's total redemptions are used up. |
| This code has already been used on this organization. | The same code cannot be used twice. |
| This organization already has a partner coupon. Promotion codes can still be added. | The partner coupon slot is taken. |
| A partner coupon can only be applied before your first paid invoice. | The organization has already paid a statement. |

---

## Unpaid Statements

When a statement goes unpaid, the organization first enters a **grace period**, and if it stays unpaid, it is **suspended**.

### Entering the grace period

1. When a statement is issued, your default card is charged. If the charge fails, your billing emails receive a notice with a payment link, and the charge is retried automatically over the next few days.
2. When the retries are over, the statement's status becomes **Uncollectible** and the organization enters the grace period.

A statement whose invoice cannot be issued because your billing address is missing also puts the organization into the grace period 10 days after it was created.

### During the grace period

- Your running services and managed services keep running.
- Operations that increase monthly spend (scaling up, upgrading a package or plan, a new paid resource) are refused with this message: "Your account is in a grace period because an invoice could not be collected. Operations that increase your monthly spend are paused until the outstanding balance is settled. Pay the invoice from Account → Billing to continue."
- The banner **Your account is in a grace period for an unpaid invoice** appears at the top of the console. It shows the number of unpaid invoices, the total and the date the account will be suspended; admins get a **Go to billing** button.

### Suspension

The grace period lasts **14 days**. If the overdue statement is still unpaid at the end of it, the organization is suspended:

- All your services are put to sleep and your PostgreSQL databases are stopped.
- Changes are blocked in the console; you can still view everything. Paying statements, adding cards and editing the billing profile stay open.
- The banner **Your account is suspended for unpaid invoices** appears at the top of the console.

### Getting the account back

1. On **Account → Statements**, open the statement with the **Uncollectible** status.
2. If your card no longer works, first add a new card on **Account → Payment Methods**.
3. Pay by card with **View & pay invoice**, or use **Pay from balance** if you have added funds.

Once no overdue statement is left, the organization returns to normal on its own. Services put to sleep by the suspension are woken and stopped PostgreSQL databases are started again. Services you put to sleep yourself before the suspension, and free services whose free hours are used up, stay asleep.

> **Warning:** Adding funds does not settle a statement whose invoice has already been issued, and it does not lift a suspension. Pay the statement with the steps above.

---

## Cost per Service

To see what a single service costs, open the **Invoice** tab in the service's left menu:

- **Current period charges** — the service's amount in the current cycle, next to **Billing Period** and **Tracked Until**.
- **Current period breakdown** — the line items for the service: **Item**, **Usage**, **Rate**, **Amount**. The service's resource usage and bandwidth plan show here.
- **Statement history** — past statements the service appears on, their status and the service's share of each. Click a row to see the service's line items on that statement.

Managed services, the API Gateway fee and build minutes do not appear on this tab; see **Account → Current Spend** and **Account → Statements** for them.

---

## Related Documents

- [Quick Start](quick-start.md) — Account, organization, free start and a summary of costs.
- [Service Sleep](service-sleep.md) — The free plan, the free hours pool and the **Always on** add-on.
- [Bandwidth Plans](bandwidth-plans.md) — Plans, speed limits and monthly fees.
- [Observability](observability.md) — Telemetry storage plans and prices.
- [Build Queue](build-queue.md) — Concurrent build limits by tier.
- [Limits](limits.md) — Every platform limit in one place.
- [Organizations and Team](organizations-team.md) — The organization admin role and members.
- [Support Tickets](support-tickets.md) — Opening a ticket for a limit increase or billing questions.

# Grace Luxe shop

A phone-first static storefront and admin page built with plain HTML, CSS and JavaScript. It uses Supabase from the browser through the official `supabase-js` CDN. There is no build command or server-side application.

## Files

- `index.html` — customer storefront, product catalogue, bag, checkout and bank-transfer instructions.
- `admin.html` — owner login, product management, photo/video uploads and order management.
- `README.md` — setup and deployment instructions.

Keep all three files in the same folder in your GitHub repository.

## 1. Add your Supabase connection details

Open Supabase → Project Settings → API. Copy the **Project URL** and the **anon public key**.

In `index.html`, find the config block near the beginning of the main script and the project connection is already configured in the committed files:

```js
const SUPABASE_URL="https://eceksawesmuzpdwsonwx.supabase.co";
const SUPABASE_ANON_KEY="sb_publishable_kwAPbdCLCih2dvjZG4VREg_1AomFlD6";
```

In `admin.html`, find its config block and the same project connection is already configured:

```js
const SUPABASE_URL="https://eceksawesmuzpdwsonwx.supabase.co";
const SUPABASE_ANON_KEY="sb_publishable_kwAPbdCLCih2dvjZG4VREg_1AomFlD6";
```

That is the only Supabase configuration needed in the website. The bank details and WhatsApp number are already filled in as requested. Never put a `service_role` key in either file, GitHub, or the browser.

## 2. Supabase checks

You said the database is already set up. Before going live, check these items in your Supabase project:

1. Tables `products` and `orders` exist with the columns described in your setup.
2. The `place_order`, `claim_paid` and `cancel_order` RPC functions exist and have the expected arguments.
3. The public `media` Storage bucket exists and is set to public read.
4. Row-level security is enabled and policies allow public visitors to read active products, authenticated admin users to manage products, and the admin to read/update orders.
5. Create Grace's admin user in Authentication → Users. Use her email and a strong password, and confirm the user if your project requires it.
6. Turn off public sign-ups in Authentication settings after creating the admin user. Do not give the admin password to customers.

The site uses Supabase Auth's browser session persistence, so Grace stays signed in on that device until she logs out or the session expires.

### Important security check

The browser contains the anon public key, which is designed to be public. Actual access control must come from your Supabase RLS policies and Storage policies. Do not weaken RLS to make the site work. Make sure only the intended owner/admin account can sign in, and test the policies using a signed-out browser as well as the admin account.

The database package's example policies grant management access to any authenticated user. That is safe only if public sign-ups are disabled and only the trusted owner account exists. If other authenticated users exist in this Supabase project, tighten the policies so only Grace's admin identity can manage products and orders.

## 3. Deploy to Render as a Static Site

1. Create a GitHub repository, for example `grace-luxe`.
2. Upload `index.html`, `admin.html` and `README.md` to the repository root (not inside an extra subfolder).
3. In Render, choose **New → Static Site** and connect the repository.
4. Use these settings:

   | Setting | Value |
   |---|---|
   | Build command | Leave completely empty |
   | Publish directory | `.` |
   | Branch | Your production branch, usually `main` |

5. Create the Static Site and wait for the deploy to finish.
6. Open the Render URL. The shop is at `/` and the owner page is at `/admin.html`.

When you update either HTML file in GitHub, Render will redeploy the static site automatically if auto-deploy is enabled.

## 4. How Grace uses the admin page

1. Open `https://YOUR-RENDER-SITE.onrender.com/admin.html`.
2. Sign in with the Supabase admin email and password.
3. Under **Products**, add a product or edit an existing one. Enter price in whole naira, stock, MOQ, sizes and whether it should be visible or featured.
4. Select multiple photos/videos from the phone gallery. Photos are resized in the browser to a maximum width of 1400px and JPEG quality around 80%. Videos larger than 50 MB are rejected. Uploads are saved in the `media` bucket.
5. Use the left/right arrows under each media item to change its order. The first item is the product's main media.
6. Under **Orders**, check new orders. The list refreshes every 20 seconds. The badge shows how many customers have tapped “I have paid”.
7. Check the transfer in the bank app before tapping **Confirm payment**. If an order will not be fulfilled, tap **Cancel order** and confirm; the database RPC restores the reserved stock.
8. Tap **Log out** when finished on a shared device.

## 5. Customer order flow

1. Customers browse products, choose a size and quantity, and add them to the bag. Quantity starts at the product MOQ and cannot be set below MOQ or above available stock.
2. Checkout collects name, phone, state and delivery address. The database `place_order` RPC validates MOQ and stock and reserves the stock atomically.
3. The customer sees the order code, exact total and the OPay account details.
4. After making the transfer, the customer taps **I have paid, send order on WhatsApp**. The page calls `claim_paid` and provides a WhatsApp message with the order details and receipt reminder.
5. Grace checks her bank app and marks the order Paid in the admin page.

**Bank transfers are not confirmed automatically.** Grace must check the bank app herself before confirming payment. An order that is never paid continues to reserve its stock until Grace cancels it. The current database tracks stock per product, not per size.

## 6. Test before sharing with customers

Use a test product and a second browser/device:

- Add a product with stock 3 and MOQ 1.
- Place an order for 2 and confirm stock falls to 1.
- Try to order 2 more; it should be refused and the bag should remain.
- Check that a customer can choose a size, see the correct MOQ and cannot order more than stock.
- Place an order, make the “I have paid” action, and verify WhatsApp opens with the order code, items, sizes, quantities, total and delivery details.
- In admin, confirm a test payment and verify its status becomes Paid.
- Place another test order and cancel it; verify the reserved stock is restored.
- Try setting MOQ to 2 and ordering 1; the database should refuse it.
- Upload several photos and a short video from a phone. Check that the first media item appears first in the storefront.
- Try an incorrect admin password and confirm login is refused.
- Open the shop in a signed-out/private browser and confirm that only active products are visible and admin data is not accessible.
- Test on a phone using a slow connection.

## 7. Notes

- The storefront uses the supplied Grace Luxe design: dark/gold styling, Instrument Serif and Instrument Sans, hero carousel, category chips, product sheet, bag and checkout. Product records and media come from Supabase; no demo products are hard-coded.
- Product photos/videos are served from the public `media` bucket. Removing media from the product editor removes it from the product's media list; it does not automatically delete the old file from Storage. You can remove unused files from Supabase Storage when you are sure they are no longer needed.
- Stock is one number per product, not separate stock counts per size.
- The current RPC design reserves stock as soon as the order is placed. Grace should regularly review unpaid orders and cancel orders that will not be paid.
- A customer can submit an order claim, but only Grace can verify the bank transfer. Do not describe the payment as automatically verified.
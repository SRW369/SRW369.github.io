# Saffron & Slate

A responsive HTML, CSS and JavaScript food ordering storefront. Open `index.html` directly in a modern browser or serve this folder with any static file server. All project files live at the project root.

## Working in this standalone build

- Home and Order navigation, responsive menu, categories, search, price/vegetarian/offer filters, sorting, and lazy-loaded food photography.
- Product detail customizations (size, add-ons, special instructions and quantity).
- Persistent browser cart, item quantity controls, delivery calculation, tax estimate, and the `FIRSTBITE` discount code.
- Validated checkout form and an order request saved in this browser's local storage.

**Checkout does not take payment or send an order to a restaurant.** Without a backend, the checkout deliberately records an unpaid request on the current device and clearly says it is not confirmed. Cart and order request data are stored in this browser and are not a production database.

The Bag now reads confirmed order records from `users/{uid}/orders/{orderId}` and any existing `orderHistory` array on the signed-in user's profile. It supports reorder from those records. The current local checkout creates only unpaid requests and has no verified payment/order confirmation event, so it cannot safely write a confirmed order to Firestore. Existing `ss_order_requests` entries are unpaid, browser-local requests without an authenticated UID; they are intentionally not promoted to confirmed order history.

For production order persistence, a trusted backend must write the order under the authenticated UID only after verifying payment/order confirmation. The browser rules in `firestore.rules` allow customers to read their own history and prohibit client order writes. Merge these rules into the existing Firebase rules instead of replacing rules for the site's other collections.

## Production integration still required

The credentials, Firebase project, and payment server were not provided. A frontend-only static site cannot securely create or verify Razorpay payments, protect admin access, or securely persist customer accounts and orders. Connect the following before accepting real customer data or orders:

The login window now has a Firebase Google popup option. Add your Firebase Web app config to `firebase-config.js`, enable Google under Authentication > Sign-in method, create Firestore, and add your localhost or production host under Authentication > Authorized domains. The Firebase SDK is loaded only when the Google button is pressed so the rest of the storefront continues to work before setup. Google login creates `users/{uid}` only when the profile document does not already exist.

1. **Firebase Authentication:** create a Firebase project and enable email/password and Google providers. Use Firebase's web SDK in `app.js` (or a dedicated root-level `firebase-config.js` if desired), with the public web app config. Public Firebase web config is not a server secret; lock down Firestore and Storage with deployed security rules. Add verified admin claims server-side only.
2. **Firestore and Storage:** deploy collections for users, products, categories, addresses, orders, payments, and coupons. Restrict private records to their owner; reserve menu writes and order-status changes for server-authorized staff. Validate product prices, coupon eligibility, inventory, and totals in trusted server code.
3. **Razorpay:** implement an HTTPS backend (for example, Firebase Cloud Functions) that authenticates the user, recalculates the basket from trusted menu data, creates a Razorpay order using a server-held `RAZORPAY_KEY_SECRET`, and returns only the public key and order ID. Load Razorpay Checkout from the official SDK, then verify the payment signature on the backend. Confirm order state from signature-verified callbacks and a signature-verified webhook. Make payment/order handling idempotent and retain only the payment references and statuses needed for operations.
4. **Secrets and launch:** keep `RAZORPAY_KEY_SECRET` and webhook secrets in server environment/secrets management, never in browser JavaScript or Firestore. Configure Firebase rules, HTTPS hosting, privacy/terms, delivery coverage, taxes, and real contact details. Test payment success, failure, cancellation, retries, webhook replay, and duplicate submissions in Razorpay test mode before launch.

Images and web fonts load from Unsplash and Google Fonts; an internet connection is needed for those assets. Replace those remote image URLs with licensed, optimized production assets before launch.

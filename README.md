<div align="center">

# Bookaroo 📚

### Better books. Better prices.

**A student-to-student textbook marketplace built for the Texas Tech community.**

[**View the Website**](https://sparky524.github.io/Bookaroo/) · [**Report a Bug**](https://github.com/Sparky524/Bookaroo/issues) · [**Suggest a Feature**](https://github.com/Sparky524/Bookaroo/issues)

</div>

---

## About Bookaroo

Bookaroo is a student-built marketplace designed to make buying and selling college textbooks easier and more affordable. Instead of paying bookstore prices or dealing with anonymous listings, students can discover books for their courses, view seller profiles, ask questions, and arrange an exchange with another student.

The project currently focuses on **Texas Tech University** and uses verified `@ttu.edu` email accounts for access to student features.

> **Independent project:** Bookaroo is not an official Texas Tech University product and is not affiliated with or endorsed by the university.

## Live Demo

**Website:** https://sparky524.github.io/Bookaroo/

Some functionality requires a verified Texas Tech email address. Payments are under development/testing; do not assume production payment readiness.

## Features

### Browse and discover textbooks

- Search listings by title, ISBN, course subject, or course number.
- Filter by course, condition, availability, and maximum price.
- Sort by price or newest listings.
- See textbook details, condition, asking price, seller, and cover image.
- Look up book metadata using Google Books and Open Library.
- Scan an ISBN barcode with a supported device camera.
- Save favorite textbooks and follow courses for matching listings.
- Share a direct link to an individual textbook listing.
- Distinguish example/demo listings from actual student listings.

### Student accounts and seller profiles

- Sign up and sign in with Supabase Authentication.
- Verified `@ttu.edu` email access for student marketplace actions.
- Customize a profile with a display name, avatar, major, graduation year, and bio.
- Crop and reposition profile photos before saving.
- View seller profiles, verified reviews, star ratings, and sales counts.
- Manage personal listings, saved books, purchases, sales, and profile settings.

### Messaging and notifications

- Message a seller with questions about a textbook.
- Read and send messages from the My Account inbox.
- Use Supabase Realtime for message updates, with periodic refresh as a fallback.
- Receive in-app notifications for supported marketplace activity.

### Checkout and order management

- Stripe Checkout integration for supported listings.
- Stripe Connect onboarding for sellers.
- Purchase and sales history, order confirmations, and receipt confirmation.
- Order progress/status display to help buyers and sellers coordinate handoff.
- Verified buyer reviews after eligible completed transactions.

**Payment caution:** These features depend on separately deployed Supabase Edge Functions, database policies, and Stripe configuration. The checkout, webhook verification, payout routing, and order-state security must be independently audited and tested before accepting real payments.

### Administration and moderation

- Admin-only navigation and dashboard.
- Review student reports and take moderation actions.
- Edit or remove eligible listings.
- Search listings for quality checks.
- View marketplace statistics and analytics.
- Owner-controlled moderator access management (requires the corresponding database setup).

### Marketplace experience

- Responsive dark-themed interface with burgundy accents.
- Compact listing cards, seller ratings, and share icons.
- About Us page with textbook exchange and campus meetup guidance.
- Account dashboard organized into separate sections.

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | HTML, CSS, vanilla JavaScript |
| Hosting | GitHub Pages |
| Authentication | Supabase Auth |
| Database | Supabase PostgreSQL |
| Backend APIs | Supabase Edge Functions |
| Live updates | Supabase Realtime |
| Payments | Stripe Checkout and Stripe Connect |
| Book metadata | Google Books and Open Library |
| Barcode scanning | `html5-qrcode` |

The current frontend is primarily contained in a **single `index.html` file**.

## Getting Started

### Prerequisites

To run the full project, you will need:

- A modern web browser.
- A Supabase project with the required database schema, authentication, storage, and row-level security policies.
- Deployed Supabase Edge Functions for Stripe-related operations.
- A Stripe account configured for **test mode**.
- A static host such as GitHub Pages (or a local development server).

### 1. Clone the repository

```bash
git clone https://github.com/Sparky524/Bookaroo.git
cd Bookaroo
```

### 2. Configure Supabase

The frontend initializes a Supabase client using a project URL and **publishable key**. If you deploy your own copy, update those values in `index.html` to match your Supabase project.

A Supabase publishable key is intended for browser use. **Never place a Supabase service-role key, Stripe secret key, or Stripe webhook signing secret in frontend code.**

The database must also have the tables, policies, functions, and storage configuration expected by the frontend. Depending on the repository version, these may include:

- Textbook listings, profiles, reviews, favorites, watches, reports, and orders.
- Conversations and messages.
- In-app notifications.
- Admin roles and moderation functions.
- Order receipt and seller review functions.

> **Database setup note:** The frontend file alone is not a complete backend installation. If SQL migrations or Edge Function source files are not present in this repository, they must be supplied before a fresh deployment can work. Do not run historical migrations blindly against an existing production database; review schema compatibility and back up your data first.

### 3. Configure authentication

In the Supabase dashboard:

1. Enable email authentication.
2. Configure the application URL and allowed redirect URLs for your deployed site.
3. Enable email confirmation as appropriate.
4. Review server-side enforcement of verified Texas Tech email addresses.

A client-side `@ttu.edu` check by itself is **not** an adequate security boundary. Database policies and backend functions must enforce authorization.

### 4. Configure Stripe (test mode)

The frontend calls Supabase Edge Functions named `create-checkout` and `stripe-connect`. These functions must be deployed separately and configured with the required Stripe credentials and permitted site origin.

Stripe secret keys and webhook signing secrets belong in **server-side environment variables**, never in GitHub Pages assets.

Before enabling live payments, verify:

- Checkout amounts are calculated server-side from trusted database records.
- The seller and Stripe Connect account are validated server-side.
- Orders are marked paid only after verified Stripe webhook events.
- Webhook processing is idempotent and safe against retries.
- Concurrent checkout attempts cannot result in duplicate sales.
- Database row-level security prevents unauthorized order changes.
- Refunds, failed payments, disputes, and payout states are handled correctly.

### 5. Run locally

Because Bookaroo is a static frontend, you can serve it with a simple HTTP server:

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000` in your browser.

Some features require HTTPS, authorized origins, and the configured Supabase backend. Camera access typically requires HTTPS or a trusted localhost context.

### 6. Deploy with GitHub Pages

1. Push the updated `index.html` to your repository.
2. Open **Settings → Pages** in GitHub.
3. Select **Deploy from a branch**, then choose the publishing branch and root folder.
4. Save and wait for GitHub Pages to publish the site.
5. Open https://sparky524.github.io/Bookaroo/.

The frontend is deployed through GitHub Pages, but database changes and Supabase Edge Functions must be deployed separately.

## How Bookaroo Works

### Buying a textbook

1. Create an account and verify your Texas Tech email.
2. Browse listings or search by course or ISBN.
3. Check the edition, condition, price, and seller profile.
4. Message the seller if you have questions.
5. Complete checkout when the listing is eligible and the payment system is properly configured.
6. Coordinate a safe handoff and confirm receipt when appropriate.
7. Leave a verified review when eligible.

### Selling a textbook

1. Sign in with a verified account.
2. Set up your seller profile and connect Stripe if required.
3. Create a listing with its title, course, condition, ISBN, photo, and price.
4. Respond to buyer messages.
5. Follow the order and handoff process after a verified purchase.

## Security and Privacy

Bookaroo handles account information, messages, and potentially payment-related records. Security must be enforced **server-side**, not only through hidden buttons or browser JavaScript.

Recommended safeguards include:

- Supabase Row Level Security on all private and mutable tables.
- Participant-only access to private conversations and messages.
- Owner-only control of moderator permissions.
- Server-validated listing changes and payment amounts.
- Stripe webhook signature verification and idempotency.
- Rate limiting and abuse protection for messaging and reports.
- Minimal personal data collection and appropriate data retention.
- Regular dependency, configuration, and permission reviews.

**Never commit credentials or private student data.** Use test accounts and Stripe test cards for development.

## Project Status

**Active student project / prototype.** The interface includes a broad set of marketplace features, but a successful frontend build does not guarantee that every backend function is deployed or that the payment system is ready for real transactions.

Known areas to validate before a public production launch:

- End-to-end payment and payout security.
- Database schema and migration consistency.
- Authorization and admin-role enforcement.
- Messaging privacy and notification delivery.
- Mobile usability, accessibility, and error handling.
- Moderation workflows and user-support processes.

## Roadmap

Possible future improvements:

- Multi-photo galleries for textbook condition verification.
- More robust message delivery and unread indicators.
- Email notifications for important activity.
- Improved reporting, moderation audit logs, and abuse prevention.
- More detailed order and handoff tracking.
- Automated testing for authentication, permissions, and checkout.
- Expanded support for additional university communities.

## Contributing

Ideas, bug reports, and feedback are welcome. Open an [issue](https://github.com/Sparky524/Bookaroo/issues) describing the problem or proposed improvement. If contributing code, explain the change and how you tested it.

Please do not include real student information, private credentials, or payment secrets in issues or pull requests.

## Disclaimer

Bookaroo is an independent, student-built marketplace project. It is **not affiliated with, sponsored by, or endorsed by Texas Tech University**. Users should independently verify textbook editions and course requirements and use appropriate care when arranging in-person exchanges.

## Creator

**Zachary Heller**  
Texas Tech University

---

<div align="center">

**Bookaroo — made for Red Raiders.**

[Explore Bookaroo](https://sparky524.github.io/Bookaroo/)

</div>

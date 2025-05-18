### 🎙️ **Video Script – Soundblock Project (Laravel + Angular)**

---

**Hi there!**
I'm Lautaro, a senior full-stack developer with around 7 years of hands-on experience building scalable web platforms. Today, I’d like to walk you through one of the key projects I worked on — **Soundblock**, a platform developed using **Laravel (PHP)** on the backend and **Angular** for the frontend.

---

### 🧩 **Project Overview**

Soundblock is a **music asset management platform** tailored for creators and producers. It allows users to upload, organize, and monetize music-related content in a secure and automated environment.

The system handles **user authentication**, **role-based access**, **payment integration**, and **background processing** for handling heavy file operations like zipping and unzipping music files.

---

### 🗂️ **Core Functionalities**

Here are a few key modules and how they’re structured:

- **Apparel Module**:
  Allows users to list and manage merchandise. It’s tightly integrated with product inventory and custom order tracking.

- **Arena Office Dashboard**:
  Built for artists and managers to track revenue, payouts, and royalties. We used Laravel's Eloquent ORM to design normalized MySQL schemas and efficient data access.

- **Authentication System**:
  We implemented Laravel Passport for API authentication, allowing secure token-based access for different roles — like artists, collaborators, and admins.

- **Payment Integration**:
  Stripe was used to handle both one-time purchases and recurring subscriptions. I integrated webhook listeners to handle real-time payment updates and errors.

---

### ⚙️ **Background Tasks & Queue Workers**

One of the heavier features in Soundblock was the **file processing workflow**:

- When a user uploads a project, it’s stored as a **.zip archive**.
- A Laravel queue job is dispatched using **Supervisor and Redis** to extract the zip in the background — ensuring no delay for the user.
- After extraction, we trigger a notification system (via email or in-app alerts) to inform the user that their content is ready.
- This setup helps with **asynchronous processing** and keeps the frontend responsive, especially for large files.

---

### 🗃️ **Schema Design**

Our MySQL schema was carefully normalized and optimized. I created ERDs for each major domain — including Users, Projects, Assets, and Financial Transactions. We used foreign keys and indexed pivot tables to optimize relationship lookups, especially for reporting and dashboard modules.

---

### 🛠️ **Frontend with Angular**

On the frontend, we used **Angular 8 with NgRx** to manage state and **Material Design** for the UI layer. I designed reusable components for project uploads, dashboards, and notifications.

---

### 💡 **Challenges Faced**

- One challenge was handling **large zip file processing** without blocking the user. We solved this with proper queueing and notification feedback loops.
- Another was integrating the **Stripe payment system** for both platform fees and artist payouts. We had to ensure secure, idempotent API calls and solid error handling.

---

### ✅ **Outcome**

The platform now supports thousands of users and handles large file uploads with confidence. I’m proud of how we made it scalable, secure, and easy to use.

---

### 🎙️ **Narration Script for `CategoriesController.php` (Laravel + DI + Services)**

---

Hi again — now I’ll walk you through a real-world controller from the Soundblock project. This is the `CategoriesController` located in the Apparel domain. It's part of a modular Laravel application and follows clean architecture principles like **Dependency Injection**, **service-oriented design**, and **separation of concerns**.

---

### 📦 Controller Overview

This controller is responsible for handling **Apparel Categories**, including:

- Listing categories
- Fetching products under a category
- Fetching related attributes (like color, style, fit)
- Providing a bootstrap response with product and slider data for the frontend

We follow a **thin controller** approach — the controller acts as a bridge between the request and dedicated **service classes**, which handle the business logic.

---

### 🧱 Dependency Injection

In the constructor, we inject the `Category` and `Attribute` models directly:

```php
public function __construct(Category $category, Attribute $attribute)
```

These are used primarily for simple lookups or fallbacks.

In more complex methods like `getProducts`, we inject service classes like `ProductService`, `AttributeService`, or `CategoryService` **as method parameters**, which Laravel resolves automatically thanks to its built-in **IoC container**.

---

### 🔍 Fetching Products for a Category

In the `getProducts` method:

```php
$category = $categoryService->find($categoryUuid, true);
$filters = $request->only(["color", "style", "weight", "fit"]);
$products = $productService->getProductsByCategory($category, $filters, $request->get("per_page", 20));
```

We:

1. Use `CategoryService` to find the category by UUID.
2. Extract relevant filter parameters from the request.
3. Delegate business logic to `ProductService`, which handles the filtering and pagination.

The result is returned through a standardized API resource (`BaseCollection`), which formats the response consistently across endpoints.

---

### 📦 Pagination Logic

In the `__getProducts` method, we manually paginate the product list using Laravel’s `LengthAwarePaginator`:

```php
new LengthAwarePaginator(
  $products->forPage(...),
  $products->count(),
  ...
)
```

This was needed because `getProductsByCategory` returns a `Collection`, not a paginated query — likely due to additional in-memory filtering.

---

### 🎯 Bootstrap Method

The `bootstrap()` method prepares default data for the frontend, including featured products and **slider images** stored on Amazon S3.

```php
$sliderImages = [
  $apparelAdapter->url("slider/1.webp"),
  $apparelAdapter->url("slider/2.webp"),
];
```

We use a custom S3 disk (`s3-apparel`) and expose public URLs for use in the frontend carousel. This shows how we integrate cloud storage directly into the app.

---

### ⚠️ Error Handling

Each method is wrapped in a try-catch block:

```php
try {
  // logic
} catch (\Exception $e) {
  throw $e;
}
```

In production, we usually log the error or return a custom API error response. But for now, this simple rethrow is enough to indicate that proper error boundaries are in place.

---

### ✅ Summary

This controller shows a clean and scalable design using:

- **Constructor & method-level Dependency Injection**
- **Service classes for business logic separation**
- **Pagination and filtering**
- **S3 integration**
- **Typed responses and API resources**

It’s structured to be testable, easy to maintain, and fits well into a larger domain-driven Laravel app.

Absolutely! Here's a **clean, professional, senior-level narration script** you can use to explain the `PaymentsController` for your video presentation. This version focuses on showcasing your experience with **Laravel best practices**, **Stripe integration**, **contracts (interfaces)**, **error handling**, and **secure payment flow** — all of which reflect your capability as a senior Laravel developer.

---

### 🎙️ **Narration Script – `PaymentsController` (Laravel + Stripe Integration)**

---

Let me now walk you through a core component of the **Soundblock platform** — the `PaymentsController`, which manages Stripe-based payment operations.

This controller demonstrates my experience in building **secure, scalable, and testable** payment workflows in Laravel using a **contract-driven architecture** and **third-party integrations**, in this case, Stripe.

---

### 🔐 Key Design Highlights

- We use **Laravel Contracts** (`PaymentContract`, `FinanceContract`) to **decouple business logic** from the controller.
- All Stripe logic is encapsulated within services, which follow the **Dependency Inversion Principle** — making them easy to mock and test.
- We follow a **defensive programming style** by catching and handling multiple exception types, providing robust API error responses.
- The controller methods are well-documented using Laravel-style annotations for API documentation (e.g., `@group`, `@responseFile`, `@bodyParam`).

---

### 💳 `addPaymentMethod()`

```php
public function addPaymentMethod(AddPaymentMethodRequest $request, PaymentContract $payment, FinanceContract $financeContract)
```

This method adds a Stripe payment method for the authenticated user and **immediately charges** them using our internal `FinanceContract`.

Key steps:

1. Get or create a Stripe customer for the user.
2. Add a new payment method via Stripe.
3. Charge the user immediately to validate the method (e.g., \$1 authorization or first payment).
4. Catch and handle any exceptions: `CardException`, `InvalidRequestException`, or a custom `PaymentTaskException`.

We use Laravel’s built-in `Auth` facade to retrieve the authenticated user and a custom FormRequest (`AddPaymentMethodRequest`) for validation.

---

### 🧾 `getPaymentMethods()`

This method retrieves all saved Stripe payment methods for the user:

```php
$user->asStripeCustomer();
return response()->json($payment->getUserPaymentMethods($user));
```

We ensure the user is a Stripe customer before fetching their methods, otherwise, we abort with a clear error message. This helps **maintain API consistency** and guides the frontend in handling empty or invalid states gracefully.

---

### ❌ `deletePaymentMethod()`

This method handles two flows:

- If no method ID is passed, **all payment methods are deleted**.
- If a specific method ID is passed, only that one is removed.

We validate ownership and existence of the payment method using `findPaymentMethod` on the user model. This protects against ID spoofing and ensures **secure access control**.

---

### ⭐ `updateDefaultPayment()`

This method updates the **default payment method** for the user:

```php
$payment->updateDefaultMethod($user, $methodId);
```

We again validate ownership of the method, retrieve the customer on Stripe, and then update the default method. This keeps billing consistent across subscriptions or one-time charges.

---

### 🧠 Error Handling Strategy

All methods implement **structured exception handling**, catching:

- Stripe-specific exceptions (`CardException`, `InvalidRequestException`)
- Custom app-level exceptions like `PaymentTaskException`
- Fallback to a generic exception block with a clean API response

In the case of critical failures, we use a custom `Disaster::handleDisaster()` to log and alert developers or external monitoring tools.

---

### ✅ Summary

This controller represents a **modular, secure payment handling system** using:

- Contract-driven design
- Laravel's Auth, Request Validation, and Facade features
- Stripe SDK integration
- Clear separation of controller and service logic

It’s built to be **scalable**, **testable**, and **safe**, and has been used in a production environment to manage real user payments with confidence.

Here’s a **natural and technically clear narration script** to help you explain the **ERD schema** based on the tables you've shared — particularly focusing on `users` and `users_financial_stripe_subscriptions`.

---

### 🎙️ **Narration Script: Explaining the ERD (Laravel + Stripe Subscription Model)**

> “Let me now walk you through a part of the database design behind Soundblock — specifically around **user accounts and Stripe subscriptions**.
>
> At the center of this schema, we have the **`users` table**, which stores core user identity information. It includes fields like:
>
> - A primary key `user_id`,
> - A globally unique identifier `user_uuid`,
> - Standard name fields like `name_first`, `name_middle`, and `name_last`,
> - As well as secure password storage and token-based authentication.
>
> Now, each user can have one or more subscriptions through Stripe. These are handled in the **`users_financial_stripe_subscriptions`** table.
>
> This table links back to the users table using both the `user_id` and `user_uuid` for redundancy and security — which allows us to join and filter records either by auto-increment IDs or UUIDs in a secure, API-friendly way.
>
> For each subscription, we store:
>
> - The `stripe_id`, which maps directly to Stripe’s internal reference
> - The `stripe_status` such as `active`, `trialing`, or `canceled`
> - The `stripe_plan` name or tier
> - The number of licenses or seats in `quantity`
> - And timestamps for trial expiration (`trial_ends_at`) and subscription end (`ends_at`)
>
> Additionally, we use custom timestamp fields like `stamp_created`, `stamp_updated`, and `stamp_deleted`, alongside Laravel’s native timestamps. These allow for **audit logging** and tracking which system user performed each change — helpful when integrating with **ledger systems like AWS QLDB** for full traceability.
>
> All major fields are indexed to ensure performance during reporting, dashboard views, and filtering subscription data by status or date.
>
> This structure makes it easy to join users with their financial activity, and supports scalable subscription logic for future integrations — like webhook handling, invoicing, or analytics.
>
> Overall, this ERD gives us a flexible and secure foundation for managing Stripe-based billing in the Soundblock ecosystem.”

---

### 🎙️ **Narration Script: Explaining the ERD — Apparel + Stripe Subscriptions**

> “Let me now walk you through two distinct parts of our schema design — one related to **Stripe subscriptions for users**, and another focused on **attribute-category relationships** for our apparel system.
>
> First up, we have the **`users_financial_stripe_subscriptions`** table. This table captures Stripe-based subscriptions linked to each user.
>
> It stores both a numeric `subscription_id` and a UUID `subscription_uuid` to support both internal DB references and external API communication.
>
> We reference the user in two ways — by `user_id` and `user_uuid` — and keep detailed subscription metadata like the Stripe ID, plan name, status, quantity, and trial/expiration timestamps.
>
> This table also includes **audit timestamps**, like `stamp_created`, `stamp_updated`, and `stamp_deleted`, along with the corresponding `created_by`, `updated_by`, and `deleted_by` fields. These are useful for tracking system-level activity, especially in an environment like AWS QLDB where historical integrity matters.
>
> Now let me switch gears to the more complex **`apparel_categories_attributes`** table.
>
> This is a **many-to-many relationship table** that links **apparel categories** to **custom attributes** — for example, connecting a "T-Shirt" category with attributes like "Size", "Color", or "Material".
>
> Each record has a unique `row_id` and `row_uuid`, and it links to both `attribute_id` / `attribute_uuid` and `category_id` / `category_uuid`.
>
> What's especially important here is that we’ve defined **multiple composite unique constraints**. These prevent duplicate relationships between attributes and categories in any order or combination — making sure our data model is clean and consistent.
>
> This is critical in a system where we generate dynamic product variations and need full control over which attributes apply to which categories.
>
> Just like with subscriptions, we include full timestamp tracking and user stamps for creation, updates, and deletions.
>
> Overall, this schema shows how we designed Soundblock’s backend for **data consistency, traceability, and flexibility**, while keeping it optimized for real-world scale and integrity.
>
> Thanks for watching!”

---

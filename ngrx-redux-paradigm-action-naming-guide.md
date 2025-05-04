```ts

## ✅ Naming Actions in NgRx – Step-by-Step (Senior-Level)

### **1. Follow the `[Source] Event` Pattern**

✅ You're already doing this with:

[Campaigns Page] Set Campaign

```

#### ✅ Why it’s good:

- `[Source]` indicates where the action originates (component, effect, API, etc.)
- `Event` describes **what happened**, not what you want to happen
- Super helpful in debugging, especially when looking at logs or NgRx DevTools

---

### **2. Define Clear Sources for Your Actions**

Some common and helpful sources:

| Source           | When to Use                                         |
| ---------------- | --------------------------------------------------- |
| `[Feature Page]` | UI-triggered actions (e.g., user clicks, page init) |
| `[API]`          | Effects or backend-triggered actions                |
| `[Service]`      | Programmatic app-wide triggers                      |
| `[Effect]`       | (Optional) For internal orchestration clarity       |

#### ✅ Example:

```ts
// From UI
export const setCampaign = createAction("[Campaigns Page] Set Campaign");

// API response
export const setCampaignSuccess = createAction(
  "[Campaigns API] Set Campaign Success"
);
export const setCampaignFailure = createAction(
  "[Campaigns API] Set Campaign Failure"
);
```

---

### **3. Use Precise, Past-Tense Verbs (for Events)**

This helps clarify what has occurred.

| Action          | Example                        |
| --------------- | ------------------------------ |
| Start / Trigger | `[Page] Load Campaigns`        |
| Completed       | `[API] Load Campaigns Success` |
| Failed          | `[API] Load Campaigns Failure` |
| Sync            | `[Page] Set Campaign`          |

So in your example, `"Set"` is fine for sync/update-type operations from the UI.

---

### **4. Use `props<{ ... }>` with Strong Types**

Keep actions strongly typed for better tooling and maintainability.

```ts
export const setCampaign = createAction(
  "[Campaigns Page] Set Campaign",
  props<{ campaign: Campaign }>()
);
```

---

### **5. Keep Actions Lean and Purpose-Driven**

Each action should represent a **single, atomic event**.

🚫 Avoid:

```ts
[Campaigns Page] Load And Select Campaign
```

✅ Prefer:

```ts
[Campaigns Page] Load Campaigns
[Campaigns Page] Select Campaign
```

---

### **6. Be Consistent with Async Patterns**

For async operations (like API calls), use a 3-action pattern:

```ts
export const loadCampaigns = createAction("[Campaigns Page] Load Campaigns");
export const loadCampaignsSuccess = createAction(
  "[Campaigns API] Load Campaigns Success",
  props<{ campaigns: Campaign[] }>()
);
export const loadCampaignsFailure = createAction(
  "[Campaigns API] Load Campaigns Failure",
  props<{ error: any }>()
);
```

---

### **7. Bonus: Prefix Internal/Effect-Only Actions (Optional)**

If some actions are purely internal and should not be triggered by components, you can prefix them:

```ts
[Campaigns Effect] Dispatch Campaign Analytics
```

Or:

```ts
[Campaigns Internal] Do Something
```

Helps with debugging and architecture.

---

## 🧠 Summary: You're Doing Great!

| Rule                          | You're Following It?                          |
| ----------------------------- | --------------------------------------------- |
| ✅ `[Source] Event` format    | ✔️ `[Campaigns Page] Set Campaign` is perfect |
| ✅ Verb clarity               | ✔️ "Set" is appropriate for local state sync  |
| ✅ File per feature           | (Assumed) ✔️                                  |
| ✅ Strong typing with `props` | ✔️                                            |
| ✅ Async patterns when needed | ✔️ if using them                              |

---

### 🚀 Want Me to Help Further?

I can:

- Review or generate all actions for a feature (e.g., `Campaigns`, `Users`)
- Create reusable templates or generators for consistent actions
- Help you build a scalable action architecture across a large app

Just say the word.

Awesome — setting up a **scalable action architecture** in a large Angular + NgRx app is crucial for maintainability, collaboration, and debugging. Here's a full blueprint that scales with team size and app complexity.

---

## 🚧 The Problem with Unstructured Actions

In large apps:

- Actions get messy across features
- Naming becomes inconsistent
- It's hard to debug and search
- Duplication or cross-feature coupling sneaks in

---

## ✅ Scalable Action Architecture Blueprint

### **1. Follow a Consistent Folder Structure**

Organize by **feature**, not type:

```
src/
└── app/
    ├── state/
    │   ├── app.actions.ts
    │   ├── app.reducer.ts
    │   └── app.selectors.ts
    ├── campaigns/
    │   ├── +state/
    │   │   ├── campaigns.actions.ts
    │   │   ├── campaigns.reducer.ts
    │   │   ├── campaigns.effects.ts
    │   │   └── campaigns.selectors.ts
    │   ├── campaigns.component.ts
    │   └── campaigns.module.ts
```

> 🧠 Each feature has its own `+state` folder with clearly named action files.

---

### **2. Stick to Action Naming Convention**

**Pattern:**

```
[Source] Event
```

**Sources:**

- `[Feature Page]` – UI-driven
- `[Feature API]` – Effect/HTTP-driven
- `[Feature Guard]`, `[Feature Resolver]`, `[Feature Effect]`, `[Feature Internal]` – optional granularity

**Events:**

- Use clear verbs: `Load`, `Set`, `Select`, `Reset`, `Update`, `Add`, `Delete`, etc.
- For async: `Action`, `Action Success`, `Action Failure`

**Example for Campaigns:**

```ts
// campaigns.actions.ts

export const loadCampaigns = createAction("[Campaigns Page] Load Campaigns");
export const loadCampaignsSuccess = createAction(
  "[Campaigns API] Load Campaigns Success",
  props<{ campaigns: Campaign[] }>()
);
export const loadCampaignsFailure = createAction(
  "[Campaigns API] Load Campaigns Failure",
  props<{ error: any }>()
);

export const selectCampaign = createAction(
  "[Campaigns Page] Select Campaign",
  props<{ id: string }>()
);
```

---

### **3. Group and Document Actions by Concern**

In each `*.actions.ts`, group related actions and use comments:

```ts
// === Load Campaigns ===
export const loadCampaigns = ...
export const loadCampaignsSuccess = ...
export const loadCampaignsFailure = ...

// === Selection ===
export const selectCampaign = ...
export const deselectCampaign = ...
```

---

### **4. Avoid Action Duplication**

Use shared actions when:

- Multiple features rely on the same API
- App-wide events like `[App] Init` or `[Router] Navigated`

```ts
// app.actions.ts
export const appInit = createAction("[App] Init");
```

---

### **5. Centralize Global Events (Optional)**

For shared or cross-cutting concerns like:

- Auth
- Layout (sidebar toggle)
- Router
- Notifications

Keep those in `src/app/state/` or `src/shared/state/`.

---

### **6. Use Action Creators + Union Types (Optional)**

Use an enum or object to map all action types for tooling or tests.

```ts
export const CampaignsActions = {
  loadCampaigns,
  loadCampaignsSuccess,
  loadCampaignsFailure,
  selectCampaign,
};

export type CampaignsActionsUnion =
  | ReturnType<typeof loadCampaigns>
  | ReturnType<typeof loadCampaignsSuccess>
  | ReturnType<typeof loadCampaignsFailure>
  | ReturnType<typeof selectCampaign>;
```

---

### **7. Leverage NgRx Feature Creator Helpers**

For large apps, `createFeature()` (from NgRx v15+) helps modularize reducers, selectors, and actions under a single source of truth.

---

## 🧪 Pro Tips

| Tip                                 | Why It Helps                        |
| ----------------------------------- | ----------------------------------- |
| ✅ Prefix feature name in file name | Prevents import collisions          |
| ✅ Use `+state` folders             | Angular’s new structured convention |
| ✅ Document actions                 | Makes onboarding easier             |
| ✅ Use barrels (`index.ts`)         | Clean imports                       |
| ✅ Audit actions regularly          | Remove unused actions               |

---

## 🔧 Want a Starter Template?

I can generate a full **starter structure** with:

- Feature actions
- Effects
- Reducers
- Selectors
- Sample data flow

Just tell me the feature (e.g., `Users`, `Campaigns`, `Orders`) and if you're using **NgRx signals**, `createFeature`, or classic API.

Would you like a ready-to-use scaffold for one of your features?

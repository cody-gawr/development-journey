Great follow-up! The answer is nuanced — it **does work sometimes**, but not always reliably, and here's why.

---

## 🧠 Why checking `ActivatedRoute` in the constructor **sometimes works**

### 1. **Synchronous Component Creation**
If Angular **already finished navigating** and **your component is being constructed as a result**, then:
- The **route is already fully resolved**
- `ActivatedRoute.snapshot` has been populated
- So when the constructor runs, `route.snapshot` appears valid ✅

### Example (Working Scenario):
```ts
constructor(private route: ActivatedRoute) {
  console.log(this.route.snapshot.paramMap.get('id')); // Might work
}
```

If you're navigating from one route to another with `routerLink`, and the component is created **after** the router has resolved the route — it’ll appear to work.

---

### 2. **Route Snapshot is Eagerly Populated**
Angular's `ActivatedRoute.snapshot` is **synchronously available** for basic routing params *after routing completes*. So **simple routes without resolvers or async guards** will have valid snapshot data right away.

---

## ⚠️ But Here's the Danger

### It **breaks** in cases like:
- **Route resolvers or guards** (delays in route readiness)
- **Lazy-loaded modules**
- **Navigation that gets canceled or redirected**
- If Angular’s lifecycle changes in future versions (edge cases)

---

## 🔥 Senior Developer Insight:

> It works not because it's the right place to do it — it works because Angular has already done the hard work *before* your constructor runs **in some cases**.

But that's fragile — you're betting on the lifecycle timing **always being favorable**, which is risky.

---

### ✅ Best Practice
Always use `ngOnInit()` (or an observable like `route.params.subscribe`) when your logic depends on:

- `@Input()`
- Route data
- Async values (resolvers, query params)

---

## 👨‍🔧 TL;DR

| Reason it works sometimes | Why you shouldn't rely on it |
|--------------------------|------------------------------|
| Route snapshot is populated after routing | Not guaranteed in all cases |
| Component created after navigation completes | Fragile in async scenarios |
| Works in simple apps | Breaks in advanced routing setups |

---

Let me know if you want a live example that works in the constructor but fails when you introduce a resolver — it really drives the point home.
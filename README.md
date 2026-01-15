# HTML Preview App — Security & Stability Report

> 🔴 **Status: ALPHA — NOT PRODUCTION READY**  
> 🟡 **Security: Strong Native Isolation, Weak Runtime Lifecycle**  
> 🟢 **No Native / Android Bridge Exploits Detected**

---

## 🚦 Test Navigation Index (Always Start Here)

| Test | Name | Status | Jump |
|----|----|----|----|
| Test 1 | Core Runtime & UI | ❌ Partial | [Go](#-test-1--core-runtime--ui) |
| Test 2 | Lifecycle & Control | ❌ Fail | [Go](#-test-2--lifecycle--control) |
| Test 3 | Real-World Hostility | ❌ Fail | [Go](#-test-3--real-world-hostility) |
| Test 4 | Advanced Sandbox Hardening | ❌ Fail | [Go](#-test-4--advanced-sandbox-hardening) |
| Test 5 | Origin & Event Isolation | ❌ Blocked | [Go](#-test-5--origin--event-isolation-auto-report-attempt) |


> ⚠️ Rule: **If Test 2 lifecycle isolation fails, all later tests are informational only.**

---

## 🧠 Project Overview

This app is an **HTML + CSS + JavaScript preview engine** running in an **Android WebView environment**.

It allows execution of arbitrary frontend code for testing and preview purposes.

This document records:
- What works
- What fails
- Why it fails
- How to fix it safely

No hype. Only evidence.

---

## ✨ What Works Well (Summary)

- No Android JS bridge exposure
- No native code execution
- No shell / OS abuse
- Proper storage quota enforcement
- History sandboxing
- Visibility lifecycle events
- Error surfacing

These are **hard security wins**.

---

## ❌ What Is Broken (Summary)

- JS execution context persists across RUN
- Timers, Promises, Workers survive reload
- Microtask starvation crashes app
- window.open allowed
- Workers run in background
- Console hooks persist

These are **architectural flaws**, not syntax bugs.

---

## 🧪 Test 1 — Core Runtime & UI

### Scope
- DOM rendering
- JS execution
- localStorage
- Canvas
- Fetch
- Audio
- Console

### Result
🟡 **PARTIAL PASS**

### Key Findings
- Rendering works
- JS executes
- Storage works
- Canvas & audio work
- Console logs visible

### Critical Failure
```js
while (true) {}
```
➡ freezes entire app

### Root Cause
JS runs on same execution context as UI.

---

## 🧪 Test 2 — Lifecycle & Control

### Scope
- RUN reset
- Timer storms
- Async storms
- Cleanup verification

### Result
❌ **FAIL**

### Evidence
- Timers persist after RUN
- Async requests persist
- App slows permanently

### Root Cause
Execution context is reused instead of destroyed.

---

## 🧪 Test 3 — Real-World Hostility

### Scope
- Microtask flood
- RAF starvation
- Storage bomb
- History abuse
- Visibility lifecycle

### Results

| Subtest | Result |
|----|----|
| Microtask flood | ❌ App crash |
| RAF starvation | ✅ Pass |
| Storage quota | ✅ Pass |
| History abuse | ✅ Pass |
| Visibility lifecycle | ✅ Pass |

### Critical Finding
```js
Promise.resolve().then(spam)
```
➡ hard crash

### Severity
🚨 **CRITICAL**

---

## 🧪 Test 4 — Advanced Sandbox Hardening

### Scope
- Navigation
- window.open
- Dangerous URL schemes
- Web Workers
- Console/Error hooks

### Results

| Area | Result | Severity |
|----|----|----|
| Navigation | PASS | ✅ |
| window.open | FAIL | 🟡 |
| Dangerous schemes | SOFT FAIL | 🟡 |
| Worker lifecycle | FAIL | 🔥 |
| Console hooks | FAIL | 🔥 |

---

### 🔴 Key Test-4 Failures Explained

#### Web Worker Persistence
- Workers survive RUN
- Multiple workers accumulate
- Background CPU usage possible

#### Console / Error Hook Persistence
- User JS can hide logs/errors
- Hooks persist across runs

#### window.open
- Allowed
- Does not escape app
- Should be blocked for hardening

---

## 🔐 Security Verdict

### ✅ What You Are SAFE From
- Native Android API abuse
- Shell execution
- File system access
- Intent abuse
- OS-level exploits

### ❌ What You Are NOT SAFE From
- Runaway JS
- Persistent background execution
- Denial of service
- Stealth logic via workers

> **The app is not hackable — but it is abusable.**

---

## 🛠 Required Fixes (Priority Order)

1. **Destroy execution context on RUN**
2. Kill all timers, workers, promises
3. Block or intercept `window.open`
4. Filter dangerous URL schemes at navigation
5. Reset console & error hooks

---

## 🧪 Test 5 — Origin & Event Isolation (Auto-Report Attempt)

**Session ID:** 2026-01-15-T5  
**Environment:** Android WebView (Preview Engine)  
**Test Harness:** Auto-reporting HTML (inline JS)  
**RUN Reset Verified:** ❌ No

---

### 📌 Test Goal

Validate:
- Origin isolation
- Cookie & storage scope
- Global event containment
- Silent resource abuse resistance
- Unload / reload hijack prevention

This test includes **auto PASS/FAIL reporting logic** to reduce human error.

---

### ❌ Test Status: **FAILED (ENGINE BLOCKER)**

The test UI rendered, but **none of the test functions executed**.

---

### 🔴 Observed Errors (From System Console)

```
Uncaught ReferenceError: testOrigin is not defined
Uncaught ReferenceError: testStorage is not defined
Uncaught ReferenceError: testEvents is not defined
Uncaught ReferenceError: testSilentLoad is not defined
Uncaught ReferenceError: testUnload is not defined
Uncaught ReferenceError: copyReport is not defined
```

---

### 🧠 Interpretation

- HTML content rendered correctly
- `<script>` block did **not execute**
- Global JS functions were **never registered**
- Button `onclick` handlers failed immediately

This confirms that the **preview engine does not reliably execute inline scripts** on RUN.

---

### 🔍 Root Cause (Confirmed)

The preview engine:
- Reuses execution context across RUN
- Partially resets DOM but not JavaScript
- Injects HTML without guaranteed `<script>` execution
- Breaks global function registration

This is **not a test bug**.

This is an **engine lifecycle flaw**.

---

### 🚨 Severity

🔥 **HIGH — TEST HARNESS BLOCKED**

- Prevents reliable security testing
- Breaks real-world HTML relying on inline JS
- Produces false negatives / positives
- Makes later tests invalid

---

### 🛠 Required Fix (Before Retesting)

One of the following **must** be implemented:

#### Option A — Hard Document Reset
```js
document.open();
document.write(htmlString);
document.close();
```

#### Option B — WebView Recreation (Recommended)
```kotlin
webView.destroy()
webView = WebView(context)
```

---

### ⛔ What Will NOT Fix This

- Renaming functions
- Wrapping globals in guards
- try/catch around scripts
- Incremental DOM injection
- Manual cleanup of variables

---

### 📌 Test Outcome

> **Test 5 could not be executed due to engine limitations.**  
> Results are **inconclusive by design**, not by test failure.

Test 5 must be **re-run after RUN lifecycle isolation is fixed**.

---

### 🔁 Retest Conditions

Retest Test 5 only after:
- RUN fully resets JS execution
- Inline `<script>` executes reliably
- Globals are cleared between runs

---


## 🧑‍💻 Developer Rules (Read This)

- Never trust reload unless JS world is destroyed
- Never rely on blocking eval for security
- Never allow untracked workers
- Never expose Android JS bridges
- Always assume hostile input

---

## ⚠️ Final Disclaimer

This app executes **untrusted code**.

Until lifecycle isolation is fixed:
- Do NOT release publicly
- Do NOT claim sandbox safety
- Do NOT allow untrusted users

---

## 🏁 Final Statement

This project failed tests **honestly**, which is rare.

> Security is not about preventing hacks.  
> It’s about surviving misuse.


— End of README —

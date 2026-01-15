# HTML Preview App

> ⚠️ **Status: Alpha / Experimental**  
> ❌ **Not production-safe**  
> ✅ **Technically promising with clear remediation path**

This project is an **HTML + CSS + JavaScript preview engine** designed for Android environments.  
It allows users to execute arbitrary frontend code and view live output.

This repository documents **what works, what breaks, why it breaks, and how to fix it properly**.

---

## 📌 Table of Contents

- Overview
- Features
- Testing Methodology
- Final Test Results
- Critical Bugs
- Architecture Analysis
- Safe Execution Architecture (Proposed)
- Known Issues
- Developer Tips (Per Bug)
- Roadmap
- Contributing
- Disclaimer

---

## 🧠 Overview

The preview engine supports:
- Live HTML rendering
- JavaScript execution
- localStorage
- Canvas & animations
- Fetch API
- WebAudio
- Console logging
- Android WebView environment

The engine was subjected to **three phases of adversarial testing**, including hostile real-world abuse.

The goal was **truth**, not demo success.

---

## ✨ Features (Current)

- Real DOM rendering
- JavaScript execution
- Canvas animation support
- Fetch API support
- localStorage with quota enforcement
- Error surfacing with stack traces
- Console logging with throttling
- Visibility lifecycle awareness
- History sandboxing

---

## 🧪 Testing Methodology

### Phase 1 – Core Capability
- DOM manipulation
- localStorage persistence
- Canvas animation
- Fetch API
- WebAudio
- Error handling

### Phase 2 – Control & Lifecycle
- Timer storms
- Async request storms
- Resize spam
- Permission sandboxing
- Reload cleanup

### Phase 3 – Real-World Hostility
- Microtask starvation
- Render starvation
- Storage quota exhaustion
- History abuse
- Visibility lifecycle abuse

All tests were executed on **Android (WebView-based runtime)**.

---

## 📊 Final Test Results Summary

| Area | Result |
|----|----|
| Rendering | ✅ Pass |
| JS Execution | ✅ Pass |
| Error Reporting | ✅ Pass |
| Storage Quotas | ✅ Pass |
| History Isolation | ✅ Pass |
| Visibility Lifecycle | ✅ Pass |
| RAF Starvation | ✅ Pass |
| Timer Cleanup | ❌ Fail |
| Async Cleanup | ❌ Fail |
| Reload Isolation | ❌ Fail |
| CPU Freeze Protection | ❌ Fail |
| Microtask Control | ❌ **Critical Fail** |

---

## 🚨 Critical Bugs (Production Blockers)

### ❌ Bug 1: No Execution Isolation (CRITICAL)

**Reproduction**
```js
while (true) {}
```

**Observed**
- Entire app freezes
- Editor becomes unresponsive
- No recovery without force close

**Root Cause**
- Preview JS runs in the same execution context as UI

**Severity**
🚨 Critical

**Developer Tip**
> Never run untrusted JavaScript in the same thread or process as your UI.

---

### ❌ Bug 2: Microtask Starvation Crash (CRITICAL)

**Reproduction**
```js
function spam() {
  Promise.resolve().then(spam);
}
spam();
```

**Observed**
- App crashes
- Android kills process
- Reload impossible

**Why This Is Dangerous**
- Microtasks execute before rendering, timers, and input
- They bypass naïve watchdogs

**Severity**
🚨 Critical

**Developer Tip**
> Microtasks are invisible denial-of-service attacks if not isolated.

---

### ❌ Bug 3: Timer & Async Leaks Across RUN

**Reproduction**
1. Run timer storm
2. Press RUN again
3. App becomes sluggish permanently

**Observed**
- Timers survive reload
- Async requests persist
- Event loop congestion

**Root Cause**
- JS context reused
- No cleanup lifecycle

**Severity**
🔥 High

**Developer Tip**
> Reload must mean “destroy everything”, not “try again”.

---

## 🏗 Architecture Analysis (Current)

### Current Model
```
Editor UI
   ↓
Shared JS Runtime
   ↓
Preview Execution
```

### Problems
- No sandbox
- No lifecycle control
- No kill switch
- No memory or CPU limits

This architecture is **unsafe by design** for arbitrary code execution.

---

## 🛡 Proposed Safe Execution Architecture

### Minimum Acceptable Architecture (Android)

```
Editor UI (Process A)
   ↓ IPC
Preview WebView (Process B)
   ↓
Destroyed & recreated on every RUN
```

### Key Principles

- **Hard isolation** between editor and preview
- **Destroy execution context on RUN**
- **Kill switch** for runaway code
- **AbortController** for fetch cleanup
- **Timer registry** for forced cleanup

### Optional Enhancements
- Watchdog timer
- Memory usage monitoring
- Execution timeout
- Worker-based partial offloading

---

## 🐞 Known Issues

| Issue | Severity | Status |
|----|----|----|
| CPU freeze kills app | Critical | Open |
| Microtask starvation crash | Critical | Open |
| Timer leak across RUN | High | Open |
| Async request leak | High | Open |
| Partial reload inconsistencies | Medium | Open |

---

## 🧑‍💻 Developer Tips (Read This Carefully)

- **Never trust reload** unless the JS world is destroyed
- **Never assume Promises are safe**
- **Never allow unbounded timers**
- **Never share UI thread with untrusted JS**
- **Always assume hostile input**

If isolation is missing, no amount of JS patching will save you.

---

## 🛣 Roadmap

### Phase 1 – Safety (Mandatory)
- Execution isolation
- Hard reload semantics
- Emergency STOP / KILL

### Phase 2 – Control
- Timer & async cleanup
- Memory caps
- Execution watchdog

### Phase 3 – DX Improvements
- Multi-file virtual filesystem
- Permission toggles
- Export / share runnable previews

---

## 🤝 Contributing

This project welcomes contributors who understand:
- security
- runtime isolation
- WebView internals
- JavaScript execution models

### Guidelines
- Do not add features before fixing isolation
- All changes must include stress tests
- No UI-only PRs without runtime safety fixes
- Document failures honestly

---

## ⚠️ Disclaimer

This app **executes arbitrary code**.

Until execution isolation is implemented:
- Do NOT expose to untrusted users
- Do NOT publish as production-ready
- Do NOT claim sandbox safety

---

## 📌 Final Statement

This project is **not broken** — it is **unfinished**.

Most tools fail because developers stop testing too early.  
This one didn’t.

> Truth before polish. Safety before features.

— End of Documentation —

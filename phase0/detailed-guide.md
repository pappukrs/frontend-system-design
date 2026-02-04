# Phase 0: Frontend System Design Foundations

## Overview
Before diving into code, a Frontend Architect must understand the "why" and the "constraints". Frontend system design is NOT just about UI components; it's about building resilient, scalable, and performant systems that live in the browser.

---

## 0.1 What is Frontend System Design?

### UI Coding vs. System Design
- **UI Coding:** "How do I build this button and make it red?"
- **System Design:** "How do I build a design system that supports 100+ apps, ensures consistency, and loads only the CSS needed for this specific page?"

### Frontend vs. Backend System Design
| Feature | Frontend Design | Backend Design |
|---------|-----------------|----------------|
| **Environment** | Unreliable (Browsers, Devices, Networks) | Controlled (Servers, Cloud) |
| **State** | High-interactive, UI state, cache | Persistent data, ACID, transactions |
| **Scaling** | Client-side performance, Bundle size | Throughput, Latency, Sharding |
| **Security** | XSS, CSRF, Secure storage | SQLi, Auth, Rate limiting |

---

## 0.2 Non-Functional Requirements (NFRs)
These are the "invisible" features that define a senior engineer's work.

1.  **Performance:** Loading time (LCP), interactivity (INP), and smoothness (CLS).
2.  **Scalability:** Can the codebase handle 50 developers? Can the app handle 1,000 components?
3.  **Accessibility (a11y):** Is it usable by everyone (WCAG compliance)?
4.  **Security:** Protecting user data and preventing malicious scripts.
5.  **Maintainability:** How easy is it to add a feature 2 years from now?
6.  **Observability:** Error logging (Sentry), Analytics, and RUM (Real User Monitoring).

---

## 0.3 Design Thinking Framework

### 1. Requirements Clarification
Always ask:
- Who is the user? (Internal tool vs. Consumer app)
- What devices? (Mobile-only or Desktop heavy?)
- What's the network? (High-speed 5G or rural 3G?)

### 2. Failure Scenarios
- What if the API is slow? (Skeletons)
- What if the user is offline? (Service Workers / Persistence)
- What if the JS bundle fails to load? (Graceful Degradation)

---

## 📚 Recommended Resources
- [Google Web Vitals](https://web.dev/vitals/)
- [Pragmatic Programmer - Design by Contract]
- [MDN - Web Security]

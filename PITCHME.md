# Service Worker Operational Concerns

Operational concerns with the service worker spec and implementations

---

## Types of bug scenarios

* JavaScript bug in the app, which is now cached by the service worker
* Programmatic bug in the service worker code itself
* Error logs show bug caused by unexpected data in persistent storage
* Error logs show odd bug that we can't reproduce, don't know root cause, but suspect service worker
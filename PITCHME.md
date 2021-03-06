# Service Worker Operational Concerns

> If something goes wrong, can we recover?

---

## Scenarios

* Bug in app, cached by service worker
* Bug in service worker code
* Bad data in persistent storage
* Error logs show unreproducable bug, don't know root cause, but suspect service worker

---

## Simple noop script killswitch

```JavaScript
self.addEventListener('install', () => {
  return self.skipWaiting();
});
self.addEventListener('activate', (e) => {
  e.waitUntil(self.clients.claim());
});
```

---

## How to measure killswitch effectiveness?

* Test on real users (~3 million)
* Beacon event from service worker on basepage request
* Hit the killswitch
* Assert no repeat events (10 minute gap)
* Assert no events after full page load and script served (10 minute gap)

---

![swsse graph](https://raw.githubusercontent.com/asakusuma/sw-operational-concerns/master/images/swsse-graph.png "SWSSE Graph")

---

## Results
Week-long test, 3,727,164 Unique Devices
<table>
  <tr>
    <th>Test</th>
    <th>Devices that failed assertion</th>
    <th>Device fail %</th>
  </tr>
  <tr>
    <td>No repeat events</td>
    <td>608</td>
    <td>0.016%</td>
  </tr>
  <tr>
    <td>No event after load + script served</td>
    <td>329</td>
    <td>0.0088%</td>
  </tr> 
</table>

---

## Killswitch API Wishlist

* Reliable with time guarantee
* No obvious effects on users
* Support conditionally choosing per registration

---

## Why need to conditionally apply killswitch?

* Bugs that only affect a subset of users
* Bug in feature flagged code in service worker
* Bugs in specific versions of service worker
* Bugs in specific version of cached application

---

## The "Stateless Recovery Window" problem

* Problem related to service worker is detected
* Hit the "killswitch" to stop bleeding, `noop` is served
* Fix is released
* Killswitch is turned off
* User with bug returns after being offline since before killswitch was activated

---

## "Straw man" idea

"Soft Terminate", with programatic + `Clear-Site-Data` API. Terminates worker on whatever happens first

* 1 minute
* Full page load
* All tasks complete

Then activate installed worker if available

---
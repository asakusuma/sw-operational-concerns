# Service Worker Operational Concerns

> If something goes wrong, can we recover?

---

## Types of bug scenarios

* JavaScript bug in the app, which is now cached by the service worker
* Programmatic bug in the service worker code itself
* Error logs show bug caused by unexpected data in persistent storage
* Error logs show odd bug that we can't reproduce, don't know root cause, but suspect service worker

---

## Built simple noop script killswitch

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

## Basepage event graph after killswitch

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

## Need to conditionally apply killswitch

Bugs often only affect a subset of users, or aren't bad enough to warrant a refresh, but bad enough that you want 100% gaurantee that user will be fixed once they return to site.

---

## "Straw man" idea

"Soft Terminate", with programatic + `Clear-Site-Data` API. Terminates worker on whatever happens first:

* 1 minute
* Full page load
* All tasks complete

Then activate installed worker if available

---

## The "Stateless Recovery Window" problem

* Problem related to service worker is detected
* Hit the "killswitch" to stop bleeding, `noop` is served
* Fix is released
* Killswitch is turned off
* User with bug returns after being offline since before killswitch was activated

Some apps don't want to use `skipWaiting`, which doesn't seem to be 100% effective anyways


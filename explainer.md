# Abstract

The intent of this API is to expose memory pressure events received by the operating system to web applications. It will enable web applications to: 
 - optimize system responsiveness and improve the likelihood that the application does not get terminated - e.g. by releasing temporary objects to free up memory and stabilize the system before it begins terminating applications.
 - collect and correlate performance telemetry against memory pressure signals - e.g. correlate responsiveness, engagement, and other metrics when under memory pressure, etc.
 - provide a more resilient experience by persisting sensitive application and user data, such that it does not get lost if the application is about to be terminated.

# Use cases

* An application with an infinite-scroll wants to know if they need to remove items from their list and how aggressively.
* An application handling large files (e.g. photo/video, large documents/datasets, etc) wants to free up cache when the user's device needs it.
* An email application saving email content for fast interaction wants to free up the cache if it is needed by the system.
* An application gathering performance telemetry wants to correlate and segment various metrics on under-pressure devices.
* An application wants to detect memory leaks and regressions based on change in volume of under-pressure events.
* ...

# Proposal

We should be able to expose a `memorypressure` event with a level that can be “low”, “normal” and “high”.
The event could be exposed on the Navigator interface. Alternatively, it can hang out of navigator.memory for namespacing
(but at this point there is little value for that).

The event is to reflect system memory pressure events to the web application. The state isn’t reflected in an attribute
because to reflect how systems usually work: they send an event about memory pressure but usually do not send an event when
memory pressure is passed. The expectations from an application is that it will do the best effort to clean things up.

The level attribute of the event represents the level of memory pressure and how likely the browser is to kill the web
application (or the system to kill the browser): 
- "low" indicates that the system is under memory pressure and that the app should release memory if possible to improve its chances of survival.
- "normal" ... ?
- "high" indicates that the application is the next on the list to get killed and should release memory to avoid being terminated (and/or should persist its data to avoid data  loss).

The web application should free memory when receiving a memory pressure event and the user agent should do a garbage
collection after sending such event. If the page wants to be able to clean memory asynchronously, it can call
`event.preventDefault()` then later call the `done()` method to signal the UA the cleanup happened. The UA may ignore a
call to `preventDefault()` or run cleanup process before `done()` if required by the current memory state or system
conventions.

On platforms that support less than three levels of memory pressure events, it is recommended to implement at least “high”.
Web applications should not expect to receive memory pressure of incremental levels.

# Web IDL

```js
partial interface Navigator {
  attribute EventHandler onmemorypressure;
};

enum MemoryPressureLevel {
  “low”,
  “normal”,
  “high”
};

dictionary MemoryPressureEventInit : EventInit {
  required MemoryPressureLevel level;
};

[Constructor(DOMString level, MemoryPressureEventInit eventInitDict)]
interface MemoryPressureEvent : Event {
  readonly attribute MemoryPressureLevel level;
};
```

# Native APIs

## iOS
* https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/PerformanceTips/PerformanceTips.html (“Observe Low-Memory Warnings”).
* https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/macro/DISPATCH_SOURCE_TYPE_MEMORYPRESSURE  (search for “Memory pressure event flags”)

## Android
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int) 
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks.html#onLowMemory() 

## Desktop
TODO

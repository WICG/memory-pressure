# Abstract

The intent of this API is to expose the memory pressure events received by the system to web applications. It will enable
web applications to be memory conscious and free up cache, drafts and such temporary objects.

# Use cases

* An application with an infinite-scroll wants to know if they need to remove items from their list and how agressively.
* An application handling large files (photo/video sharing for example) wants to free up cache when the user's device need it.
* An email application saving email content for fast interaction wants to free up the cache if it is needed by the system.
* ...

# Proposal

We should be able to expose a `memorypressure` event with a level that can be “low”, “normal” and “high”.
The event could be exposed on the `Navigator` interface. Alternatively, it can hang out of navigator.memory for namespacing
(but at this point there is little value for that).

The event is to reflect system memory pressure events to the web application. The state isn’t reflected in an attribute
because to reflect how systems usually work: they send an event about memory pressure but usually do not send an event when
memory pressure no longer applies. The expectations from an application is that it will do the best effort to clean things up when needed.

The level attribute of the event represents the level of memory pressure and how likely the browser is to kill the web
application (or the system to kill the browser). After receiving a “high” memory pressure event, a web application should
assume that it might die soon after if it was not able to clean up memory. The web application should free memory when receiving a memory pressure event and the user agent should do a garbage collection after sending such event.

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

# Alternative approach

An alternative approach would be to allow web pages to ask to deal with memory pressure asynchronously. This comes with a few drawbacks but for completeness, it is mentioned here. In such a situation, we could imagine that the event can be cancelled using `event.preventDefalt()` and a method called `done()` could be added to the event interface. If `event.preventDefault()` is called, the UA would wait for `done()` to be called before running garbage collection or run it again if it did before. It has the benefit of being more flexible for heavily asynchronous web applications. However, it is unlikely that the UA will have the time to wait for the web application in most cases. Furthermore, it would expose garbage collection to the web pages.

# Native APIs

## iOS
* https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/PerformanceTips/PerformanceTips.html (“Observe Low-Memory Warnings”).
* https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/macro/DISPATCH_SOURCE_TYPE_MEMORYPRESSURE  (search for “Memory pressure event flags”)

## Android
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int) 
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks.html#onLowMemory() 

## Desktop
TODO

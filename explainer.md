# Abstract

The intent of this API is to expose the memory pressure events received by the system to web applications. It will enable
web applications to be memory conscious and free up cache, drafts and such temporary objects.

# Proposal

We should be able to expose a `memorypressure` event with a level that can be “low”, “normal” and “high”.
The event could be exposed on the Navigator interface. Alternatively, it can hang out of navigator.memory for namespacing
(but at this point there is little value for that).

The event is to reflect system memory pressure events to the web application. The state isn’t reflected in an attribute
because to reflect how systems usually work: they send an event about memory pressure but usually do not send an event when
memory pressure is passed. The expectations from an application is that it will do the best effort to clean things up.

The level attribute of the event represents the level of memory pressure and how likely the browser is to kill the web
application (or the system to kill the browser). After receiving a “high” memory pressure event, a web application should
assume that it might die soon after if it was not able to clean up memory.

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

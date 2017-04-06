# Abstract

The intent of this API is to expose the memory pressure events received by the system to web applications. It will enable
web applications to be memory conscious and free up cache, drafts and such temporary objects.

# Use cases

* An application with an infinite-scroll wants to know if they need to remove items from their list and how agressively.
* An application handling large files (photo/video sharing for example) wants to free up cache when the user's device need it.
* An email application saving email content for fast interaction wants to free up the cache if it is needed by the system.
* ...

# Proposal

We should be able to expose a `memorypressure` event exposed on the `Navigator` interface. The event would be a simple event
and would have a `onmemorypressure` `EventHandler` on the `Navigator` object.

The event reflects system memory pressure events to the web application. The state is not reflected in an attribute
in order to reflect how systems usually work: they send an event about memory pressure but usually do not send an event when
memory pressure no longer applies. The expectations from an application is that it will do the best effort to clean things up when needed.

# Web IDL

```js
partial interface Navigator {
  attribute EventHandler onmemorypressure;
};
```

# Potential extension

Some platforms expose various memory pressure events. This proposal do not handle them as it is not even always clear how
native applications should react to various memory pressure types. Instead of exposing memomry pressure types without a defined
behaviour, this proposal prefer to leave the types as a potential extension to the event.

# Native APIs

## iOS
* https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/PerformanceTips/PerformanceTips.html (“Observe Low-Memory Warnings”).
* https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/macro/DISPATCH_SOURCE_TYPE_MEMORYPRESSURE  (search for “Memory pressure event flags”)

## Android
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int) 
* http://developer.android.com/intl/zh-tw/reference/android/content/ComponentCallbacks.html#onLowMemory() 

## Desktop
TODO

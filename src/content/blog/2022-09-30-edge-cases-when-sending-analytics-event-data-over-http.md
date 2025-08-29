---
title: 'Edge cases when sending analytics event data over HTTP'
description: 'Some corner cases I had to tackle while sending analytics events from client-side JS'
pubDate: '2022-06-22'
---

The Fetch API is the go-to method to manage HTTP calls in the browser. However, it is not always the right tool.<br/>
In this post we'll go through some corner cases I had to tackle while sending analytics events from client-side JS.

### TLDR

-   Generally, for analytics events use the Beacon API.
-   If you need to set custom headers in your API call, use the Fetch API with the `keepalive` flag.
-   If you need custom headers AND must support Safari and Firefox, use syncronous XML requests (when navigating to a new page while sending the event)

# Why sending analytics events is different from normal HTTP calls

When sending tracking data from the front-end up to a backend API, our HTTP calls should be small and fast. That's because we might perform many of them and they shouldn't take away resources from the app.<br/>
In practice, the idea is to take away all the unnecessary and make these calls as lightweight as possible. That's what the beacon API is for.

# The Beacon API

The beacon API sends basic HTTP requests (POST) to a server without requiring a response status. That makes them significanlty faster.<br/>
Another advantage is that the beacon API will perfom a successful HTTP call even when navigating to a different page while sending the event. That is because it runs synchronously with your JS, which must finish executing before unloading the current page (this doesn't happen with async requests made by the Fetch API).<br/>

```javascript
navigator.sendBeacon('<your api endpoint>', your_event_data)
```

However, this API only performs basic requests. This means that it doesn't support custom headers.

# Analytics requests with custom headers

When having to include custom headers, the Fetch API can be used. To avoid killing the request while unloading the page, you can use the `keepalive` flag. This option allows the request to outlive the page it was intialized from.<br/>

```javascript
fetch('<your api endpoint>', {
	method: 'POST',
	headers: your_custom_headers,
	body: your_event_data,
	keepalive: true,
})
```

However, this flag is only supported by chromium browsers (at the time of writing this).

# Custom headers + browser support

Use the above methods as much as you can. However, if your request must contain custom headers, could happen while unloading the current page AND must work on all major browsers, there's one solution.<br/>
If you have been developing for a while you might remember using XML requests to send data over HTTP.
XML requests can be performed synchronously, avoinding the issue of the page being unloaded while performing the request.<br/>
**Note**: you should only use this method for requests that you know will happen just before unloading the page, and nothing else. This method will pause the JS execution and wait for the call to finish before continuing (it doesn't "escape" the event loop).<br/>
To use this technique set the async flag to false in your XML request.

```javascript
const req = new XMLHttpRequest()
req.open('POST', '<your api endpoint>', false)

req.setRequestHeader('custom_header', custom_header_value)
req.send(JSON.stringify(your_event_data))
```

Hope you found this notes useful 😄.

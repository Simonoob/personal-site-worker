---
title: 'Track Youtube embedded videos with YT API'
description: 'Handy snippet to track all embeded YT videos on a page.'
pubDate: '2022-06-22'
---

I recently had to track multiple embeded YT videos on a page.
The idea is simple: use the YouTube API to catch events coming from from the iframes, and trigger functions on such events.
However, I found a couple of gotchas along the way and I had to work around them.

The following snippet allows you to fire functions on video player state change, for both regular and lazy loaded YT embeds.
Hopefully it saves you (or future me) some time.

```js
const trackYTembeds = () => {
	const iframes = document.querySelectorAll('iframe')

	if (!iframes.length) return

	// Filter YT iframes
	const ytIframes = Array.from(iframes).map(iframe => {
		//Check src if normal iframe
		if (iframe.src && iframe.src.match(/youtube(-nocookie)?.com\/embed/i))
			return iframe
		//check dataset if lazy loaded
		if (
			iframe.dataset?.src &&
			iframe.dataset.src.match(/youtube(-nocookie)?.com\/embed/i)
		)
			return iframe
	})

	if (!ytIframes.length) return

	// Load the Youtube JS api (to catch messages coming from the iframes)
	const tag = document.createElement('script')
	const firstScript = document.getElementsByTagName('script')[0]
	tag.src = 'https://www.youtube.com/iframe_api'
	tag.async = true
	firstScript.parentNode.insertBefore(tag, firstScript)

	window.onYouTubeIframeAPIReady = () => {
		// Init tracking for each Youtube video
		ytIframes.forEach(iframe => {
			let srcOrigin = iframe.src || iframe.dataset.src
			// Allow the use of the API on the iframe
			if (!srcOrigin.match(/enablejsapi=1/i)) {
				srcOrigin = `${srcOrigin}${
					srcOrigin.match(/\?/i) ? '&' : '?'
				}enablejsapi=1`
			}

			// Modify the original url source so it will stay in sync for all attributes
			iframe.dataset.src
				? (iframe.dataset.src = srcOrigin)
				: (iframe.src = srcOrigin)

			// Fix src attribute in lazyloadedn YT iframes (issue: https://github.com/aFarkas/lazysizes/issues/529)
			if (iframe.dataset.src) iframe.src = iframe.dataset.src

			// ID attribute is needed by the YouTube API to find the player
			iframe.id = iframe.src

			// Init a JS player to execute functions on video state changes
			new window.YT.Player(iframe.id, {
				events: {
					onStateChange: handlePlayerStateChange,
				},
			})
		})
	}

	const handlePlayerStateChange = e => {
		const playerStatus = e.data

		switch (playerStatus) {
			case 1: // Playing
				alert('Video playing')
				break

			case 2: // Paused
			case 0: // Ended
				alert('Video paused or ended')
				break
			default:
				break
		}
	}
}
```

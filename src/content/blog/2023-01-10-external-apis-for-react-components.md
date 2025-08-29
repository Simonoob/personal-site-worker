---
title: 'External APIs for React Components'
description: 'Leveraging the `useImperativeHandle` hook to expose functions as an API.'
pubDate: '2023-01-10'
---

# Why expose an API from a React component?

An API allows other parts of your front-end app to interact with the component
in a controlled manner. It can be especially useful when you need to interact
with a component from outside your React root element.
A prime example are web-components, custom HTML elements that can be used and controlled from any webpage.

In this short post, we'll look at leveraging the `useImperativeHandle` hook to expose functions as an API.

# Basic example

The useImperativeHandle hook lets you take control of a component instance from
its parent. It takes two arguments: the ref object passed from the parent
component, and an object that defines the exposed functions.

Here's an example:

```jsx
import { forwardRef, useImperativeHandle, useRef } from 'react'

const ChildWithRef = forwardRef((props, ref) => {
	useImperativeHandle(ref, () => ({
		sayHello: () => {
			console.log('Hello!')
		},
	}))

	return <div>I am a child with a ref</div>
})

const Parent = () => {
	const childRef = useRef(null)

	const handleSayHello = () => {
		childRef.current.sayHello()
	}

	return (
		<div>
			<ChildWithRef ref={childRef} />
			<button onClick={handleSayHello}>Say hello</button>
		</div>
	)
}
```

In this example, we expose a sayHello function from the ChildWithRef component. This function can be called from the parent using the ref object passed to the child.

## Interactive example

Here's a more practical example that exposes an API to control the behavior of a 3D cube.
To allow for more control,You could store the ref in a global variable and use
its API from anywhere in your front-end.

  <iframe
    src="https://codesandbox.io/embed/r3f-starter-forked-2s17it?fontsize=14&hidenavigation=1&moduleview=1&theme=dark&view=preview"
    title="useImperativeHandle example"
    allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
    sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
          style="width:100%; aspect-ratio:16/9;"
  ></iframe>

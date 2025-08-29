---
title: 'GLSL Shaders'
description: 'A collection of older GLSL shader'
pubDate: '2023-09-08'
---

This is a list of older GLSL shaders I've made over the last year, in no particular order.
They all run in a WebGL context, all the ones not hosted on ShaderToy are run via Three.js.

## Step Perlin Noise

An implementation of perlin noise (by Stefan Gustavson), combined with a step function to force the pixels to be black or white.

<iframe src='https://basic-shaders-iew7wm3nj-simonoob.vercel.app/#debug' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## UV Color Sampling

An attempt at using UV coordinates values for pixel colors, and a step on a wave function for the shape.

<iframe src='https://basic-shaders-bfons4gqn-simonoob.vercel.app/#debug' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## Rough Sea

Raiging sea effect created by applying vertex and fragment shaders with noise to a plane geometry in ThreeJS.

<iframe src='https://29-raging-sea-eight.vercel.app/' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## Fractal Drop

Fragment shader using fractals and SDFs to create a 'dancing' texture.

<iframe src='https://www.shadertoy.com/embed/dl3Szl?gui=false&t=10&paused=false&muted=true' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## Fractal Brownian Motion

Using fractal brownian motion noise to generate an ever changing smooth texture.

<iframe src='https://fractal-brownian-motion.vercel.app/' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## Image Pixelator

Image pixelation tool powered by a fragment shader.
[**LINK**](https://image-pixelator.vercel.app/)

## Block Sampling Texture

Using custom texture sampling to distort an image into a grid of blocks.

<iframe src='https://block-sampling-texture.vercel.app/#debug' style='width:100%; aspect-ratio: 16/9' loading="lazy"></iframe>

## Color Layering

Multiple texture sampling instances to individually source R G B colors and displace them separately.
[**LINK**](https://color-displacement.vercel.app/)

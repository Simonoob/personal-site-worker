---
title: 'Block Sampling Textures in a GLSL shader'
description: |-
    I've started to experiment with textures in my GLSL shaders.
    Check out how I created this block sampling effect.
author:
    - 8744e0bf-6d5b-442a-885d-b823fdfb28b5
pubDate: '2022-04-02'
---

I've recently started to experiment with textures in my GLSL fragment shaders. This post will break down the logic behind the effect below in GLSL. In doing that, we'll explore texture sampling and how to manipulate it to produce different outputs.

  <iframe title="final project" src="https://block-sampling-texture.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

## NOTE

I created the project as part of the [Shaders for the Web](https://www.superhi.com/courses/shaders-for-the-web) course over at [Superhi.com](https://superhi.com/). If you want a step by step tutorial, please take that course.

## Step 1: Translate the texture on the shaded surface

Using a texture within a shader is to look up the colour info of the current fragment from the texture image.
The first stage is to just use the fragment colour from the image as the final colour of the current fragment. This will just translate the texture to the final image.

```glsl
//LOAD EXTERNAL INPUTS
varying vec2 vUV; //coordinates of the current fragment on the plane
uniform sampler2D uTexture; //texture information

//function to compute the colour of the current fragment on the plane
void main()
{
    vec4 textureColor = texture2D(uTexture, vUV); //get color from matching fragment

    gl_FragColor = textureColor; //assign the color
}
```

At this point, we might get some weird stretching in the final output. That's because the aspect ratio of the texture is different from the aspect ratio of the surface we apply the shader on.

To solve this we must match the aspect ratio of the texture in the shader. Let's create a function for that (before the `main` function definition). The function will modify the UV coordinates based on the aspect ratios.

```glsl
//get right aspect ratio
vec2 getAspectRatio(vec2 uv, float surface_ratio, float texture_ratio) {
    if(texture_ratio > surface_ratio) {
        //image is wider than surface
        //stretch the surface on x by the difference
        float ratio_diff = surface_ratio/texture_ratio;
        uv.x *= ratio_diff;
        //adjust the new uv coordinates to center the image
        uv.x += (1.0 - ratio_diff) / 2.0;
    } else {
        //image is taller than surface
        //stretch the surface on y by the difference
        float ratio_diff = texture_ratio/surface_ratio;
        uv.y *= ratio_diff;
        //adjust the new uv coordinates to center the image
        uv.y += (1.0 - ratio_diff) / 2.0;
    }
    return uv;
}
```

Now let's pass the texture and surface dimensions to the shader.

```glsl
//LOAD EXTERNAL INPUTS
varying vec2 vUV; //coordinates of the current fragment on the plane
uniform sampler2D uTexture; //texture information
uniform vec2 uSurfaceResolution; //surface X and Y dimentions
uniform vec2 uTextureResolution; //texture X and Y dimentions
```

Computing the right `uv` coordinates then looks like this.

```glsl
//get aspect ratios
float surfaceRatio = uSurfaceResolution.x/uSurfaceResolution.y;
float textureRatio = uTextureResolution.x/uTextureResolution.y;

//Correct aspect ratio for Texture rendering
vec2 uv = getAspectRatio(uv, surfaceRatio, textureRatio);
vec4 textureColor = texture2D(uTexture, uv); //get color from matching fragment

gl_FragColor = textureColor; //assign the color
```

  <iframe title="step 1" loading="lazy" src="https://texture-stretched-glsl-91t67zq4y-simonoob.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

## Step 2: Sample by block instead of fragment

the next step is to understand how to divide the final output in an arbitrary grid instead of sampling by fragment.
In other words, we want to "combine" multiple fragments in a square and give them all the same value, then repeat for the whole grid.

Let's start by creating a grid of "blocks".

```glsl
//sample block instead of individual fragment

float blocks = 12.0;

vec2 block = vec2(
    floor(uv.x * blocks)/blocks,
    floor(uv.y * blocks)/blocks
);
```

The code above groups the `uv` coordinates by the number of rows and columns we want to create. The `floor()` operation is important since it assigns the same value to elements close together (by removing the decimal point).

This leaves us with a grid of blocks. Each fragment in the block holds the same `uv` values (X and Y).

To see this in action we can modify our shader to sample the texture based on the current block.

```glsl
//add color
vec4 textureColor = texture2D(uTexture, block);

gl_FragColor = textureColor;
```

Because the fragments in the block hold the same `uv` coordinates, `texture2D()` will "look up" the same fragment info for the whole block.

  <iframe title="step 2" loading="lazy" src="https://texture-stretched-glsl-6802ihlqj-simonoob.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

We created a pixelated version of the texture!
That's cool, but we still want to show every fragment from the image, we ony want to _move_ them based on the blocks.

### Step 3: Use blocks as distortion

Let's get back to sampling by fragment but let's interpolate it with the values from the blocks.

```glsl
//              |limit| |X and Y distortion|
vec2 distortion = 0.1 * vec2(block);

//add color
vec4 textureColor = texture2D(uTexture, uv + distortion);

gl_FragColor = textureColor;
```

Now the grid is still slightly visible but only _distorts_ the sampling point.

  <iframe title="step 3" loading="lazy" src="https://texture-stretched-glsl-3g93a23za-simonoob.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

However, we have a problem. If you look closely, at the positive Y and/or X border there are some weird "stripes". In my case, you can see them towards the right border. Why is that?

Let's backtrack on how we distort the `uv` coordinates when sampling the texture.
We take the `uv` values and add the `block` values (capped at `0.1`). This means that for
`uv = vec2(0.5)` we will get

`sampling point = vec2(0.5 + block) => vec2(0.5 + 0.1*floor(0.5 * 6)/6) => vec2(0.5 + 0.05)`

so `sampling point = vec2(0.55)`.
That works fine as the available sampling points go from `0` to `1`.
When we have `uv = vec2(1.0)` however, `sampling point = vec2(1.1)`. We ran out of sampling points! there's no `(1.1, 1.1)` point to sample from. Because of that `texture2D()` keeps using the last available point until the end, creating that striped effect.

To solve this we must make sure that we never run out of sampling points when distorting. A simple solution is to zoom the picture by the maximum possible distortion, i.e. `0.1`.
Let's do that before making any alterations to the `uv`.

```glsl
//zoom the image in to add a "padding" around the image to distort into
uv = mix(vec2(0.1), vec2(0.9), uv);
```

<iframe title="step 4" loading="lazy" src="https://texture-stretched-glsl-gxezxw648-simonoob.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

### Step 4: Add motion

At this point, we have all we want, but it is still static. We can add motion to it by pairing the distortion to a number that changes over time. The obvious choice is to use the elapsed time.

```glsl
//              |limit| |X and Y distortion|
vec2 distortion = 0.1 * vec2(sin(block + time));
```

  <iframe title="step 5" loading="lazy" src="https://texture-stretched-glsl-gg4jcqo6v-simonoob.vercel.app/" style="width:100%; aspect-ratio:16/9;"></iframe>

It's alive!

### Step 5: Interactivity

From here on we can go crazy and add all kinds of parameters to the distortion. I added time as well as mouse position on hover.

The final version has also sliders attached to the parameters, so you can play around with them in real-time.

#### [Check it out here](https://block-sampling-texture.vercel.app#debug)

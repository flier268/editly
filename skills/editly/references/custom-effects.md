# Custom effects

`progress` on every callback below is `0` at the layer's `start` and `1` at its `stop` (or the end of the clip). It is not a wall-clock second. Draw the whole frame from `progress` each call. Editly does not keep your previous drawing.

The `editly` binary JSON5-parses its spec file, including a `.js` path. A function in that file never runs. `gl` with a fragment file is the custom effect that stays in JSON5. `canvas`, `fabric`, and `fabricImagePostProcessing` need a Node script:

```js
import editly from "@flier268/editly";

await editly({
  outPath: "./out.mp4",
  width: 1280,
  height: 720,
  fps: 30,
  clips: [
    /* layers below */
  ],
});
```

Run that file with `node` (ESM) or `tsx`. `ffmpeg` and `ffprobe` must be on `PATH`.

## GLSL (`type: "gl"`)

Valid in JSON5:

```json5
{ type: "gl", fragmentPath: "./effect.frag", speed: 1 }
```

`speed` defaults to `1`. Editly binds only two uniforms, then draws a fullscreen quad:

- `resolution`: `vec2`, output width and height in pixels
- `time`: `float`, `progress * speed`, so it runs from `0` to `speed` across the layer

No other uniform is set. A custom vertex shader must declare `attribute vec2 position`. If `vertexPath` and `vertexSrc` are both omitted, this vertex shader is used:

```glsl
attribute vec2 position;
void main(void) {
  gl_Position = vec4(position, 0.0, 1.0);
}
```

Minimal fragment shader:

```glsl
#ifdef GL_ES
precision mediump float;
#endif

uniform float time;
uniform vec2 resolution;

void main() {
  vec2 st = gl_FragCoord.xy / resolution.xy;
  gl_FragColor = vec4(st.x, st.y, abs(sin(time)), 1.0);
}
```

If both a path and an inline source are set, the file wins: `fragmentPath` overwrites `fragmentSrc`, and `vertexPath` overwrites `vertexSrc`.

## node-canvas (`type: "canvas"`)

`func` is called once. Every frame, editly clears the canvas, then calls `onRender(progress)`. Draw with the 2D context. Return nothing from `onRender`. `onClose` runs when the clip's frame source closes.

```js
{
  type: "canvas",
  func({ canvas }) {
    const context = canvas.getContext("2d");
    return {
      onRender(progress) {
        const radius = 40 * (1 + progress * 0.5);
        context.beginPath();
        context.arc(canvas.width / 2, canvas.height / 2, radius, 0, 2 * Math.PI);
        context.fillStyle = "hsl(350, 100%, 37%)";
        context.fill();
      },
    };
  },
}
```

The returned pixels become this layer. Later layers in the same clip are composited above them.

## Fabric (`type: "fabric"`)

`func` is called once with `{ width, height, fabric, params }`. `params` is this layer object. `onRender(progress, canvas)` receives the clip's `StaticCanvas`. Lower layers of this frame are already on it.

Add objects with `canvas.add`. Do not call `clear`, `renderAll`, or `dispose`. Editly renders and clears that canvas after the last layer. A new canvas is created for every frame, so create objects inside `onRender`.

```js
{
  type: "fabric",
  func({ width, height, fabric }) {
    return {
      onRender(progress, canvas) {
        canvas.add(
          new fabric.FabricText(`${Math.floor(progress * 100)}%`, {
            originX: "center",
            originY: "center",
            left: width / 2,
            top: height / 2,
            fill: "white",
          }),
        );
      },
    };
  },
}
```

Default object origin is the top-left. Set `originX` and `originY` when a center anchor is required. Register a custom font with `registerFont` from `canvas` before calling `editly`, then set `fontFamily` to that family name.

## Video frame hook (`fabricImagePostProcessing`)

Only on `type: "video"`. Called after the frame is placed and before that image is `canvas.add`ed. Arguments: `{ image, canvas, fabric, progress, time }`. `time` is seconds from this layer's `start`, not from the start of the output file.

The caller does not await the return value. Finish the work before returning.

```js
{
  type: "video",
  path: "./clip.mp4",
  resizeMode: "cover",
  width: 0.5,
  height: 0.5,
  position: { x: 0.5, y: 0.5, originX: "center", originY: "center" },
  fabricImagePostProcessing({ image, canvas, fabric }) {
    const circle = new fabric.Circle({
      radius: Math.min(image.width, image.height) * 0.4,
      originX: "center",
      originY: "center",
    });
    image.set({ clipPath: circle });
    const center = image.getCenterPoint();
    canvas.add(
      new fabric.Circle({
        radius: circle.radius,
        originX: "center",
        originY: "center",
        left: center.x,
        top: center.y,
        fill: "transparent",
        stroke: "white",
        strokeWidth: 22,
      }),
    );
  },
}
```

`image` is the frame already scaled to this layer. Extra objects added to `canvas` sit behind the frame, because the frame itself is added afterward.

# Sprite Packer

A small browser tool that packs frame-by-frame PNG animations into a single **sprite sheet (PNG)** and an **atlas file (JSON)** that loads straight into [Phaser 3](https://phaser.io/).

It's a lightweight alternative to TexturePacker for the common case: you have a folder of PNG frames and you need a sprite sheet for Phaser.

- **No install, no build, no server.** It's one `index.html` file.
- **Runs entirely in your browser.** Your images are never uploaded anywhere.
- **Phaser-ready output** in the TexturePacker "JSON Hash" format, which Phaser 3 reads with `this.load.atlas()`.

## Features

- Drag and drop PNG files or a whole folder.
- Frames are sorted by file name using natural order (`walk_2` comes before `walk_10`). That order is the animation order.
- Tight packing with the MaxRects algorithm.
- **Per-group trim.** Select frames, put them in a group, and turn transparent-border trimming on or off for that group. Trimmed and untrimmed frames still end up on the same sheet. Trim offsets are saved in the JSON, so frames don't shift in game.
- Sheet options: padding, max size (1024–8192), power-of-two size, keep or strip `.png` in frame names.
- Animation preview with FPS, loop, and on-complete behavior (stop on last frame, return to first frame, or hide sprite).
- **Test in Phaser 3.** Loads the generated PNG + JSON into a real Phaser instance and plays it.
- Generates ready-to-copy Phaser code for loading the atlas and creating the animation.

## Getting started

1. Download or clone this repository.
2. Open `index.html` in a modern browser (Chrome, Edge, Firefox or Safari).

That's it. To host it, put `index.html` on any static host, such as GitHub Pages, Netlify or your own server.

> The **Run in Phaser** test loads Phaser 3.80.1 from the jsDelivr CDN, so it needs an internet connection. Everything else works offline.

## How to use

### 1. Add frames

Drop PNG files or a folder onto the left panel, or use **+ Files** / **+ Folder**.

All frames start in the **Ungrouped** list. The number next to each frame is its position in the animation, based on its file name. Adding a file with the same name as an existing frame replaces that frame.

### 2. Group frames and set trim (optional)

Use groups when some frames should be trimmed and others shouldn't.

1. Click frames to select them. **Shift+click** selects a range, and the checkbox on a group header selects the whole group.
2. Click **New group** to put the selected frames into a new group (trim is on by default for new groups), or use **Move to…** to move them into an existing group.
3. Tick **Trim** on a group header to trim every frame in that group. The Ungrouped list has its own Trim checkbox too.
4. Click a group's name to rename it. **Ungroup** moves its frames back to Ungrouped.

Groups only control trimming. They don't change frame names or animation order.

### 3. Adjust options

The right panel has these options:

| Option | Description |
|---|---|
| Padding (px) | Space between frames on the sheet. Default `0`. |
| Max size | Largest allowed sheet width/height. |
| Power of two size | Rounds the sheet size up to 256, 512, 1024, and so on. |
| Keep .png in frame names | Names frames `hero_001.png` (on) or `hero_001` (off). |
| FPS | Animation frame rate, used in the preview and generated code. |
| Loop | Loops forever (`repeat: -1`) or plays once (`repeat: 0`). |
| On complete | When Loop is off: stop on last frame, return to first frame, or hide sprite. |
| File name | Base name for the exported `.png` and `.json`. |

The center panel shows the sheet with frame bounds (colored by group), the animation preview, the Phaser test, the generated code and the raw JSON.

### 4. Export

Click **Download PNG + JSON** and put both files in your game's assets folder.

## Using the output in Phaser 3

```js
// preload()
this.load.atlas('sprite', 'sprite.png', 'sprite.json');

// create()
this.anims.create({
  key: 'sprite_anim',
  frames: this.anims.generateFrameNames('sprite', {
    prefix: 'Scatter_', suffix: '.png',
    start: 0, end: 9, zeroPad: 5
  }),
  frameRate: 24,
  repeat: -1
});

const sprite = this.add.sprite(400, 300, 'sprite').play('sprite_anim');
```

The tool writes this code for you, filling in your frame names, FPS and loop settings.

### JSON format

Each frame entry follows the TexturePacker JSON Hash layout:

```json
{
  "frames": {
    "Scatter_00000.png": {
      "frame": { "x": 0, "y": 0, "w": 179, "h": 181 },
      "rotated": false,
      "trimmed": true,
      "spriteSourceSize": { "x": 158, "y": 164, "w": 179, "h": 181 },
      "sourceSize": { "w": 500, "h": 500 }
    }
  },
  "meta": {
    "app": "Sprite Packer",
    "version": "1.0",
    "image": "sprite.png",
    "format": "RGBA8888",
    "size": { "w": 418, "h": 847 },
    "scale": "1"
  }
}
```

- `frame`: where the frame sits on the sheet.
- `spriteSourceSize`: where the trimmed image sits inside the original frame.
- `sourceSize`: the original, untrimmed frame size.

Phaser uses `spriteSourceSize` and `sourceSize` to put trimmed frames back in their original position.

## Compared with TexturePacker

Sprite Packer covers the basic job of packing frames for Phaser. It does **not** support:

- frame rotation
- duplicate-frame aliasing
- polygon trimming
- multipack (splitting across several sheets)
- extrude
- image compression or reduced color formats
- multiple resolutions (@1x/@2x)
- export formats other than Phaser JSON Hash
- a CLI or folder watching
- saving project settings

If you need any of these, use [TexturePacker](https://www.codeandweb.com/texturepacker).

## Browser support

Any current version of Chrome, Edge, Firefox or Safari. Dropping a folder uses the browser's directory drag-and-drop support, which all of these browsers have.

## Limitations

- Settings and groups aren't saved. Reloading the page clears everything.
- Very large frame sets may take a moment to pack, because all processing happens in the browser.

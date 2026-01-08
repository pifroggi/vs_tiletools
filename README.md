
# Tiling and Padding functions for VapourSynth
A collection of spatial and temporal tiling and padding utilities for VapourSynth. The original idea was just a tiling function to make AI filters less VRAM-hungry and to provide additional options that built-in solutions might not. Over time, more related functions were added.

The functions often come in pairs, with one doing a thing and the other inversing it. For example:
```python
import vs_tiletools
clip = vs_tiletools.tile(clip, width=256, height=256) # splits frames into 256x256 tiles
clip = core.someheavyfilter.AIUpscale(clip)           # placeholder resource intensive filter
clip = vs_tiletools.untile(clip)                      # reassembles the tiles into full frames
```

<br />

## Table of Contents
* [Requirements](#requirements)
* [Setup](#setup)
* [Spatial Functions](#spatial-functions)  
<sub>     *Tiling*</sub>  
  ⚬ [Tile](#tile) - Splits each frame into tiles of fixed dimensions  
  ⚬ [Untile](#untile) - Auto reassembles tiles from `tile()`, even if resized  
<sub>     *Padding/Cropping*</sub>  
  ⚬ [Pad](#pad) - Pads a clip with various padding modes  
  ⚬ [Mod](#mod) - Pads or crops a clip so width and height are multiples of the given modulus  
  ⚬ [Crop](#crop) - Auto crops padded clip from `pad()` or `mod()`, even if resized  
  ⚬ [Croprandom](#croprandom) - Crops to given dimensions, but randomly repositions the window each frame  
<sub>     *Inpainting/Filling*</sub>  
  ⚬ [Fill](#fill) - Fills the borders of a clip with various filling modes  
  ⚬ [Autofill](#autofill) - Auto detects borders and fills them with various fill modes  
  ⚬ [Inpaint](#inpaint) - Inpaints areas based on a mask with various inpainting modes
* [Temporal Functions](#temporal-functions)  
<sub>     *Duplicate Detection*</sub>  
  ⚬ [Markdups](#markdups) - Marks identical frames as duplicates, which can later be skipped using `skipdups()`  
  ⚬ [Skipdups](#skipdups) - Skips processing of duplicate frames marked by `markdups()`  
<sub>     *Windowing*</sub>  
  ⚬ [Window](#window) - Inserts temporal overlaps a the end of fixed length temporal windows  
  ⚬ [Unwindow](#unwindow) - Auto removes or crossfades overlaps added by `window()`  
<sub>     *Padding/Trimming*</sub>  
  ⚬ [Extend](#extend) - Extends a clip with various temporal padding modes  
  ⚬ [Trim](#trim) - Auto trims extended clip from `extend()`  
<sub>     *Other*</sub>  
  ⚬ [Crossfade](#crossfade) - Crossfades between two clips
* [Usage Examples](#usage-examples)
* [Mode Explanations](#mode-explanations)

<br />

## Requirements
* [fillborders](https://github.com/dubhater/vapoursynth-fillborders)
* [cv_inpaint](https://github.com/dnjulek/VapourSynth-cv_inpaint)
* [autocrop](https://github.com/Irrational-Encoding-Wizardry/vapoursynth-autocrop) *(optional, only for autofill)*
* [akarin](https://github.com/Jaded-Encoding-Thaumaturgy/akarin-vapoursynth-plugin) *(optional, only for markdups/skipdups)*
* [vship](https://github.com/Line-fr/Vship) *(optional, only for markdups/skipdups, requires v4.0.0 or newer)*

## Setup
Put the `vs_tiletools.py` file into your vapoursynth scripts folder.  
Or install via pip: `pip install -U git+https://github.com/pifroggi/vs_tiletools.git`

<br />

## Spatial Functions
* ### Tile
  Splits each frame into tiles of fixed dimensions to reduce resource requirements. Outputs a clip with all tiles in order. All filters applied to the tiled clip should be spatial only.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tile(clip, width=256, height=256, overlap=16, padding="mirror")
  ```
  
  __*`clip`*__  
  Clip to tile. Any format.
  
  __*`width`*, *`height`*__  
  Tile size of a single tile in pixel.

  __*`overlap`*__  
  Overlap from one tile to the next. When overlap is increased the tile size is not altered, so the amount of tiles per frame increases. Can be a single value or a pair for horizontal and vertical `[16, 32]`.

  __*`padding`*__  
  How to handle tiles that are smaller than tile size. These can be padded with modes `mirror`, `wrap`, `repeat`, `fillmargins`, `telea`, `ns`, `fsr`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, or just discarded with `discard`.

---

* ### Untile
  Automatically reassembles a clip tiled with `tile()`, even if tiles were since resized. [Example](#reduce-vram-usage-on-heavy-ai-models-via-tiling)
  ```python
  import vs_tiletools
  clip = vs_tiletools.untile(clip, fade=False) # automatic
  clip = vs_tiletools.untile(clip, fade=False, full_width=None, full_height=None, overlap=None) # manual
  ```

  __*`clip`*__  
  Tiled clip. Any format.
  
  __*`fade`*__  
  If fade is True, the overlap will be used to feather/blend between the tiles to remove visible seams.  
  If fade is False, the overlap will be cropped.

  __*`full_width`*, *`full_height`*, *`overlap`* (optional)__  
  You can also enter untile parameters manually. Needed is the full assembled frame dimensions and the overlap between tiles. In manual mode you have to account for resized or discarded tiles yourself.  
  Tip: If tiles were discarded, the full_width/full_height are now smaller and a multiple of the original tile size.  
  Tip: If tiles were resized 2x, simply double all values.

---

* ### Pad
  Pads a clip with various padding modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.pad(clip, left=0, right=0, top=0, bottom=0, mode="mirror")
  ```
  
  __*`clip`*__  
  Clip to be padded. Any format.
  
  __*`left`*, *`right`*, *`top`*, *`bottom`*__  
  Padding amount in pixel.
  
  __*`mode`*__  
  Padding mode can be `mirror`, `wrap`, `repeat`, `fillmargins`, `telea`, `ns`, `fsr`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

---

* ### Mod
  Pads or crops a clip so width and height are multiples of the given modulus.
  ```python
  import vs_tiletools
  clip = vs_tiletools.mod(clip, modulus=64, mode="mirror")
  ```
  
  __*`clip`*__  
  Source clip. Any format.
  
  __*`modulus`*__  
  Dimensions will be a multiple of this value. Can be a single value, or a pair for width and height `[64, 32]`.
  
  __*`mode`*__  
  Mode to reach the next upper multiple via padding can be `mirror`, `wrap`, `repeat`, `fillmargins`, `telea`, `ns`, `fsr`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, or `discard` to crop to the next lower multiple.

---

* ### Crop
  Automatically crops padding added by `pad()` or `mod()`, even if the clip was since resized. [Example1](#fix-issues-around-borders-with-some-filters-via-padding) [Example2](#fix-filters-that-require-the-input-to-be-divisible-by-a-factor-via-padding)
  ```python
  import vs_tiletools
  clip = vs_tiletools.crop(clip) # automatic
  clip = vs_tiletools.crop(clip, left=0, right=0, top=0, bottom=0) # manual
  ```
  
  __*`clip`*__  
  Padded clip. Any format.
  
  __*`left`*, *`right`*, *`top`*, *`bottom`* (optional)__  
  Optionally you can also enter crop values manually.

---

* ### Croprandom
  Crops to the given dimensions, but randomly repositions the crop window each frame.
  ```python
  import vs_tiletools
  clip = vs_tiletools.croprandom(clip, width=256, height=256, seed=0)
  ```
  
  __*`clip`*__  
  Clip to be cropped. Any format.

  __*`width`*, *`height`*__  
  Cropped window dimensions in pixels.

  __*`seed`*__  
  Seed used for deterministic crop randomization.

---

* ### Fill
  Fills the borders of a clip with various filling modes. Basically padding, but inwards.
  ```python
  import vs_tiletools
  clip = vs_tiletools.fill(clip, left=0, right=0, top=0, bottom=0, mode="mirror")
  ```
  
  __*`clip`*__  
  Clip to be filled. Any format.
  
  __*`left`*, *`right`*, *`top`*, *`bottom`*__  
  Fill amount in pixel.
  
  __*`mode`*__  
  Filling mode can be `mirror`, `wrap`, `repeat`, `fillmargins`, `telea`, `ns`, `fsr`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

---

* ### Autofill
  Detects uniform colored borders (like letterboxes/pillarboxes) and fills them with various filling modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.autofill(clip, left=0, right=0, top=0, bottom=0, offset=0, color=[16,128,128], tol=16, fill="mirror")
  ```
  
  __*`clip`*__  
  Source clip. Only YUV formats are supported.

  __*`left`*, *`right`*, *`top`*, *`bottom`*__  
  Maximum border fill amount in pixels.

  __*`offset`*__  
  Offsets the detected fill area by an extra amount in pixels. Useful if the borders are slightly blurry.  
  Does not offset sides that have detected 0 pixels.

  __*`color`*__  
  Source clip border color in 8-bit scale `[16, 128, 128]`.

  __*`tol`*__  
  Tolerance to account for fluctuations in border color. Can be a single value or a list `[16, 16, 16]`.

  __*`fill`*__  
  Filling mode can be `mirror`, `repeat`, `fillmargins`, `telea`, `ns`, `fsr`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

---

* ### Inpaint
  Inpaints areas in a clip based on a mask with various inpainting modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.inpaint(clip, mask, mode="telea")
  ```
  
  __*`clip`*__  
  Clip to be inpainted. Any format.
  
  __*`mask`*__  
  Black and white mask clip where white means inpainting. Can be a single frame long, or longer and different each frame. If too short, the last frame will be looped. Can be any format and doesn't have to match the base clip.
  
  __*`mode`*__  
  Inpainting mode can be `telea`, `ns`, `fsr` or `shiftmap`.

---

<br />

## Temporal Functions
* ### Markdups
  Marks up to 5 consecutive frames as duplicates if they are near identical, which can later be skipped using `skipdups()`. [Example](#skip-heavy-filters-on-duplicate-frames-most-useful-for-anime)
  ```python
  import vs_tiletools
  clip = vs_tiletools.markdups(clip, thresh=0.3)
  ```
  
  __*`clip`*__  
  Clip were duplicates should be marked. Any format.
  
  __*`thresh`*__  
  Similarity threshold. If the difference between two consecutive frames is lower than this value, the frame is marked as a duplicate. If the value is 0, only 100% identical frames will be marked as duplicate. Keep it a little above 0 due to noise and compression. The default worked nicely for me on anime.

---

* ### Skipdups
  Skips processing of up to 5 consecutive duplicate frames marked by `markdups()`. That means the marked frames will copy one of the previous 5 frames instead of submitting the current frame for processing. This speeds up heavy filters sandwiched inbetween `markdups()` and `skipdups()`. [Example](#skip-heavy-filters-on-duplicate-frames-most-useful-for-anime)

  Keep in mind that if you use a heavy spatial filter, followed by a temporal filter, both inside of the sandwich, the speedup will be negated, because the temporal filter will request the marked frames anyway. For this reason, it is recommended to use temporal filters outside the sandwich.
  ```python
  import vs_tiletools
  clip = vs_tiletools.skipdups(clip, debug=False) # automatic
  clip = vs_tiletools.skipdups(clip, prop_src=None, debug=False) # manual
  ```

  __*`clip`*__  
  Clip with marked duplicates. Any format.
  
  __*`prop_src`* (optional)__  
  Frame properties source clip. This should be detected automatically. But if the frame props of the first clip got lost, you can set it here manually. It should be the clip directly returned by `markdups()`. 

  __*`debug`*__  
  Overlays the frame number of the selected frame and the difference value to the previous frame onto the output. This is useful to finetune the sensitivity threshold in `markdups()`.

---

* ### Window
  Inserts temporal overlaps at the end of each temporal window into the clip. That means a window with `length=20` and `overlap=5` will produce a clip with this frame pattern: `0–19`, `15–34`, `30–49`, and so on. In combination with the unwindow function, the overlap can then be used to crossfade between windows and eliminate sudden jumps/hitches that can occur on window based functions like [vs_undistort](https://github.com/pifroggi/vs_undistort). [Example](#fix-jumpshitches-on-temporal-windowchunk-based-filters-via-crossfading)
  ```python
  import vs_tiletools
  clip = vs_tiletools.window(clip, length=20, overlap=5, padding="mirror")
  ```
  
  __*`clip`*__  
  Clip that should be windowed. Any format.
  
  __*`length`*__  
  Temporal window length.

  __*`overlap`*__  
  Overlap from one window to the next. When overlap is increased, the temporal window length is not altered, so the total amount of windows per clip increases.

  __*`padding`*__  
  How to handle the last window of the clip if it is smaller than length. It can be padded with modes `mirror`, `loop`, `repeat`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, discarded with `discard`, or left as is with `None`.
  
---

* ### Unwindow
  Automatically removes the overlap added by `window()` and optionally uses it to crossfade between windows. [Example](#fix-jumpshitches-on-temporal-windowchunk-based-filters-via-crossfading)
  ```python
  import vs_tiletools
  clip = vs_tiletools.unwindow(clip, fade=False) # automatic
  clip = vs_tiletools.unwindow(clip, fade=False, full_length=None, window_length=None, overlap=None) # manual
  ```

  __*`clip`*__  
  Windowed clip. Any format.
  
  __*`fade`*__  
  If fade is True, the overlap will be used to crossfade between the windows.  
  If fade is False, the overlap will be trimmed.

  __*`full_length`*, *`window_length`*, *`overlap`* (optional)__  
  You can also enter unwindow parameters manually. Needed is the full clip length, window length and the overlap between windows. In manual mode you have to account for a discarded window yourself.  
  Tip: If the last window was discarded, the full_length is now smaller and a multiple of window_length.  
  Tip: If the windowed clip was interpolated to 2x, simply double all values.

---

* ### Extend
  Extends (temporally pads) a clip using various padding modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.extend(clip, start=0, end=0, length=None, mode="mirror")
  ```
  
  __*`clip`*__  
  Clip to extend. Any format.

  __*`start`*, *`end`*__  
  Number of frames to add at the start and/or end. Mutually exclusive with `length`. 

  __*`length`*__  
  Extends clip to exactly this many frames. Mutually exclusive with `start`/`end`. 

  __*`mode`*__  
  Padding mode can be `mirror`, `loop`, `repeat`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

---

* ### Trim
  Automatically trims temporal padding added by `extend()`. [Example](#filters-with-multiple-input-clips-often-require-both-to-have-the-same-length)
  ```python
  import vs_tiletools
  clip = vs_tiletools.trim(clip) # automatic
  clip = vs_tiletools.trim(clip, start=0, end=0, length=None) # manual
  ```
  
  __*`clip`*__  
  Temporally padded clip. Any format.
  
  __*`start`*, *`end`* (optional)__  
  Optional manual number of frames to remove from start and/or end. End is mutually exclusive with `length`.

  __*`length`* (optional)__  
  Optional manual trim to exactly this many frames, starting from start. Mutually exclusive with `end`.

---

* ### Crossfade
  Crossfades between two clips.
  ```python
  import vs_tiletools
  clip = vs_tiletools.crossfade(clipa, clipb, length=10)
  ```
  
  __*`clipa`*, *`clipb`*__  
  Input clips to crossfade. Any format, as long as they match.

  __*`length`*__  
  Length of the crossfade. For example `length=10` will fade the last 10 frames of clipa into the first 10 frames of clipb.

---

<br />

## Usage Examples
Examples of how the paired functions are used together.

* #### Reduce VRAM usage on heavy AI models via tiling.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tile(clip, width=256, height=256, overlap=16) # splits frames into 256x256 tiles with an overlap of 16
  clip = core.trt.Model(clip, engine_path="2x_heavy_model.engine")  # heavy AI upscale model
  clip = vs_tiletools.untile(clip, fade=True)                       # reassembles the tiles and uses the overlap to feather
  ```

* #### Fix issues around borders with some filters via padding.
  ```python
  import vs_tiletools
  clip = vs_tiletools.pad(clip, left=8, right=8, top=8, bottom=8) # pad 8 pixels on all sides
  clip = core.trt.Model(clip, engine_path="model.engine")         # AI model with issues near borders
  clip = vs_tiletools.crop(clip)                                  # automatically crop the padding
  ```

* #### Fix filters that require the input to be divisible by a factor via padding.
  ```python
  import vs_tiletools
  clip = vs_tiletools.mod(clip, modulus=16)                      # pad to make width and height divisible by 16
  clip = core.trt.Model(clip, engine_path="2x_DAT_model.engine") # DAT and HAT based AI models have this constraint
  clip = vs_tiletools.crop(clip)                                 # automatically crop the padding
  ```

* #### Skip heavy filters on duplicate frames. Most useful for anime.
  ```python
  import vs_tiletools
  clip = vs_tiletools.markdups(clip, thresh=0.3)                   # marks duplicate frames with a low threshhold
  clip = core.trt.Model(clip, engine_path="2x_heavy_model.engine") # heavy AI upscale model
  clip = vs_tiletools.skipdups(clip)                               # skips duplicates and replaces them with a previous frame
  ```

* #### Fix jumps/hitches on temporal window/chunk based filters via crossfading.
  ```python
  import vs_tiletools
  clip = vs_tiletools.window(clip, length=10, overlap=4) # creates a temporal overlap of 4 frames
  clip = vs_undistort.tensorrt(clip, temp_window=10)     # filter has 10 input frames and 10 output frames
  clip = vs_tiletools.unwindow(clip, fade=True)          # uses the overlap to fade between temporal windows
  ```

* #### Filters with two input clips often require both to have the same length.
  ```python
  import vs_tiletools
  clip = vs_tiletools.extend(clip, length=clip2.num_frames) # pad clip to the length of clip2
  clip = some.multi_input_filter(clip, clip2)               # filter with multiple inputs
  clip = vs_tiletools.trim(clip)                            # automatically trims the added frames
  ```

<br />

## Mode Explanations
Full explanations for all padding/filling/inpainting modes.

* __Spatial__
  * `mirror` Reflects the image into the padded region.
  * `wrap` Wraps the image around to create a periodic tiling.
  * `repeat` Repeats the outermost pixel row/column.
  * `fillmargins` Similar to repeat, but the top/bottom padding gets more blurry the further away it is.
  * `telea` Telea's algorithm. Similar to fillmargins, but all padding gets more blurry the further away it is.
  * `ns` Navier-Stokes algorithm. Similar to telea, but less blurry.
  * `fsr` Frequency Selective Reconstructiom algorithm. Better at keeping patterns/textures, but is slow.
  * `shiftmap` Shifts part of the existing image to fill the holes. Only for inpainting.
  * `black` Solid black padding.
  * `[128, 128, 128]` Solid custom color padding. 8-bit values per plane in the clip’s color family.

* __Temporal__
  * `mirror` Reverses the clip at the start/end.
  * `loop` Loops the clip to start over.
  * `repeat` Repeats the first/last frame.
  * `black` Appends solid black frames.
  * `[128, 128, 128]` Appends frames in a solid custom color. 8-bit values per plane in the clip’s color family.

<br />

> [!NOTE]
> The padded/filled/inpainted regions may be generated at a lower bit depth due to plugin limitations (16-bit for fillborders, 8-bit for cv_inpaint), then upsampled and merged onto the original high depth frames. This should usually not be an issue.
>
> Padding mode `fixborders` is additionally supported in all functions, if the [fillborders](https://github.com/dubhater/vapoursynth-fillborders) plugin is compiled from source. See [this](https://github.com/dubhater/vapoursynth-fillborders/issues/7) issue. Modes `fillmargins` and `fixborders`, which are partially broken when using the fillborders plugin directly, are also fixed here.

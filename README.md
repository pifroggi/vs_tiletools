


















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
  * [Tile](#tile) - Splits each frame into tiles of fixed dimensions
  * [Untile](#untile) - Auto reassembles tiles from `tile()`, even if resized
  * [Pad](#pad) - Pads a clip with various padding modes
  * [Crop](#crop) - Auto crops padded clip from `pad()` or `mod()`, even if resized
  * [Mod](#mod) - Pads or crops a clip so width and height are multiples of the given modulus
  * [Autofill](#autofill) - Auto detects borders and fills them with various fill modes
* [Temporal Functions](#temporal-functions)
  * [Window](#window) - Inserts temporal overlaps a the end of fixed length temporal windows
  * [Unwindow](#unwindow) - Auto removes or crossfades overlaps added by `window()`
  * [TPad](#tpad) - Temporally pads with various padding modes
  * [Trim](#trim) - Auto trims temporally padded clip from `tpad()`
  * [Crossfade](#crossfade) - Crossfades between two clips

<br />

## Requirements
* [fillborders](https://github.com/dubhater/vapoursynth-fillborders)
* [autocrop](https://github.com/Irrational-Encoding-Wizardry/vapoursynth-autocrop) *(optional, only for autofill)*

## Setup
Put the `vs_tiletools.py` file into your vapoursynth scripts folder.  
Or install via pip: `pip install -U git+https://github.com/pifroggi/vs_tiletools.git`

<br />

## Spatial Functions
* ### Tile
  Splits each frame into tiles of fixed dimensions to reduce resource requirements. Outputs a clip with all tiles in order.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tile(clip, width=256, height=256, overlap=16, padding="mirror")
  ```
  
  __*`clip`*__  
  Clip to tile. Any format.
  
  __*`width`*, *`height`*__  
  Tile size of a single tile in pixel.

  __*`overlap`*__  
  Overlap from one tile to the next. When overlap is increased the tile size is not altered, so the amount of tiles per frame increases.  
  Can be a single value or a pair for horizontal and vertical `[16, 16]`.

  __*`padding`*__  
  How to handle tiles that are smaller than tile size. These can be padded with modes `mirror`, `repeat`, `fillmargins`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, or just discarded with `discard`.

<br />

* ### Untile
  Automatically reassembles a clip tiled with `tile()`, even if tiles were since resized.
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
  __Tip:__ If tiles were discarded, the full_width/full_height are now smaller and a multiple of the original tile size.  
  __Tip:__ If tiles were resized 2x, simply double all values.

<br />

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
  Padding mode can be `mirror`, `repeat`, `fillmargins`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

<br />

* ### Crop
  Automatically crops padding added by `pad()` or `mod()`, even if the clip was since resized.
  ```python
  import vs_tiletools
  clip = vs_tiletools.crop(clip) # automatic
  clip = vs_tiletools.crop(clip, left=0, right=0, top=0, bottom=0) # manual
  ```
  
  __*`clip`*__  
  Padded clip. Any format.
  
  __*`left`*, *`right`*, *`top`*, *`bottom`* (optional)__  
  Optionally you can also enter crop values manually.

<br />

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
  Mode to reach the next upper multiple via padding can be `mirror`, `repeat`, `fillmargins`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, or `discard` to crop to the next lower multiple.

<br />

* ### Autofill
  Detects uniform colored borders (like letterboxes/pillarboxes) and fills them with various filling modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.autofill(clip, left=0, right=0, top=0, bottom=0, offset=0, color=[16,128,128], tol=16, tol_c=None, fill="mirror")
  ```
  
  __*`clip`*__  
  Source clip. Only YUV formats are supported.

  __*`left`*, *`right`*, *`top`*, *`bottom`*__  
  Maximum border fill amount in pixels.

  __*`offset`*__  
  Offsets the detected fill area by an extra amount in pixels. Useful if the borders are slightly blurry.  
  Does not offset sides that have detected 0 pixels.

  __*`color`*__  
  Source clip border color in 8-bit scale `[16,128,128]`.

  __*`tol`*__, (*`tol_c`*)  
  Tolerance to account for fluctuations in border color.  
  Tolerance chroma is optional and defaults to `tol` if not set. 

  __*`fill`*__  
  Filling mode can be `mirror`, `repeat`, `fillmargins`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

<br />

## Temporal Functions
* ### Window
  Inserts temporal overlaps at the end of each temporal window into the clip. That means a window with `length=20` and `overlap=5` will produce a clip with this frame pattern: `0–19`, `15–34`, `30–49`, and so on. In combination with the unwindow function, the overlap can then be used to crossfade between windows and eliminate sudden jumps/hitches that can occur on window based functions like [vs_undistort](https://github.com/pifroggi/vs_undistort).
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
  How to handle the last window of the clip if it is smaller than length. It can be padded with modes `mirror`, `repeat`, `black`, a custom color in 8-bit scale `[128, 128, 128]`, discarded with `discard`, or left as is with `None`.
  
<br />

* ### Unwindow
  Automatically removes the overlap added by `window()` and optionally uses it to crossfade between windows.
  ```python
  import vs_tiletools
  clip = vs_tiletools.unwindow(clip, fade=False) # automatic
  clip = vs_tiletools.unwindow(clip, fade=False, full_length=None, window_length=None, overlap=None) # manual
  ```

  __*`clip`*__  
  Windowed clip. Any format.
  
  __*`fade`*__  
  If fade is True, the overlap will be used to crossfade between the windows to remove jumps/hitches.  
  If fade is False, the overlap will be trimmed.

  __*`full_length`*, *`window_length`*, *`overlap`* (optional)__  
  You can also enter unwindow parameters manually. Needed is the full clip length, window length and the overlap between windows. In manual mode you have to account for a discarded window yourself.  
  __Tip:__ If the last window was discarded, the full_length is now smaller and a multiple of window_length.  
  __Tip:__ If the windowed clip was interpolated to 2x, simply double all values.

<br />

* ### TPad
  Temporally pads (extends) a clip using various padding modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tpad(clip, start=0, end=0, length=None, mode="mirror")
  ```
  
  __*`clip`*__  
  Clip to pad. Any format.

  __*`start`*, *`end`*__  
  Number of frames to add at the start and/or end. Mutually exclusive with `length`. 

  __*`length`*__  
  Pads clip to exactly this many frames. Mutually exclusive with `start`/`end`. 

  __*`mode`*__  
  Padding mode can be `mirror`, `repeat`, `black`, or a custom color in 8-bit scale `[128, 128, 128]`.

<br />

* ### Trim
  Automatically trims temporal padding added by tpad().
  ```python
  import vs_tiletools
  clip = vs_tiletools.trim(clip) # automatic
  clip = vs_tiletools.trim(clip, start=0, end=0, length=None) # manual
  ```
  
  __*`clip`*__  
  Temporally padded clip. Any format.
  
  __*`start`*, *`end`* (optional)__  
  Optional manual number of frames to remove from start and/or end. Mutually exclusive with `length`.

  __*`length`* (optional)__  
  Optional manual trim to exactly this many frames. Mutually exclusive with `start`/`end`.

<br />

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

<br />

<br />

> [!NOTE]
> Padding mode "fixborders" is additionally supported in all functions, if the [fillborders](https://github.com/dubhater/vapoursynth-fillborders) plugin is compiled from source. See [this](https://github.com/dubhater/vapoursynth-fillborders/issues/7) issue.

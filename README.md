



























# Collection of Tiling Related Functions for VapourSynth

<br />

## Table of Contents
* [Requirements](#requirements)
* [Setup](#setup)
* [Spatial Functions](#spatial-functions)
  * [Pad](#pad) - Pad a clip with various padding modes
  * [Crop](#crop) - Auto crops a padded clip, even if it has been resized
  * [Tile](#tile) - Split clip into tiles of fixed dimensions
  * [Untile](#untile) - Auto reassemble a tiled clip, even if the tiles have been resized
  * [Autofill](#autofill) - Detect solid borders and fill them with various fill modes
* [Temporal Functions](#temporal-functions)
  * [TPad](#tpad) - Temporally pads with various padding modes
  * [Window](#window) - Turns clip into segments of fixed temporal window length with overlap
  * [Unwindow](#unwindow) - Removes added overlaps from a windowed clip or crossfades them
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
  Padding mode can be `mirror`, `repeat`, `fillmargins`, `black`, or a custom color in 8 bit scale `[128, 128, 128]`.

<br />

* ### Crop
  Automatically crops what was added with the pad function, even if the clip was since resized.
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

* ### Tile
  Splits a clip into tiles of fixed dimensions to reduce resource requirements. Outputs a clip with all tiles in order.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tile(clip, width=256, height=256, overlap=16, padding="mirror")
  ```
  
  __*`clip`*__  
  Clip that should be tiled. Any format.
  
  __*`width`*, *`height`*__  
  Tile size of a single tile in pixel.

  __*`overlap`*__  
  Overlap from one tile to the next. When overlap is increased the tile size is not altered, so the amount of tiles per frame increases.  
  Can be a single value or a list for vertical and horizontal `[16, 16]`.

  __*`padding`*__  
  How to handle tiles that are smaller than tile size. These can be padded with modes `mirror`, `repeat`, `fillmargins`, `black`, a custom color in 8 bit scale `[128, 128, 128]`, or just discarded with `discard`.

<br />

* ### Untile
  Automatically reassembles a tiled clip, even if the tiles were since resized.
  ```python
  import vs_tiletools
  clip = vs_tiletools.untile(clip, fade=False) # automatic
  clip = vs_tiletools.untile(clip, fade=False, full_width=None, full_height=None, overlap=None) # manual
  ```

  __*`clip`*__  
  Tiled clip. Any format.
  
  __*`fade`*__  
  If fade is True, the overlap will be used to feather/blend between the tiles.  
  If fade is False, the overlap will be cropped.

  __*`full_width`*, *`full_height`*, *`overlap`* (optional)__  
  You can also enter untile parameters manually. Needed is the full assembled frame dimensions and the overlap between tiles. In manual mode you have to account for resized or discarded tiles yourself.  
  __Tip:__ If tiles were discarded, the full_width/full_height are now smaller and a multiple of the original tile size.  
  __Tip:__ If tiles were resized 2x, simply double all values.

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
  Maximum border fill thickness in pixels.

  __*`offset`*__  
  Offsets the detected fill area by an extra amount in pixels. Useful if the borders are slightly blurry.  
  Does not offset sides that have detected 0 pixels.

  __*`color`*__  
  Source clip border color in 8-bit range `[16,128,128]`.

  __*`tol`*__, (__*`tol_c`*__)  
  Tolerance and optionally tolerance chroma to set how much the source clip border color is fluctuating.  
  Tolerance chroma is optional and is the same as tolerance if not set. 

  __*`fill`*__  
  Filling mode can be `mirror`, `repeat`, `fillmargins`, `black`, or a custom color in 8 bit scale `[128, 128, 128]`.

<br />

## Temporal Functions
* ### Window
  Segments a clip into temporal windows with a fixed length and adds overlap on the tail end of each window. In combination with the unwindow function, the overlap can then be used to crossfade between windows and eliminate sudden jumps/seams that can occur on window based functions like [vs_undistort](https://github.com/pifroggi/vs_undistort).
  ```python
  import vs_tiletools
  clip = window(clip, length=100, overlap=10, padding="mirror")
  ```
  
  __*`clip`*__  
  Clip that should be windowed. Any format.
  
  __*`length`*__  
  Temporal window length.

  __*`overlap`*__  
  Overlap from one window to the next. When overlap is increased the temporal window length is not altered, so the total amount of windows per clip increases.

  __*`padding`*__  
  How to handle windows that are smaller than length. These can be padded with modes `mirror`, `repeat`, `black`, a custom color in 8 bit scale `[128, 128, 128]`, discarded with `discard`, or left as is with `None`.
  
<br />

* ### Unwindow
  Automatically removes the overlap from a windowed clip and optionally uses it to crossfade between windows.
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
  __Tip:__ If the last window was discarded, the full_length is now smaller and a multiple of window_length.  
  __Tip:__ If the windowed clip was interpolated to 2x, simply double all values.

<br />

* ### TPad
  Temporally pads (extends) a clip by appending frames using various padding modes.
  ```python
  import vs_tiletools
  clip = vs_tiletools.tpad(clip, length=1000, mode="mirror", relative=False)
  ```
  
  __*`clip`*__  
  Clip to extend. Any format.

  __*`length`*__  
  Total length of padded clip, or number of frames to add, depending on `relative`.

  __*`mode`*__  
  Padding mode can be `mirror`, `repeat`, `black`, or a custom color in 8 bit scale `[128, 128, 128]`.

  __*`relative`*__  
  If relative is False `length` is the total length of the output clip.  
  If relative is True, `length` is the number of frames to append at the end.

<br />

* ### Crossfade
  Crossfades between two clips (without FrameEval/ModifyFrame).
  ```python
  import vs_tiletools
  clip = vs_tiletools.crossfade(clipa, clipb, length=10)
  ```
  
  __*`clipa`*, *`clipb`*__  
  Input clips to crossfade. Any format, as long as they match.

  __*`length`*__  
  Length of the crossfade. For example `length=10` will fade the last 10 frames of clipa into the first 10 frames of clipb.

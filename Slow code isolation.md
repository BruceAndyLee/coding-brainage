setup: refterm - a terminal app on Windows that talks to kernel and commands from subshell. The pipe between that and the terminal app is the bottle-neck. Whatever infrastructure is set up each time there is a command run - is a bottle-neck.
- Know your theoretical limit (imposed by other nodes in the call-tree).
- don't do anything to the data, that's dumped into your code from kernel/cmd. Hit the max bandwidth by not subscribing to the buffer change and instantiating structures of the terminal-app. Only change the portion of the buffer to be shown on the screen.

btw the hierarchy of encodings:
- ascii (1 byte per symbol)
- unicode (4 bytes per symbol)
- UTF-8 (varied character-length)
- UTF-16

use rolling(circular) buffer


forwards renderer: process meshes, rasterize, draw pixels that are touched by the triangles
backwards renderer: iterate over pixels, lookup what shapes do they touch, rasterize (has the potential to be much more efficient - not always true due to what primitives does the GPU have)

when a project does not get restricted to a single layer of text - the renderer should be picked by timing things

?pipe to the graphics card
?unicode parsing and glyph generation (d\[irect\]write, uniscribe) - these are the system's utilities to render characters on the screen, they're a bottle-neck too, wrap them in some code to call them as rarely as possible(?)

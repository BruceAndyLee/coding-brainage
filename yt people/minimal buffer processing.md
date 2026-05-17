There are escape sequences that control visuals (new-line, caret-return, change-color, make the cursor blink)

Scrollback low-down buffer is potentially up to 16-20 GB.

Over the scrollback buffer we decide to create a line-buffer where each line is represented by the start and the end pointers.

Line buffer is also a rolling buffer.

there are things in POSIX, like "actually print every character from the kernel/command response", that are pessimizing for performance, so have a check-box for them in the settings.

![[minimal buffer processing 2026-01-11 13.55.22.excalidraw|1200]]

All the bytes that come in the `buffer` are indexed in absolute offsets, so whenever the line buffer points to a byte that was overridden in the buffer by the ROLLING of it, we know not to display that line.

The pre-pass parser chunks up the buffer and takes care of the control characters (breaking lines, wrapping lines)

We don't render every byte of the buffer, only the tail-end.

Character -> glyph transition in terms of how many characters on the grid are occupied:
- many to one - for some hiroglyph
- one to many - for some multi-character ligature-glyph that is defined by that original character
- many to many - natural continuation of the previous one (more realistic one btw)

Know also how to:
- make hashes
- make a LRU cache
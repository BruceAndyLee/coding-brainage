Learn to think about stuff that is pessimized and see how to make it less so. Learn things every day to get better and better at not giving the CPU work that is not necessary.

One of the complications handled: support for ligatures - a drop-in replacement for a specific combination of characters that looks a bit different (trimmed and more readable) then the tacked-on version of all the constituent parts.

So like a common case would be 5 bytes that produce three glyphs without a ligature or a single glyph if a ligature is applied but three-characters long (1x3 on the grid).

Glyphs are stored in a texture, pointers(address of the glyph on the texture) to those glyphs are stored in a hash-table.

Here's what that entails for the cache look-ups:
- feed arbitrary number of bytes into the hasher (i.e. for unicode-described glyph)
- get a hash whose length is a maximum of 12 bytes (an euristic value, imposed by the longest ever supported set of glyphs per one unit of the language being displayed)
	- clarification: this means that each cache hit may produce multiple glyph pointers ()
- be sad, because the table is bloated: less than 0.1% of this is MORE than will ever be necessary to render any glyph. So come up with a new hash.

It seems like the solution would be to take a hash and then concatenate to it the INDEX of the ligatured glyph to be produced onto it.
This is sort of a second level indexing that lets the hash-table know that we need one of the few versions of the same glyph-shape to be drawn on the screen.

(How the heck does it resolve the problem of there being NO ligature, hence, when a thing to be drawn is drawn with set of non-adorned glyphs?)


| bytes       | hashed          | w/ ligature index | hashed info  | comment                                                                  |
| ----------- | --------------- | ----------------- | ------------ | ------------------------------------------------------------------------ |
| B1 B2 B3 B4 | hB1 hB2 hB3 hB4 | hB1 hB2 hB3 hB4 0 | (2, 2, 1, 3) | the coordinate and the width + height in the texture with all the glyphs |
??
- what is a glyph tho? is it really something that can be put onto a texture? it not a set of curves defined mathematically?

![[Glyph hash table 2026-01-11 21.43.30.excalidraw]]
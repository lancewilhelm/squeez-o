# squeez-o
OLED bitmap compression for deflating QMK-compatible graphics

## How does it work?
This tool compresses a 128x32 pixel bitmap generated with [img2cpp](https://javl.github.io/image2cpp/) by marking null bytes (0x0) and non-null bytes using a single bit in a smaller byte array. It works best when the image has lots of whitespace. The compressed images stop being smaller than the input images when the overhead (64 bytes for each 512 byte input) is greater than the savings. Most bitmap images break even or better, but it's worth checking the statistics after running the tool to make sure the savings are worth the effort.

## What's the point?
One of the most popular [NIBBLE keymaps](https://github.com/qmk/qmk_firmware/tree/master/keyboards/nullbitsco/nibble/keymaps/oled_bongocat) was too large to enable VIA on. The frames had a lot of whitespace and compressing them seemed like a promising idea. The firmware-side decompression algorithm is very simple, takes up only a few bytes of flash, and fast enough that it doesn't noticeably slow down the matrix scan. After compressing the 7 original frames, there is over 1kB of free flash remaining for other cool firmware features, or the addition of extra frames for more complex animations. 

## Usage
Generate a 128x32 byte array using img2cpp (black background, plain bytes, Vertical - 1 bit per pixel) and save only the raw bytes to `input_bytes.tmp`. Optionally, users may save the raw bytes to another file of their choosing and use the `-f` flag followed by their file name when running the program. Run
`python3 squeez-o.py` and copy the output `block_list` and `block_maps` into your keymap or keyboard code. For projects with multiple animations, do this for each frame.

Within the keyboard firmware, use the example decompression algorithm in `decompress.c` to decompress the `block_list` and `block_maps` and write to the OLED display. 

Running on the included `input_bytes.tmp` file yields this output: 
```c
static const char PROGMEM block_x_map[] = {
0x00, 0x00, 0x00, 0x00, 0x00, 0xc0, 0xff, 0xff, 0xff, 0xff, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xff, 0xff, 0xff, 0xff, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0xf0, 0xff, 0xff, 0xff, 0xff, 0x07, 0x00, 0x00, 0x00, 0x00, 0x00,
0x00, 0x00, 0x00, 0x00, 0x00, 0xf8, 0xff, 0xff, 0xff, 0xff, 0x0f, 0x00, 0x00, 0x00, 0x00, 0x00
};

static const char PROGMEM block_x_list[] = {
0x80, 0xe0, 0xe0, 0xf0, 0xf8, 0xf8, 0xfc, 0xfc, 0xfc, 0xfc, 0xfc, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe,
0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe, 0xfe,
0xfe, 0xfe, 0xfe, 0xfe, 0xfc, 0xf8, 0xf8, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0x0f,
0x07, 0x07, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03,
0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x03, 0x01, 0x1f, 0xbf,
0x9f, 0xdf, 0xdf, 0xdf, 0xdf, 0xdf, 0xdf, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0,
0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0, 0xc0,
0xc0, 0xc0, 0xc0, 0x80, 0x80, 0x0c, 0x3f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f,
0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f,
0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x7f, 0x3f
};
Input was 512 bytes, deflated block list is now 158 bytes + 64 bytes overhead
Space savings: 56.64%
```

Which can be used within QMK by calling `oled_write_compressed_P(block_x_map, block_x_list);`

## Examples
For more info, take a look at the [bongo cat keymap]().

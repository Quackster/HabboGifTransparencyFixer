# Habbo Gif Transparency Fixer

A short little tool that changes the transparency layer of a .gif file to have the RGB values of 255,255,255 instead of 0,0,0 to fix transparency in Habbo Shockwave client versions. No tools expose this easily through any existing image editing API, so we edit the bytes of the .gif itself.

## Converting Colour Table

The .gif must have a single frame and a "global" colour table (a single colour table used for all .gif frames, as opposed to local) to parse the bytes of the .gif correctly.

To convert it to have a global colour table, this code can be used (using SixLabors.ImageSharp library):

```c

Image<Rgba32> img = Image.Load<Rgba32>(inputFile);
img.Save(outputFile, new SixLabors.ImageSharp.Formats.Gif.GifEncoder
{
    ColorTableMode = SixLabors.ImageSharp.Formats.Gif.GifColorTableMode.Global,
    Quantizer = new OctreeQuantizer(new QuantizerOptions
    {
        Dither = null
    })
});

```

# Sample Code

This is the code itself that changes the transparency layer of a .gif.

```c
public void SetTransparencyLayer(string inputFile, string outputFile, byte r, byte g, byte b)
{
    byte[] gifBytes = File.ReadAllBytes(inputFile);

    // The color table starts at byte 10 and is 3 * (2^(N+1)) bytes long
    int colorTableStart = 10;
    int colorTableLength = 3 * (1 << (gifBytes[10] & 0x07) + 1);

    // Extract the color table bytes
    byte[] colorTableBytes = new byte[colorTableLength];
    Array.Copy(gifBytes, colorTableStart, colorTableBytes, 0, colorTableLength);

    int colorTableColors = (colorTableBytes[1] * 3); // Find all the colours used in the colour table (multiplied by 3 for each RGB value)
    int transparencyStartIndex = (colorTableColors + 3); // Find the transparency layer (the last colour found in the colour table)

    colorTableBytes[transparencyStartIndex + 1] = r;
    colorTableBytes[transparencyStartIndex + 2] = g;
    colorTableBytes[transparencyStartIndex + 3] = b;
    
    Array.Copy(colorTableBytes, 0, gifBytes, colorTableStart, colorTableLength);
    File.WriteAllBytes(outputFile, gifBytes);
    
```

```

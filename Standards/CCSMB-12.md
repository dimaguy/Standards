# *CCSMB 12:* 3D JSON Array Format

*Author: Dimaguy*

*Version: 1.0.0*

*Last revised: 2024-04-02*

The words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are defined in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).
This RFC introduces the 3D JSON Array Format, used for printing 3D Blocks in ComputerCraft using the mod [sc-peripherals](https://github.com/SwitchCraftCC/sc-peripherals)

## Quick information

| Information |                           |
| ----------- | ------------------------- |
| Version     | 1.0.0                     |
| Type        | Model Array file format   |
| MIME        | `model/3dja`              |
| Extensions  | `.3dja`                   |


## Glossary
This document introduces a few words to help distinguish roles in file handling:
- Writer - Software or Library that writes 3dja files by method of merging single block files or out of other representations of block data.
- Reader - Software or Library that reads 3dja files and either previews the data to the user or passes it to a printer and/or builder.
- Printer - Machine/Software combo that turns 3dja files or reader instructions into printing instructions and executes them.
- Builder - Machine/Software combo that turns 3dja files or reader instructions into building instructions and executes them.

## Technical details
The file MUST be composed of a single JSON object [^1] with the following ruleset:
- The writer SHOULD specify global non-default (according to 3dj) common settings for the printed 3D blocks in the global namespace of the JSON object.
- The writer MAY define a dictionary of presets (which are tint/texture combinations), if this isn't done every block MUST use their own texture and tint for each shape.
- The writer MUST define an array of block models, following the specifications of the [3dj](https://docs.sc3.io/features/sc-peripherals.html#_3dj-format) format with the addition of a coordinates array (if the user foregoes presets each block model object should be compatible with a 3dj file, the coordinates field SHOULD be disregarded by 3dj-only readers).
- Each block model MAY override a global setting by redefining it on it's context.
- Size is 1-based indexed, ordered by X, Y and Z each being unsigned integers. It represents number of blocks in each axis
- Coordinates are 0-based indexed, ordered by x, y and z each being signed integers. It represents position of blocks in each axis
- The reader MAY provide builtin presets, which MUST be overriden if the namespace coincides with one in the file.
- The reader MAY provide means of swapping in external presets profiles for recoloring or retexturing, which MUST override part or the entirety of the namespace in file.
- The reader SHOULD facilitate the printing of a single block of the entire set by providing facilities to print a selected coordinate, a selected label of a block or it's index in the file.
- The reader SHOULD warn about files whose coordinates don't match boundaries in set in size, if one is set. A builder MUST fail and refuse the file in such circumstances.
- The reader SHOULD reject blocks that try to refer to unknown presets.
- A printer SHOULD produce the block in the amount of the sets of coordinates of the block.
- A fallback "amount" parameter MAY be accepted per block for mass production purposes, such parameter SHOULD match the amount of coordinates if those are set, but if it doesn't a printer MAY use it to override the amount of sets of coordinates.
- Writers SHOULD avoid using "amount" parameter if there's coordinates facilities available.
The following represents the file format in a more visual aspect:
```jsonc
{
  "label": "BuildName: $BLABEL",  // (RECOMMENDED) The name scheme of the build, the string SHOULD be processed to make use of the Label Variables that are set below, if blank the result SHALL be "blocks[i].label"
  "tooltip": "...",       // (OPTIONAL) The description of the build, shared between all blocks unless overridden
  "size": [1,1,1],        // (RECOMMENDED) Size in number of blocks required for the build, not needed for printing but is an indicator for an automated builder
  "isButton": false,      // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "collideWhenOn": true,  // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "collideWhenOff": true, // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "lightLevel": 0,        // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "redstoneLevel": 0,     // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "lightWhenOn": false,   // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "lightWhenOff": false,  // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "seatPos": undefined,   // (OPTIONAL) Global setting for all blocks (see 3dj format)
  "presets": {            // (RECOMMENDED) Dictionary of presets, used to define the texture and tint of a shape for compresion and ease of recoloring
    "presetname": {
      "texture": "",      // (OPTIONAL) Texture of the preset (see 3dj format)
      "tint": "FFFFFF"    // (OPTIONAL) Tint of the preset (see 3dj format)
    },
  },
  "blocks": [
    {
      "label": "...",                // (OPTIONAL) The name scheme of a block, if empty the "blocks[i].label" SHALL be "Untitled", the string MUST be processed to make use of the Label Variables below
      "overrideGlobalLabel": false,  // (OPTIONAL) If true, label of this block SHALL override the global label entirely
      "tooltip": "...",              // (OPTIONAL) The description of the block, overrides the tooltip of the build
      "coordinates": [[0, 0, 0]],    // Coordinates of the block (if not, automated building programs SHALL fail and a printer SHOULD only print the block once)
      "amount": 64,                  // SHOULD NOT be used if coordinates is set, but if it is, SHOULD match their amount of sets. Can be used for mass production.
      "lightLevel": 2,               // (OPTIONAL) The light level of the block, overrides the global setting
      "redstoneLevel": 14,           // (OPTIONAL) The redstone level of the block, overrides the global setting
      //Any global setting can be overridden here
      "shapesOff": [
        { "bounds": [0, 0, 0, 16, 16, 16], "preset": "presetname"},           //A shape can use a preset, which MUST be defined in presets dictionary
        { "bounds": [0, 0, 0, 16, 16, 16], "texture": "", "tint": "FFFFFF" }, //Or define its own texture and tint
      ],
      "shapesOn": [
        { /* ... */ }
      ]
    }
  ]
}
```
The preset dictionary and array of coordinates per block's goal is to achieve some sort of compression within the file.
This format provides enough data for an automated building system.

### Label Variables
This list shows placeholder variables for text substitutions that SHOULD be made by Readers using string manipulation at runtime.
Readers SHOULD refuse to process a label containing itself as a variable, but MAY let it pass as a raw
- $BLABEL - Block Label
- $BX - Block X Coordinate
- $BY - Block Y Coordinate
- $BZ - Block Z Coordinate
- $BS - Block Serial Number (reset in every printing session)
This list MAY be extended by implementations.

[^1]: [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259), "The JavaScript Object Notation (JSON) Data Interchange Format")

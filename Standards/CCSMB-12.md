# *CCSMB 12:* 3D JSON Array Format

*Author: Dimaguy*

*Version: 1.0.0*

*Last revised: 2024-03-30*

This RFC introduces the 3D JSON Array Format, used for printing 3D Blocks in ComputerCraft using the mod [sc-peripherals](https://github.com/SwitchCraftCC/sc-peripherals)

## Quick information

| Information |                           |
| ----------- | ------------------------- |
| Version     | 1.0.0                     |
| Type        | Model Array file format   |
| MIME        | `model/3dja`              |
| Extensions  | `.3dja`                   |

## Technical details

The file MUST be composed of a single JSON object with this basic ruleset:
- The writer SHOULD specify global settings for the printed 3D blocks in the global namespace of the object, if a global setting isn't set each block model MUST have it set individually.
- The writer MAY define a dictionary of presets (which are tint/texture combinations), if this isn't done every block MUST use their own texture and tint.
- The writer MUST define an array of block models, following the specifications of the [3dj](https://docs.sc3.io/features/sc-peripherals.html#_3dj-format) format with the addition of a coordinates array (if the user foregoes presets each block model object should be compatible with a 3dj file if the global settings are inserted, the coordinates field WILL be disregarded).
- Each block model MAY override a global setting.
The following represents the file format in a more visual aspect
```jsonc
{
  "label": "...",         // (RECOMMENDED) The name of the build, if specified end result of a block in the build SHALL be "BuildName: blocks[i].label", if not end result SHALL be blocks[i].label
  "tooltip": "...",       // (OPTIONAL) The description of the build, shared between all blocks unless overridden
  "size": [1,1,1],        // (RECOMMENDED) Size in number of blocks required for the build, not needed for printing but is an indicator for an automated builder
  "isButton": false,      // Global setting for all blocks, if not assigned every block must have it's own setting
  "collideWhenOn": true,  // Global setting for all blocks, if not assigned every block must have it's own setting
  "collideWhenOff": true, // Global setting for all blocks, if not assigned every block must have it's own setting
  "lightLevel": 0,        // Global setting for all blocks, if not assigned every block must have it's own setting
  "redstoneLevel": 0,     // Global setting for all blocks, if not assigned every block must have it's own setting
  "presets": {            // (OPTIONAL) Dictionary of presets, used to define the texture and tint of a shape for compresion
    "presetname": {
      "texture": "",      // (REQUIRED) Texture of the preset
      "tint": "FFFFFF"    // (REQUIRED) Tint of the preset
    },
  },
  "blocks": [
    {
      "label": "...",             // (OPTIONAL) The name of the block, if empty SHALL use "(blocks[i].x,blocks[i].y,blocks[i].z)" as blocks[i].label
      "tooltip": "...",           // (OPTIONAL) The description of the block, overrides the tooltip of the build
      "coordinates": [[0, 0, 0]], // 0,0,0 represents the origin of the build, used for automated builders and desambiguation, each set of coordinates SHOULD be globally unique (if not, automated building programs SHALL fail), the block MUST be printed for the amount of coordinates in this array
      "lightLevel": 2,            // (OPTIONAL if assigned globally) The light level of the block, overrides the global setting
      "redstoneLevel": 14,        // (OPTIONAL if assigned globally) The redstone level of the block, overrides the global setting
      //Any global setting can be overridden here
      "shapesOff": [
        { "bounds": [0, 0, 0, 16, 16, 16], "preset": "presetname"},           //A shape can use a preset, which MUST be in presets dictionary
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

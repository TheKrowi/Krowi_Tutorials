A comprehensive library for creating and displaying tutorial pages with images, text, and navigation controls to guide users through curated content in World of Warcraft addons.

## Features

### Tutorial System (`Krowi_Tutorials-3.0`)
- **Page-Based Tutorials**: Create multi-page tutorials with images and text
- **Image Support**: Display images with customizable sizes and margins
- **Flexible Layout**: Customize text margins, frame titles, and page content
- **Navigation Controls**: Built-in page navigation for easy browsing
- **State Persistence**: Remembers the last viewed page using SavedVariables
- **Easy Integration**: Simple API for quick implementation
- **LibStub Support**: Standard LibStub library structure for dependency management

## Usage Examples

### Basic Tutorial Setup
```lua
local tutorials = LibStub("Krowi_Tutorials-3.0")
local featuresTutorial = tutorials:New("FeaturesTutorial", SavedData)

-- Configure the tutorial
featuresTutorial:SetFrameTitle("Your Title")

-- Create pages
local pages = {}
tinsert(pages, {
    Image = "Interface\\AddOns\\YourAddon\\Images\\Feature1",
    ImageSize = {930, 158},
    SubTitle = "Feature 1 Overview",
    Text = "Detailed description of your first feature..."
})
tinsert(pages, {
    Image = "Interface\\AddOns\\YourAddon\\Images\\Feature2",
    ImageSize = {930, 158},
    SubTitle = "Feature 2 Overview",
    Text = "Detailed description of your second feature..."
})

-- Apply pages and display
featuresTutorial:SetPages(pages)
featuresTutorial:SetImageMargin(10)
featuresTutorial:SetTextMargin({10, 0, 10, 20})
featuresTutorial:ShowTutorial(1) -- Show page 1
```

## API Reference

### Tutorial Arguments
When creating a new tutorial with `New(key, savedVariables, icon, font)`:

| Argument | Required | Description |
|----------|----------|-------------|
| `key` | Yes | The variable name used to save the last viewed page index |
| `savedVariables` | Yes | The table used in the .toc file for SavedVariables |
| `icon` | Optional | Path to image (tga or blp). Default is '?'-mark |
| `font` | Optional | Font name to be used. Default is GameFontHighlight |

### Tutorial Functions

| Function | Description |
|----------|-------------|
| `New(key, savedVariables, icon, font)` | Creates a new tutorial frame |
| `SetPages(pages)` | Sets the pages the tutorial contains |
| `AddPage(page)` | Adds a single page to the tutorial |
| `Reset()` | Resets the tutorial to start on the first page |
| `ShowTutorial(pageIndex)` | Shows the tutorial on the specified page |
| `SetFrameTitle(title)` | Sets the title at the top of the tutorial window (can be overwritten by page's 'Title') |
| `SetFrameWidth(width)` | Sets the internal frame width (without borders). Default is 532. Can be overwritten by page's 'Width' |
| `SetCloseButtonHook(func)` | Hooks a function to the closing of the tutorial |
| `SetImageSize(width, height)` | Set the image size for all pages |
| `SetImageMargin(margin)` | Set the image margins for all pages |
| `SetTextSize(width, height)` | Set the text size for all pages |
| `SetTextMargin(margin)` | Set the text margins for all pages |

### Page Arguments
All page arguments are optional and can also be declared at the tutorial level:

| Argument | Type | Default | Description |
|----------|------|---------|-------------|
| `Point` | string | "CENTER" | Used in Tutorial:SetPoint |
| `RelativeFrame` | frame | UIParent | Used in Tutorial:SetPoint |
| `RelativePoint` | string | "CENTER" | Used in Tutorial:SetPoint |
| `OffsetX` | number | 0 | Used in Tutorial:SetPoint |
| `OffsetY` | number | 0 | Used in Tutorial:SetPoint |
| `Title` | string | "Tutorial" | Overwrites SetFrameTitle for the specific page |
| `Width` | number | 50 | Overwrites SetFrameWidth for the specific page |
| `Image` | string | none | Image path (tga or blp) |
| `ImageSize` | table | {0, 0} | {Width, Height} of the image |
| `ImageTexCoord` | table | {0, 1, 0, 1} | {Left, Right, Top, Bottom}. See [SetTexCoord](https://wowpedia.fandom.com/wiki/API_Texture_SetTexCoord) |
| `ImageMargin` | table | {0, 0, 0, 0} | {Left, Top, Right, Bottom} margins of the image |
| `Text` | string | none | Text string displayed under Image if provided |
| `TextSize` | table | {0, 0} | {Width, Height} of the text area. Auto Width/Height if default |
| `TextMargin` | table | {0, 0, 0, 0} | {Left, Top, Right, Bottom} margins of the text area |
| `Shine` | frame | none | The frame to anchor the flashing "look at me!" glow |
| `ShineAll` | number | none | Set ShineTop, ShineBottom, ShineLeft, ShineRight to the same value |
| `ShineWidth` | number | none | Set ShineLeft, ShineRight to the same value |
| `ShineHeight` | number | none | Set ShineTop, ShineBottom to the same value |
| `ShineTop` | number | none | Top offset for shine effect |
| `ShineBottom` | number | none | Bottom offset for shine effect |
| `ShineLeft` | number | none | Left offset for shine effect |
| `ShineRight` | number | none | Right offset for shine effect |

## Use Cases
- Feature walkthroughs for new users
- Update announcements with visual guides
- Configuration help screens
- Achievement or quest guides
- Step-by-step instructions
- Changelog displays with images
- Any scenario requiring visual documentation

## Requirements
- LibStub
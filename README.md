# 2D Character Starter

Codex skill for generating and editing 2D playable character assets with explicit modes.

## Usage

Reference the skill as `$2DCS` or `$2dcs`, then choose a mode:

```text
$2DCS s      one image -> light/medium/heavy simplification
$2DCS ct     two images -> image 1 character + image 2 target style
$2DCS p      two or more images -> image 1 appearance + each later image as a strict pose reference
$2DCS sq     one isolated character -> walk/run/jump/attack/dash single frames
$2DCS cs     one concept -> 4:3 green-screen idle character at low/medium/high complexity
$2DCS h      show usage
```

All image-producing modes output 4:3 images on a plain chroma green screen background.

## Install

Copy this repository folder into your Codex skills directory as `2dcs`, or install it with your usual Codex skill installation workflow.

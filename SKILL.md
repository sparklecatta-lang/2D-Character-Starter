---
name: 2dcs
description: "Generate/edit 2D playable game character images after an explicit no-hyphen mode. All outputs use 3:2 chroma green screen. Modes: s/simplify = three simplification levels; ct/concept target = image1 character + image2 target style; p/pose = image1 appearance + skeleton-only pose refs, output one refined strict pose-transfer image per pose reference; cs/concept simplify = one concept to right-facing green-screen idle at low/medium/high complexity; sq/sequence = one single-character image to walk/run/jump/attack/dash single frames; h/help = usage only. cs is canonical without a space, but c s is accepted as the same mode. Any image-producing mode except h/help accepts a numeric suffix for independent subagent samples. If no mode, ask for s, ct, p, cs, sq, or h. Triggers include simplify lines, simplify silhouette, concept target mode, concept simplify mode, style reference, pose reference, green screen playable character, animation frames, standing idle character, 设定图转画风, 姿势迁移, 简化线稿, 绿幕背景, 可游玩角色."
---

# 2D Character Starter

## Purpose

Use this skill only after the user provides an explicit no-hyphen mode. Mode matching is case-insensitive. `s` / `simplify` produces three simplified-linework 2D playable character images from one reference image. `ct` / `concept target` produces one concept-to-target-style transfer from two reference images. `p` / `pose` uses image 1 as the locked appearance reference and every remaining image as a separate strict skeleton-only pose reference, producing one refined pose-transfer image per pose reference. `cs` / `concept simplify` produces three 3:2 chroma-green, right-facing standing-idle playable character images from one character concept at low/medium/high visual complexity. `cs` is the canonical spelling, but `c s` is accepted as the same mode. `sq` / `sequence` produces five 3:2 chroma-green single action frames from one isolated single-character image: walk, run, jump, attack, and dash. Any image-producing mode except `h` / `help` may take a numeric suffix for independent subagent samples. `h` / `help` shows usage and never processes images. All image-producing modes output 3:2 images on a plain chroma green screen background. Treat prompt rewriting as an internal step for image generation unless the user explicitly asks for prompt text only.

The goal is not to remove character identity; it is to preserve the readable macro silhouette, pose, costume, colors, gameplay role, and visible self-shading while reducing line noise, tiny internal details, noisy outer-contour fragments, and overly complex shadow shapes. Do not preserve the reference edge contour exactly.

## Default Behavior

- First choose the mode from the user's message after lowercasing the mode tokens:
  - standalone `h` or standalone `help`: show Help Mode usage and stop. Ignore numeric suffixes after Help Mode.
  - standalone `cs`, standalone `c` plus standalone `s`, or standalone `concept` plus standalone `simplify`, without `p`: use Concept Simplify Mode with one character concept image or source prompt.
  - standalone `sq` or standalone `sequence`: use Sequence Mode with one isolated single-character image.
  - standalone `s` or standalone `simplify`: use Simplification Mode with one input image.
  - standalone `ct`, or standalone `concept` plus standalone `target`: use Concept Target Mode with two input images.
  - standalone `p` or standalone `pose`: use Pose Mode with two or more input images.
- Treat mode markers as standalone tokens only. Do not match letters inside ordinary words.
- Do not require or suggest hyphens for modes.
- After resolving any image-producing mode except `h` / `help`, also parse one optional standalone positive integer immediately after the mode tokens, such as `s 5`, `ct 5`, `p 5`, `sq 5`, `cs 2`, or `c s 2`. Treat this integer only as an orchestration count: the current coordinator should run that many independent subagents with the same source images and the same base mode request. Do not treat the integer as an image prompt, style strength, seed, simplification strength, complexity level, sequence frame count, or pose-control strength.
- The orchestration count must stay within the current Codex subagent tool's allowed range. If the tool does not expose a concrete limit, accept counts from 1 to 5 by default and ask the user before exceeding 5. If subagents are unavailable and the requested count is greater than 1, say so plainly instead of simulating separate runs in one agent.
- The orchestration count multiplies the selected mode's normal outputs. For example, `s 5` means five subagents each emit the three simplification outputs; `ct 5` means five subagents each emit one concept target output; `p 5` means five subagents each emit one refined pose-transfer output per pose reference after image 1; `sq 5` means five subagents each emit five sequence-frame outputs; `cs 5` means five subagents each emit the three concept simplify outputs.
- When using an orchestration count, give every subagent the same input order and the same minimal mode text without the count, for example pass only `$2DCS s`, `$2DCS ct`, `$2DCS p`, `$2DCS sq`, or `$2DCS cs` plus the required source image(s). Do not add per-subagent variants, extra hand notes, prompt tweaks, or different priorities. The point of the count is independent repeated sampling, not different instructions.
- When using an orchestration count, isolate that request's outputs in a fresh run-group directory before launching subagents. Use run ids only for file organization, such as `run01_` or a `run01/` subfolder, so agents do not overwrite each other and old outputs do not appear mixed with the current request. Do not use run ids as creative prompt content.
- Numeric suffixes do not apply to Help Mode. `h 5` or `help 5` must only show usage and must not start subagents, generate images, or write prompts.
- If Help Mode is present together with other markers, show usage and do not process images.
- If the user includes `s` together with `p`, includes `ct` together with `p`, includes `cs` together with `p`, includes `c s p`, or includes `sq` / `sequence` together with any other image-producing mode token, stop and ask them to choose a compatible mode. `cs` and `c s` are valid, but `s p`, `ct p`, `cs p`, `c s p`, `sq s`, `sq ct`, `sq cs`, and `sq p` are not. Do not process images, infer priority, write prompts, or call image generation.
- If the user invokes this skill without a recognized mode, stop and ask them to choose a mode. Do not process images, infer intent, write prompts, or call image generation.
- Natural-language intent such as `简化`, `三档简化`, `概念`, `设定图转画风`, `姿势`, `姿势迁移`, `序列`, `动作帧`, `绿幕`, or `待机` is not enough by itself. Treat it as missing mode unless it also includes standalone `s`, `simplify`, `ct`, standalone `concept` plus standalone `target`, `p`, `pose`, `sq`, `sequence`, `cs`, standalone `c` plus standalone `s`, standalone `concept` plus standalone `simplify`, `h`, or `help`.
- Use this reminder when mode is missing: `请选择模式：s 简化三档，ct 概念图转目标画风，p 图1外形+图2及后续姿势，cs 设定图生成绿幕待机三档复杂度，sq 单角色图生成走/跑/跳/攻击/dash 单帧，或 h 查看用法。`
- Use this reminder when modes conflict: `检测到不兼容模式。请选择：s 简化三档，ct 概念图转目标画风，p 姿势迁移，cs 生成绿幕待机三档复杂度，或 sq 单角色动作单帧序列。输入 h 查看用法。`
- All image-producing modes must output a 3:2 image with a plain chroma green screen background. Do not preserve, inherit, or copy background colors from source or reference images.
- In Pose Mode, treat image 1 as the locked appearance reference and each image after image 1 as a separate strict neutralized pose skeleton/control reference. Pose references provide geometry only, never appearance, style, costume, hair, weapon, palette, or rendering cues.
- In Pose Mode, generate one refined final pose-transfer image for each pose reference after image 1. Do not create pose-lock tiers, artistic variants, prompt-direction variants, candidate pools, or AI-side winner selection. If the user explicitly uses a numeric suffix such as `p 5`, each subagent still returns one independent result per pose reference and the coordinator must not pick winners automatically.
- In Simplification Mode, if the user provides an image and invokes this skill, directly generate/edit three output images using the best available image generation tool.
- In Simplification Mode, generate three clearly different simplification levels: `轻度`, `中度`, and `重度`.
- Keep the same character identity, pose, 3:2 framing, macro silhouette, and chroma green background across all three outputs so the user can compare simplification level directly.
- Make the outer contour visibly simpler from `轻度` to `重度`; do not only clean up line clarity.
- Keep visible character self-shadows in all three outputs by default. Simplify shadow shapes across levels, but do not let `重度` become completely shadowless unless the user explicitly asks for a flat icon.
- Use a plain chroma green screen background on a 3:2 canvas.
- If the image tool can reliably bind each returned output to its intended semantic level, request all three levels in one batch and then save them with the required filenames. If display or return order is unreliable, run sequential image generations/edits from the same reference image in semantic order.
- Do not create `中度` or `重度` by locally posterizing, quantizing, reducing colors, thresholding, vectorizing, edge-filtering, or otherwise processing a previously generated `轻度` image. Each level must be generated/edited from the original reference image and its own level-specific prompt.
- Do not stop after returning a rewritten prompt unless the user explicitly asks for prompts only.
- Use the attached image as the reference for identity, pose, proportions, palette, and macro silhouette.
- If no source image or prompt is available, ask for the character image or source prompt.
- In Simplification Mode, keep the final reply short: display the three generated images directly in numeric order with Markdown image embeds, without standalone links or file lists.
- Do not place text labels inside the generated images.
- In Concept Simplify Mode, generate three clearly different complexity levels: `低复杂度`, `中复杂度`, and `高复杂度`.
- In Concept Simplify Mode, keep all outputs 3:2, chroma green background, right-facing, standing idle, and comparable in framing.
- In Concept Simplify Mode, keep the final reply short: display the three generated images directly in numeric order with Markdown image embeds, without standalone links or file lists.
- In Sequence Mode, require one isolated single-character image as the source. Generate five independent single-frame action images: `walk`, `run`, `jump`, `attack`, and `dash`. Do not create an animation sheet, timeline, contact sheet, or multi-frame collage.

## Output Naming And Ordering

Do not rely on the visual order of image tiles returned by an image generation tool. Some UIs may display generated images out of semantic order. The file name and final reply image order are authoritative.

- For any mode with multiple outputs, prefer sequential generation in the intended semantic order when practical.
- If a backend returns several images at once, inspect or track which output belongs to which semantic level, then rename or save files with the correct numeric prefix before replying.
- If you cannot confidently identify which generated image belongs to which level, regenerate the affected outputs sequentially instead of guessing.
- Use zero-padded numeric prefixes so file managers and galleries sort correctly.
- Final replies for image-producing modes must display every generated image directly in numeric order using Markdown image syntax with absolute local paths, for example `![01_p_refined_pose](C:/path/to/run01/01_p_refined_pose.png)`.
- Do not output standalone Markdown links such as `[name](path)`, plain text file paths, file attachment lists, file-type metadata, output-directory summaries, or collapsed "show additional files" style lists unless the user explicitly asks for paths.
- Use only brief text separators when they materially help readability, such as `run01`, `run02`, or source ids for orchestration-count runs. Otherwise, let the image embeds be the result.
- Put any necessary labels in the final reply text, not inside the images.

Default filenames:

- Simplification Mode: `01_s_light.png`, `02_s_medium.png`, `03_s_heavy.png`.
- Concept Simplify Mode: `01_cs_low_complexity.png`, `02_cs_medium_complexity.png`, `03_cs_high_complexity.png`.
- Concept Target Mode: `01_ct_concept_target.png`.
- Pose Mode: `01_p_refined_pose.png` for one pose reference; for multiple pose references, `01_p_pose01_refined.png`, `02_p_pose02_refined.png`, and so on in input order.
- Sequence Mode: `01_sq_walk.png`, `02_sq_run.png`, `03_sq_jump.png`, `04_sq_attack.png`, `05_sq_dash.png`.

When processing multiple source images, batch tests, multi-pose Pose Mode runs, or orchestration-count runs, create a fresh output group directory for that request. Inside it, prefix filenames with a source, pair, pose, or run id when needed, then keep the same semantic suffix, for example `source01_01_s_light.png`, `ct_pair01_01_ct_concept_target.png`, `02_p_pose02_refined.png`, or `run01_02_p_pose02_refined.png`.

## Help Mode

Trigger Help Mode when the user includes standalone `h` or standalone `help`, case-insensitive.

Help Mode output:

- Show usage and stop. Do not process images, write prompts, or call image generation.
- Use this concise usage text:

```text
$2DCS s      一张图 -> 轻度/中度/重度三档简化
$2DCS ct     两张图 -> 图1角色设定 + 图2目标画风
$2DCS p      两张或更多图 -> 图1角色设定 + 后续每张不同姿势，每个姿势输出一张精细姿势迁移
$2DCS sq     一张单角色图 -> 走路/跑步/跳跃/攻击/dash 五张单帧
$2DCS cs     一张设定图 -> 3:2绿幕右朝向待机三档复杂度
$2DCS h      查看用法

数字后缀适用于除 h/help 以外的所有产图模式：
$2DCS s 5    打开 5 个 subagent，各自执行同样的 s 模式
$2DCS ct 5   打开 5 个 subagent，各自执行同样的 ct 模式
$2DCS p 5    打开 5 个 subagent，各自执行同样的 p 模式，每个pose各出一张，不自动挑选
$2DCS sq 5   打开 5 个 subagent，各自执行同样的 sq 模式
$2DCS cs 5   打开 5 个 subagent，各自执行同样的 cs 模式
$2DCS h 5    仍然只显示帮助，不启动 subagent

所有输出默认 3:2 绿幕背景。模式大小写不敏感，S/CT/P/SQ/CS/H 也可以。cs 推荐连写，写成 c s 也可以。
```

## Simplification Mode

Trigger Simplification Mode only when the user includes standalone `s` or standalone `simplify`, case-insensitive.

Simplification Mode expects one image:

1. Source image: a 2D playable character or character concept to simplify.

Simplification Mode output:

- Generate three output images labeled `轻度`, `中度`, and `重度`.
- Save them with these exact filenames and display them in this exact semantic order: `01_s_light.png` (`轻度`), `02_s_medium.png` (`中度`), `03_s_heavy.png` (`重度`).
- Preserve the source character identity, pose, proportions, palette, major costume logic, weapon/tool, and macro silhouette.
- Simplify linework, outer contour, detail density, and shadow complexity progressively across the three levels.
- Make all three outputs 3:2 aspect ratio with a plain chroma green screen background.
- Do not derive `中度` or `重度` from another generated level through local filters.

## Concept Simplify Mode

Trigger Concept Simplify Mode when the user includes standalone `cs`, standalone `c` plus standalone `s`, or standalone `concept` plus standalone `simplify`, case-insensitive, and `p` is not present.

Concept Simplify Mode expects one source:

1. Character concept/source design image or source prompt. Use this for character identity and design.

Concept Simplify Mode output:

- Generate three independent output images labeled `低复杂度`, `中复杂度`, and `高复杂度`.
- Save them with these exact filenames and display them in this exact semantic order: `01_cs_low_complexity.png` (`低复杂度`), `02_cs_medium_complexity.png` (`中复杂度`), `03_cs_high_complexity.png` (`高复杂度`).
- Make all three outputs 3:2 aspect ratio with a plain chroma green screen background.
- Redraw the character as a single 2D playable game character, full body when possible, facing right, in a standing idle state.
- Preserve the source character identity, age/gender presentation, face type, hair, costume logic, weapon/tool concept, palette relationships, role, and important macro silhouette.
- If the source is a concept sheet with multiple views, use the main/front full-body design as the primary source. Use side/back views only to clarify costume structure. Ignore text, logos, callouts, weapon-only panels, and layout marks unless the user says otherwise.
- Keep the pose stable across all three outputs: right-facing profile or three-quarter-right orientation, upright standing idle, feet planted, neutral ready posture, no attack pose, no walking/running frame, no dramatic action.
- Vary visual complexity, not character identity, pose, background, or framing.
- Generate each complexity level from the original source concept, not from another generated level.
- Do not use local posterization, palette reduction, thresholding, vectorization, or other filters to create lower-complexity variants.
- Keep the result clean and usable as a game asset: one character only, centered, no text, no concept-sheet layout, no labels, no logos, no extra views, no scenery.

Use these three complexity levels:

1. `低复杂度`: Ultra-minimal gameplay sprite. Use the fewest readable shapes: bold silhouette, 3 to 5 large color regions, almost no interior lines, no folds, no seams, no small ornaments, no hair strands, no material texture, and at most one tiny identity mark if essential. Use flat colors plus one very simple shadow shape only if needed for readability.
2. `中复杂度`: Minimal playable asset. Still much simpler than ordinary concept art. Use large merged costume panels, very few interior lines, simplified hair masses, simplified weapon/tool silhouette, no tiny decorations, no fabric wrinkles, and 1 simple cel-shadow tone with 1 to 2 broad shadow masses.
3. `高复杂度`: Simple playable asset; this is the old low-complexity target. Use large readable shapes, clean contour, few interior lines, simple flat colors, one simple cel-shadow tone, and only the ornaments or fabric folds needed to preserve character identity. Do not exceed this into rich concept-art detail.

Concept Simplify Mode internal instruction:

```text
Use the source character concept as the identity and design reference. Generate three independent 3:2 images of the same single 2D playable game character on a plain chroma green screen background: ultra-minimal low complexity, minimal medium complexity, and simple high complexity. In all three, redraw the character full-body when possible, facing right, standing idle, feet planted, neutral ready posture, centered, with consistent framing. Preserve the character's identity, face type, hair, costume logic, palette relationships, weapon/tool concept, role, and important macro silhouette. Vary only visual complexity: low is extremely reduced with 3 to 5 big color regions and almost no interior lines; medium is still minimal with large merged costume panels and 1 to 2 broad shadow masses; high equals the previous low-complexity target with clean contour, few interior lines, simple flat colors, and one simple cel-shadow tone. Do not create rich concept-art detail in any level. No action pose, no walking/running, no extra views, no text, no labels, no logos, no scenery, no concept-sheet layout.
```

## Sequence Mode

Trigger Sequence Mode only when the user includes standalone `sq` or standalone `sequence`, case-insensitive, and does not include any other image-producing mode marker.

Sequence Mode expects one image:

1. Source image: one isolated single-character image. Use it for character identity, proportions, silhouette, costume design, palette, material finish, weapon/tool concept, and rendering style.

Sequence Mode output:

- Generate five independent single-frame action images, not an animation sheet: `walk`, `run`, `jump`, `attack`, and `dash`.
- Save them with these exact filenames and display them in this exact semantic order: `01_sq_walk.png`, `02_sq_run.png`, `03_sq_jump.png`, `04_sq_attack.png`, and `05_sq_dash.png`.
- Make all five outputs 3:2 aspect ratio with a plain chroma green screen background.
- Keep all five frames comparable: same character identity, same outfit logic, same palette, same right-facing orientation by default, similar camera angle, similar character scale, and similar canvas placement.
- If the source image contains a weapon or tool, keep that weapon/tool concept and use it only when it naturally belongs to the action, especially `attack` and optionally `dash`. If the source has no weapon, make `attack` an unarmed strike, kick, defensive shove, magic gesture, or body action consistent with the character.
- If the source image is a concept sheet, multi-view sheet, group image, UI screenshot, or image with several characters, stop and ask for one isolated single-character image or an explicit crop. Do not extract a character from a sheet by default in Sequence Mode.
- Do not add action labels, frame numbers, onion-skin ghosts, extra poses, motion trails, speed lines, impact effects, scenery, ground planes, UI, or cast shadows on the background unless the user explicitly asks.

Use these five action frame definitions:

1. `walk`: A readable mid-step walking frame. One foot forward, one foot back, mild torso shift, arms or props moving naturally, calm playable-game locomotion.
2. `run`: A stronger locomotion frame than walk. Longer stride, forward lean, clearer arm swing or prop counterbalance, lifted trailing foot, still a single readable game-frame pose.
3. `jump`: A single airborne or takeoff frame. Feet off the ground or one foot just leaving, knees bent or extended according to character type, arms/props balanced, silhouette clearly separated from idle.
4. `attack`: A single attack anticipation or strike frame. Use the character's weapon/tool if present; otherwise use a body, kick, punch, spell, defensive shove, or gesture attack. Keep the pose readable and do not add hit effects or enemies.
5. `dash`: A quick burst movement frame. Use low center of mass, forward lean, compressed or stretched silhouette, trailing cloth/hair/limbs, and strong directional intent. Do not turn it into a run duplicate.

Sequence Mode internal instruction:

```text
Use the source image as the locked identity and design reference for one single playable game character. Generate five independent 3:2 single-frame action images on a plain chroma green screen background: walk, run, jump, attack, and dash. Preserve the source character's identity, proportions, face/head type, costume design, palette, material finish, weapon/tool concept, and rendering style across all five frames. Make all frames right-facing by default, full body when possible, centered with comparable scale and camera angle. Vary only the action pose. Do not make a sprite sheet, contact sheet, animation timeline, multi-pose collage, or labeled diagram. No text, labels, frame numbers, logos, scenery, ground planes, motion trails, speed lines, impact effects, enemies, extra characters, or extra views.
```

## Concept Target Mode

Trigger Concept Target Mode only when the user includes standalone `ct`, or standalone `concept` plus standalone `target`, case-insensitive, and does not also include Pose Mode markers.

Concept Target Mode expects two images:

1. First image: character concept/source design. Use this for character identity and design.
2. Second image: target style reference. Use this for rendering style, finish level, shape language, detail density, proportion language, edge treatment, value grouping, and framing tendency. Do not use it for output background color.

Concept Target Mode output:

- Generate one final character image by default, not three simplification levels.
- Save the output as `01_ct_concept_target.png` unless a batch/source prefix is needed, then display it directly as a Markdown image.
- Preserve the first image's character identity anchors: species/body type, face type, age/gender presentation when readable, important hair/head features, costume logic, weapon/tool if present, palette relationships, and the few silhouette features that make the character recognizable.
- If the first image is a concept sheet with multiple views, use the main/front full-body design as the primary source and ignore text, logos, callouts, weapon-only panels, side/back views, and layout marks unless the user says otherwise.
- Treat the first image as identity, not style. Do not preserve the first image's rendering style, line density, anatomy detail, texture, finish level, or exact proportions when they conflict with the target style.
- Apply the second image's art direction strongly: rendering finish, line/edge treatment, silhouette grammar, lighting softness, shadow style, material simplification, value grouping, detail density, proportion language, and full-body presentation.
- If the target style is highly simplified, compact, icon-like, silhouette-heavy, proportion-shifted, abstract, dominant-value-shape based, low-detail, or otherwise far from the source style, push the output far enough that it clearly belongs to the target style family. Do not stop at a conservative source-style redraw with thicker outlines.
- Do not copy the second image's exact pose in Concept Target Mode. Preserve image 1's broad stance/attitude when possible, but adapt pose and proportions into a natural target-style idle/readable game-asset pose when the target style requires a different body scale, compactness, or abstraction level.
- Output a 3:2 image with a plain chroma green screen background. Do not copy the second image's background color.
- Do not copy the second image's character identity, face/head design, distinctive facial coverings, clothing design, weapon/tool, symbols/markings, pose-specific props, or story details unless the user explicitly asks.
- Keep the result as a clean single-character game asset, full body when possible, centered, with no text, no concept-sheet layout, no labels, no logos, and no side-view panels.
- Before generation, silently build two separate checklists:
  1. Source identity anchors from image 1: the minimum features needed for the character to remain recognizable.
  2. Target style anchors from image 2: shape language, silhouette/value grouping, line density, detail budget, proportion language, material simplification, shadow style, and framing.
- In the final prompt, name the concrete target style anchors observed in image 2. Use task-specific observations, not canned examples. Do not use generic phrases like `polished rendering style` by themselves.
- In the final prompt, explicitly forbid the main observed source-style failure modes, such as retaining the source rendering style, source anatomy/proportion language, source detail density, source texture habits, or exact source linework, when those conflict with the target style.
- If the first result mostly looks like image 1 with superficial target colors, outlines, or background, reject it and regenerate with stronger target-style transfer. This is a CT failure, not an acceptable variant.

Concept Target Mode internal instruction:

```text
Use image 1 only as the source character identity/design reference and image 2 only as the target style reference. First extract the minimum source identity anchors that must survive: species/body type, face/head features, readable age/gender presentation if relevant, key costume logic, weapon/tool if present, palette relationships, and the most important macro silhouette cues. Then extract concrete target style anchors from image 2: shape language, silhouette/value grouping, line density, detail budget, proportion language, material simplification, shadow style, edge treatment, finish level, and framing tendency. Redraw the image 1 character as one full-body 2D playable game character that clearly belongs to image 2's style family. Let image 2 control rendering style, simplification level, proportions, line/detail density, value grouping, and finish. Let image 1 control identity only. If the target style is compact, icon-like, dominant-value-shape based, flat, low-detail, or highly abstract, compress and simplify the source design accordingly instead of preserving the source's original anatomy/proportion language or detailed rendering. Do not copy image 2's character identity, face/head design, distinctive facial coverings, clothing design, weapon/tool, symbols/markings, exact pose, story details, or background color. Output a 3:2 image on a plain chroma green screen background. No text, no logos, no callout lines, no concept-sheet layout, no extra views.
```

Do not preserve the second image's background color in Concept Target Mode; always use the global 3:2 chroma green output rule.

## Pose Mode

Trigger Pose Mode only when the user includes standalone `p` or standalone `pose`, case-insensitive, and does not also include Concept Target Mode markers.

Pose Mode expects two or more images:

1. First image: locked appearance reference. Use this for character identity, face, hair, body proportions, costume design, costume silhouette, palette, materials, weapon/tool design, and rendering style.
2. Every image after image 1: a separate strict neutralized pose reference. Treat each pose image independently as a skeleton/control diagram, not as an appearance or style reference. Use it only for body pose, gesture, head direction, torso tilt, shoulder/hip angle, arm and leg joint angles, hand and foot positions, prop holding angle/axis, camera angle, orientation, relative character scale, character position within the canvas, ground/contact anchor, center of mass, and framing. Adapt each pose layout into its own required 3:2 output canvas.

Pose reference neutralization:

- Before generation, mentally reduce each pose reference to neutral control data: joints, torso axis, limb angles, hand/foot anchors, prop axis/contact, center of mass, camera angle, and canvas placement.
- Ignore the pose reference's character identity, face, hair shape, costume design, costume silhouette, cloth shape, weapon design, palette, line style, shadow style, detail density, material finish, and rendering finish.
- Pose reference priority applies only to geometry: joint layout, body angle, hand/foot anchors, prop axis/contact, camera angle, center of mass, and canvas placement. It never applies to costume silhouette, hair design, cloth shape design, weapon design, palette, line style, detail density, lighting, or rendering finish; those always come from image 1.

Pose Mode output:

- Generate one refined final character image for each pose reference after image 1. With two input images total, output one result. With three input images total, output two results: image 1 appearance + image 2 pose, then image 1 appearance + image 3 pose.
- This mode is for one best-effort, carefully interpreted pose transfer per pose reference, not a set of tiers, variants, candidates, or AI-selected favorites.
- If there is only one pose reference, save the output as `01_p_refined_pose.png` unless a batch/source prefix is needed, then display it directly as a Markdown image.
- If there are multiple pose references, save the outputs in pose-reference order as `01_p_pose01_refined.png`, `02_p_pose02_refined.png`, `03_p_pose03_refined.png`, and so on unless a batch/source prefix is needed, then display them directly as Markdown images in the same order.
- Strictly preserve the first image's appearance: character identity, face, hair, body proportions, costume design, costume silhouette, weapon/tool design, palette, material finish, and rendering style.
- Strictly apply the current pose reference image's neutralized pose geometry: match head direction, torso tilt, shoulder/hip angle, arm and leg joint angles, hand and foot placement, prop/weapon holding angle, camera angle, orientation, relative character scale, character position within the canvas, ground/contact anchor, center of mass, and framing. Adapt these geometry cues into the required 3:2 canvas.
- Do not let image 1's original pose pull the result back toward the source pose. Image 1 controls all appearance; the current pose reference controls pose geometry only.
- Treat each pose reference independently. Do not blend multiple pose references, average them, combine limbs from different pose images, or use one pose reference to correct another.
- When the user gives extra limb, hand, or weapon-grip constraints, treat them as local constraints layered onto each current pose reference's locked pose. Keep the current pose reference's head direction, torso tilt, shoulder/hip angle, leg stride, foot anchors, center of mass, camera angle, and canvas placement unchanged. Adjust only the minimum wrist, fingers, hand occupancy, and prop contact needed to satisfy the user constraint.
- If the user specifies which hand holds or does not hold a weapon/tool, build an internal hand-occupancy map before generation, using only the user's specified hands, objects, and contact points, for example: `[specified hand] holds [specified weapon/tool]; [specified hand] is [specified empty/contact state]`. Include both the positive occupancy and the forbidden grip/contact in the prompt.
- Even when the user gives no extra hand instruction, treat hand pose as validation-critical. Build a hand map from the current pose reference for every visible hand: wrist location, palm direction, finger curl or open-hand state, occlusion order, grip/contact point, and whether the hand is empty or touching the prop.
- Do not copy any pose reference's character identity, face, hair shape, clothing design, clothing silhouette, cloth shapes, color palette, weapon design, line style, detail density, shadow style, background color, or rendering style.
- Preserve the pose reference's relative canvas layout within the 3:2 output: if the skeleton/pose figure is off-center, low/high in frame, or occupies a specific portion of the canvas, place the source character similarly after adapting to 3:2.
- If the source and current pose weapons differ, keep the first image's weapon design but align it to the current pose reference's hand/arm gesture and prop axis when possible. Never preserve the pose reference weapon's blade profile, markings, material, palette, ornament, or rendering style. If the current pose reference has no weapon but image 1 does, keep image 1's weapon in a pose-compatible hold or carry position without inventing the pose reference's weapon.
- If preserving image 1's clothing shape conflicts with the current pose reference's body pose, prioritize the current pose reference's body/limb placement and wrap image 1's costume design around that pose. Do not import the pose reference's clothing silhouette, loose-cloth/accessory shapes, hair shape, or simplified design language.
- Do not solve a hand/weapon constraint by changing the running direction, converting the pose into a standing pose, moving the feet, straightening the torso, changing the stride, changing the camera angle, or re-centering the character differently from the current pose reference.
- Keep the result as a clean single-character game asset, full body when possible, centered, with no text, no labels, no logos, and no extra views.

Pose Mode refined generation standard:

1. Carefully interpret each pose reference before generating that pose's result. Internally read the pose as if making a neutral control sheet: facing direction, camera angle, bounding box, ground/contact anchors, center of mass, head/chin/nose direction, neck, spine curve, torso lean, shoulder line, hip line, each shoulder/elbow/wrist/palm/finger group, each hip/knee/ankle/heel/toe anchor, weapon/prop axis and contact points, motion direction for hair/cloth, occlusion order, and relative canvas placement. Do not read costume, hair, weapon, palette, line style, detail density, or rendering style from the pose reference.
2. Convert that interpretation into one strict image-space pose-control prompt for that pose reference. Treat the current pose reference's body pose, hand pose, foot anchors, prop axis/contact, body geometry envelope, center of mass, and motion direction as higher priority than image 1's original stance or comfortable default anatomy. Do not treat the pose reference's costume silhouette, hair shape, cloth shapes, weapon design, line style, or detail density as priority pose data.
3. Wrap image 1's identity and costume around the interpreted pose. Preserve image 1's face/head design, body proportions, outfit structure, costume silhouette, palette, weapon/tool design, and rendering finish, but bend costume panels, body coverings, loose cloth, accessories, and props to follow the current pose reference's geometry layout without tracing the pose reference's clothing, hair/head shape, or weapon/tool shapes.
4. Generate one polished result at the strongest practical lock level for each pose reference. Do not output intermediate `outline lock`, `pose layout lock`, `pixel align lock`, or candidate files.
5. Do not generate multiple candidates and choose one internally. Only regenerate when the visible result plainly violates hard constraints such as pose drift, copied identity, broken hands, or wrong background.

Pose Mode internal instruction:

```text
Use image 1 as the locked appearance reference and every image after image 1 as an independent strict neutralized pose skeleton/control reference. For each pose reference, first reduce it to neutral geometry control data: facing direction, camera angle, bounding box, ground/contact anchors, center of mass, head/chin/nose direction, neck, spine curve, torso lean, shoulder and hip angles, every visible elbow/wrist/palm/finger group, every visible knee/ankle/heel/toe anchor, hand occupancy, weapon/prop axis and contact points, motion direction for hair/cloth, occlusion order, and relative canvas placement. Ignore the pose reference's character identity, face, hair shape, costume design, costume silhouette, cloth shapes, weapon design, palette, line style, shadow style, detail density, material finish, rendering style, and background color. Generate one refined 3:2 output for each pose reference on a plain chroma green screen background, in the same order as the pose reference images. Redraw the character from image 1 with image 1's identity, face/head design, body proportions, costume design, costume silhouette, weapon/tool design, palette, material finish, and rendering style preserved. Apply the current pose reference's interpreted geometry as strictly as possible: match joint layout, hands, feet, prop axis/contact, camera angle, orientation, relative character scale, character position within the canvas, ground/contact anchor, center of mass, and framing, adapted into a 3:2 output canvas. Wrap image 1's costume, body coverings, loose cloth, accessories, and weapon/tool design around the current pose geometry instead of letting image 1's original pose influence the result. Do not copy any pose reference's character identity, clothing design, clothing silhouette, hair/head shape, weapon/tool design, color palette, line style, detail density, rendering style, background color, or non-3:2 canvas aspect ratio. Do not blend or average multiple pose references. No text, no logos, no labels, no extra views.
```

Before calling image generation in Pose Mode, silently create one appearance-anchor checklist from image 1 and one separate neutral pose-control checklist for each pose reference after image 1. The pose-control checklist has priority over image 1's original stance only for geometry; the appearance-anchor checklist has priority for every non-geometry detail. The hand-occupancy map may only change hands, wrists, fingers, and weapon contact when the user explicitly overrides a hand/weapon detail.

After generation, reject or regenerate a specific output if that output has pose drift against its own pose reference: upright standing fallback, changed stride, changed foot anchors, changed torso lean, changed running direction, changed camera angle, wrong center of mass, wrong canvas placement, wrong hand occupancy, swapped hands, floating or disconnected hands, missing fingers, extra fingers, palm facing the wrong direction, wrist on the wrong side of the weapon/prop, fingers not wrapping when the pose reference shows a grip, an empty hand touching the weapon/prop, copied pose-reference identity/costume/weapon, copied pose-reference hair shape/clothing silhouette/line style/detail density, missing major image 1 appearance anchors, or hand/prop contact that violates the pose reference or the user's request.

## Workflow

Use this workflow for Simplification Mode. For `ct`, use Concept Target Mode instead. For `p`, use Pose Mode instead. For `cs` or `c s`, use Concept Simplify Mode instead. For `sq`, use Sequence Mode instead. If no mode is present, do not use any workflow.

1. Inspect the user's source material: prompt text, attached image, or both.
2. Extract only the essential character facts: role/species, body proportions, pose, camera angle, major silhouette masses, key costume pieces, key props, palette, visible lighting direction, and self-shadow style. Keep the output constraint fixed: 3:2 canvas with plain chroma green screen background.
3. Build three internal positive prompts, one for each simplification level, including linework simplification, outer-contour simplification, and shadow simplification.
4. Build an internal negative prompt that blocks dense rendering habits such as hatching, tiny seams, ornate folds, sketchiness, painterly texture, realistic surface detail, tiny edge fragments, shadowless flat cutouts, and background clutter.
5. Call the available image generation/editing workflow with the source image and the three internal prompts.
   - Generate each simplification level from the original reference, not from another generated level.
   - Preserve semantic ordering through file names and final reply image order. If multi-output display order is unreliable, generate levels sequentially.
   - Do not use local image processing as a substitute for image generation. Local steps may rename or arrange outputs, but must not simplify art, reduce colors, remove backgrounds, or create missing levels.
6. Compare the three outputs by outline and self-shadow readability, not only by interior detail. If the ragged/torn outer edge barely changes or `重度` loses all shadow, regenerate the unchanged level with stronger contour/shadow simplification wording.

## Simplification Levels

Use these three levels by default:

1. `轻度`: Clean up line noise while keeping most recognizable costume shapes, important secondary details, and most readable self-shadow masses. Remove sketchy strokes, texture noise, tiny random marks, and overly dense folds.
2. `中度`: Balanced game-asset simplification. Merge hair/head masses, clothing/body-covering shapes, constructed costume pieces, props, outer edge details, and shadow areas into larger readable shapes. Keep only the interior lines, contour breaks, and shadow blocks needed to explain pose, costume boundaries, volume, and gameplay identity.
3. `重度`: Maximum simplification while still recognizable. Use a bold redesigned silhouette, very few interior lines, broad flat color blocks, minimal props/detail marks, icon-like sprite readability, and one simple visible cel-shadow tone. Do not make this level completely shadowless by default.

## Outer Contour Simplification

Treat outer-contour simplification as a separate requirement from line cleanup.

- Preserve the character's macro silhouette: head size, body proportions, pose, main weapon/tool, and major clothing volume.
- Redesign the micro silhouette: do not trace every notch, tear, hair strand, strap tip, cloth point, material chip, or broken edge from the reference.
- For `轻度`, remove tiny edge noise and merge small torn points while keeping the overall tattered feeling.
- For `中度`, reduce ragged hems, broken sleeves, hair spikes, and cloth edges into a few larger readable chunks. A torn hem should become roughly 3 to 5 broad points instead of many small teeth.
- For `重度`, convert ragged or shredded edges into a simple readable contour with only 1 to 2 large tears or asymmetries if they are important to the character. Avoid sawtooth hems, fringe, shredded strips, and many dangling scraps.
- If worn, torn, damaged, or irregular materials are an identity cue in the source, preserve that story cue while simplifying the edge language aggressively across levels.

## Shadow Simplification

Treat shadow simplification as a separate requirement from flat color simplification.

- Preserve visible character self-shadows by default. Simplify them; do not remove them entirely.
- Preserve the reference light direction if it is clear. If unclear, use a simple upper-left light direction consistently across all three outputs.
- Use self-shadows on the character only. Do not add ground shadows, scenery shadows, or background gradients unless the user asks.
- For `轻度`, keep the main reference shadow shapes but remove noisy texture, tiny shade patches, and soft painterly blending.
- For `中度`, merge shadows into 2 to 4 clean cel-shaded masses on the body, clothing, hair, and props.
- For `重度`, use 1 simple darker tone with 1 to 3 broad shadow masses that show volume and pose. Avoid a completely flat, shadowless cutout.
- Avoid gradients, airbrush shading, rim-light effects, material texture, tiny fabric wrinkles rendered as shadows, and realistic rendering.

## Output Canvas And Background

- Use a 3:2 aspect ratio in every image-producing mode.
- Use one plain chroma green screen background in every image-producing mode.
- Do not preserve, inherit, sample, or copy source/reference background colors.
- Do not add scenery, gradients, decorative shapes, ground planes, UI, text, or cast shadows on the background.
- In Pose Mode, preserve each pose reference's relative character scale, position, ground/contact anchor, and framing for that pose's output, but adapt each into the required 3:2 canvas instead of copying a non-3:2 canvas.

## No Local Simplification Substitutes

Use image generation/editing to create the three visual levels. Never fake the three levels by running local filters over one generated image.

Forbidden substitutes include:

- palette reduction, color quantization, posterization, thresholding, dithering, or color-count reduction
- vector tracing, edge simplification filters, blur/sharpen pipelines, or mask erosion/dilation to simulate contour simplification
- generating only `轻度` and deriving `中度`/`重度` from it
- removing or replacing the background locally unless the user explicitly asks for post-processing

Allowed local operations:

- rename, move, or package already-generated outputs

If the available image generation backend cannot produce three independent level-specific edits, say that plainly instead of filling the gap with local simplification.

## Internal Positive Prompt Recipe

Prefer this structure:

```text
2D playable game character, full body, centered, 3:2 aspect ratio, plain chroma green screen background, [front/three-quarter/side view], [character identity and role], [pose], [key outfit/props], [simplification level instruction], clean simplified linework, simplified outer contour, macro silhouette preserved but micro edge contour redesigned, do not trace every small edge notch, few confident contour lines, minimal interior lines, large flat color shapes, controlled simple cel shading, visible self-shadow masses, [shadow level instruction], animation-ready sprite design, crisp readable shape language, no scenery
```

Use several of these phrases when appropriate:

- `clean simplified linework`
- `simplified outer contour`
- `macro silhouette preserved, micro edge contour redesigned`
- `merge ragged clothing edges into larger readable shapes`
- `remove tiny silhouette notches`
- `reduced interior detail lines`
- `few confident strokes`
- `bold readable outer silhouette`
- `large flat color regions`
- `simple cel shading`
- `controlled simple cel shading`
- `visible self-shadow masses`
- `one-tone cel shadow`
- `broad readable shadow shapes`
- `iconic game sprite shape language`
- `animation-ready 2D character asset`
- `3:2 aspect ratio`
- `plain chroma green screen background`

## Internal Negative Prompt Recipe

Include a concise negative prompt. Choose the terms that match the input:

```text
no dense linework, no sketchy strokes, no cross-hatching, no scratchy texture, no tiny fabric folds, no excessive seams, no ornate micro details, no exact traced silhouette, no tiny jagged edge nicks, no sawtooth hem, no shredded fringe, no many torn cloth points, no dangling tiny scraps, no shadowless flat cutout, no missing self-shadows, no gradients, no airbrush shading, no realistic pores, no painterly brush noise, no complex background, no scenery, no text, no watermark, no cropped body, no extra limbs
```

For Pose Mode, add pose-transfer negatives when appropriate:

```text
no loose pose interpretation, no source-pose drift, no keeping image 1's original pose, no upright standing fallback, no changed stride, no changed foot anchors, no changed torso lean, no changed running direction, no changed camera angle, no using pose reference as appearance reference, no copying pose-reference outfit, no copying pose-reference clothing silhouette, no copying pose-reference hair shape, no copying pose-reference cloth shapes, no copying pose-reference weapon design, no copying pose-reference weapon markings, no copying pose-reference palette, no copying pose-reference line style, no copying pose-reference detail density, no copying pose-reference rendering style, no copying pose-reference character identity, no changing source character appearance, no missing source appearance anchors, no wrong hand occupancy, no swapped hands, no floating hands, no disconnected hands, no missing fingers, no extra fingers, no wrong palm direction, no wrist on the wrong side of the weapon or prop, no fingers failing to wrap when gripping, no violating user-specified hand occupancy, no two-handed grip when a one-handed grip is specified, no empty hand touching the weapon
```

For Sequence Mode, add sequence-frame negatives when appropriate:

```text
no sprite sheet, no contact sheet, no animation timeline, no multi-frame collage, no multiple poses in one image, no onion-skin ghosts, no motion trails, no speed lines, no impact effects, no enemies, no extra characters, no frame labels, no frame numbers, no idle duplicate for walk/run/dash, no scenery, no ground plane
```

## Simplification Rules

- Preserve the character's identity, pose, proportions, core palette, weapon/tool, and distinctive macro silhouette.
- Simplify outer contours as strongly as interior details. Small silhouette noise is detail, not identity.
- Simplify hair into larger clumps. Avoid many thin strands.
- Simplify clothing into broad panels. Remove most folds, stitches, seams, buckles, tiny ornaments, shredded strips, and small jagged edge points unless they define the character.
- Simplify hard-surface, layered, or constructed costume elements into a few major readable pieces. Avoid many panel lines, scratches, rivets, engravings, and small construction marks.
- Use one clean outer contour and only the interior lines needed to explain anatomy, clothing boundaries, or props.
- Prefer flat colors plus controlled simple cel shading. Keep at least one readable self-shadow tone unless the user explicitly asks for totally flat colors.
- Keep the background a single flat chroma green screen color on a 3:2 canvas. Do not add ground shadows, scenery, props, UI, text, effects, or decorative shapes.
- Make the asset readable at small game size. If detail would disappear at thumbnail size, remove or merge it.

## Reference Image Handling

When the user provides a character image, use this as the image-to-image/edit instruction:

```text
Use the reference image for character identity, pose, proportions, palette, macro silhouette, and lighting direction. Redraw as three cleaner 2D playable game character asset variants: light simplification, medium simplification, and strong simplification. Keep the same 3:2 framing and character identity in all variants, but do not preserve the exact outer edge contour. Simplify torn clothing edges, ragged hems, hair spikes, straps, small silhouette notches, and shadow complexity more aggressively at each level. Keep visible self-shadows in all three variants: light keeps most main shadow masses, medium merges them into a few clean cel-shaded blocks, strong uses one simple shadow tone with 1 to 3 broad shadow masses. Use a plain chroma green screen background. Remove sketchy strokes, tiny folds, ornate micro details, texture noise, tiny edge fragments, painterly shadow noise, and background elements according to each simplification level. Do not add text labels inside the images.
```

Always redraw the output on a plain chroma green screen background with a 3:2 aspect ratio. Do not preserve the source background color and do not use neutral fallback colors.

## Prompt-Only Mode

Use this section only when the user explicitly asks for a prompt instead of an image. Output:

```markdown
**模式**
[s / ct / p / sq / cs]

**轻度简化提示词**
[light positive prompt]

**中度简化提示词**
[medium positive prompt]

**重度简化提示词**
[strong positive prompt]

**负向提示词**
[negative prompt]

**画布/背景**
[3:2, plain chroma green screen background]

**保留重点**
[short list of identity details preserved]
```

For Concept Target Mode prompt-only requests, output one `概念目标画风提示词` instead of three simplification prompts. For Pose Mode prompt-only requests, output one `精细姿势迁移提示词` per pose reference after image 1, labeled `pose01`, `pose02`, and so on when multiple pose references are provided. Each prompt must focus on carefully interpreting that pose reference before generating one final image. For Sequence Mode prompt-only requests, output five prompts labeled `walk`, `run`, `jump`, `attack`, and `dash`. For Concept Simplify Mode prompt-only requests, output three prompts labeled `低复杂度`, `中复杂度`, and `高复杂度`. In normal use, do not show this prompt-only format; generate the image directly.

## Generic Prompt Assembly Example

Input:

```text
$2DCS s
[one detailed single-character source image or source prompt]
```

Output:

```text
Generate three 2D playable game character variants from the reference/prompt: light simplification, medium simplification, and strong simplification. 3:2 aspect ratio, full body when possible, centered, [source view angle], [source character identity and role], [source pose], [key outfit/body-covering/accessory/prop anchors]. Keep [source identity anchors], pose, palette relationships, lighting direction, and framing consistent across all three. Use clean simplified linework, simplified outer contour, bold readable macro silhouette, fewer interior detail lines and fewer edge notches at each stronger level, large flat color shapes based on the source palette, controlled simple cel shading, visible self-shadow masses in all three variants, one simple shadow tone in the strong variant, animation-ready sprite design, on a plain chroma green screen background, no scenery, no text labels inside the images.
```

Negative:

```text
no dense linework, no sketchy strokes, no cross-hatching, no tiny source-specific detail noise, no excessive seams or construction marks, no exact traced silhouette, no tiny jagged edge nicks, no over-complex edge fragments, no shadowless flat cutout, no missing self-shadows, no gradients, no airbrush shading, no painterly texture, no inherited source background, no text, no watermark, no cropped body
```

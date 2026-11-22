---
name: film-prompt-rewriter
description: >-
  Rewrite ordinary image-to-video or text-to-video requests into specific, executable cinematic prompts. Use whenever the user asks to make a prompt more cinematic, film-like, atmospheric, emotionally controlled, visually coherent, or better for Vidu, Seedance, Kling, Veo, Runway, Sora, Luma, or similar video models. Also use when the user provides a rough scene description and wants camera movement, shot size, composition, lighting, color, atmosphere, performance, continuity, or model-adapted prompt guidance. Do not merely add cinematic adjectives: preserve the user's subject and intent, choose a coherent shot grammar, and explain the reusable visual structure.
---

# Film Prompt Rewriter

Turn a rough visual idea into a prompt that gives a video model a clear shot to execute, not a pile of aesthetic adjectives.

## Core promise

A useful cinematic rewrite makes four things observable:

1. **What is in the frame** — subject, setting, time, material and spatial relationship.
2. **What changes during the shot** — one primary event, subject motion and/or environmental motion.
3. **Why the camera moves** — shot size, camera position, movement, speed and what the movement reveals.
4. **What makes the image feel coherent** — lighting logic, color relationship, atmosphere, performance and continuity anchors.

The local sample library in `references/local-sample-library.json` is evidence for the project's Chinese cinematic vocabulary and reusable patterns. It is not a universal truth or a license to invent details. Use it to find analogous structures, then adapt the structure to the user's subject.

## First classify the request

- **Rewrite**: preserve the user's scene and improve specificity.
- **Adapt**: rewrite for a named model or workflow; read `references/model-adapters.md`. Treat every adapter as a conservative template, not a tested performance guarantee.
- **Compose**: combine selected visual modules; check for conflicting movement, lighting or scale.
- **Multi-shot**: detect more than just words like “then” or “cut”: multiple locations, time jumps, incompatible shot sizes, separate events, or a new camera position. Return a short timed shot plan, then one prompt per shot.
- **Critique**: identify the weakest executable part and provide a targeted revision, not a generic rewrite.
- **Static-image-only**: if the user asks only for a still image with no motion or temporal change, do not force this video Skill; provide a brief note or use the appropriate image skill.

If the subject is clear, infer the minimum needed detail and list it under **Assumptions**. Never present an inferred lens, aperture, sound, exact color value, identity detail, era detail or prop as a user fact unless the user supplied it, a bundled sample explicitly supports it, or the target model requires it. Ask only when the subject itself is genuinely ambiguous and a wrong assumption would make the prompt unusable.

This Skill is not for video editing, subtitles, full storyboards/director books, or rendering media. It rewrites prompts and plans; another tool must perform generation or editing.

## The one-shot contract

For a single generated clip, prefer:

- one shot
- one primary camera move, or an explicit locked camera
- one primary subject event
- one light idea
- one clear end state
- at most two **significant** motion sources among camera, subject and environment

This is a coherence rule, not a ban on subtle supporting motion. A slow orbit with the subject as the main event and flower petals or fabric responding subtly is coherent; count the orbit and the subject event as significant, and describe the other motion as a fine environmental response. A push, orbit, whip pan, crane and explosion in the same eight seconds is not.

## Rewrite workflow

1. Extract the user's **must-keep facts**: subject, action, setting, mood, era, aspect ratio, duration, references and target model.
2. Choose the shot's **visual intent** in one phrase: for example, “draw the viewer into a private sadness” or “make the tiny human feel overpowered by scale.”
3. Select one camera grammar: locked, push-in, pull-back, orbit, tracking, handheld, whip pan, crane or focus transition. Give it motivation and an end state.
4. Add only the missing constraints that improve execution: shot size, camera height, focal-length tendency, foreground/midground/background layers, light source and color relationship.
5. Make the action temporal: describe the starting state, the turn, and the ending state. Replace “cinematic movement” with visible behavior.
6. Add human or material truth: gaze target, weight shift, hand behavior, fabric lag, reflections, dust, focus settling or another grounded cue when appropriate.
7. Remove contradictions and overloaded adjectives. Keep the prompt within the target model's practical length.
8. Run the quality check below, then write the output in the user's language unless the user asks for another prompt language.

## Output contract

For a normal rewrite, use these three sections by default. A quick request may collapse sections 2 and 3 into a short note, but it must still include the executable prompt and assumptions.

### 1. 可直接使用的提示词
One coherent prompt in prose. Put negative constraints in a separate `负面约束` line when useful. Do not bury the prompt inside a long essay.

### 2. 镜头拆解
Start with **Assumptions** (only details inferred rather than supplied). Then use a compact table or bullets:

- 视觉意图
- 主体与环境
- 起始状态 → 转折 → 结束状态
- 景别与机位
- 构图与空间层次
- 镜头运动与动机
- 光影
- 色彩关系
- 氛围与材质
- 人物表演 / 物理细节
- 连续性锚点

### 3. 复用说明
State which local sample pattern(s) were used, if any, and what was deliberately changed. When a sample is used, cite its exact `id`, `title`, `confidence`, and the mechanism transferred; never paste the whole sample unchanged. Distinguish:

- **本地样本依据** — supported by the bundled reviewed library; cite `sample_id` and `confidence`.
- **通用电影语言** — a general craft choice, not a claim from the samples.
- **待验证** — model-specific behavior or a detail not covered by the samples.

If no sample reaches the retrieval threshold, say `未找到足够相似的本地样本` and use only clearly marked general craft guidance.

For a quick request, collapse sections 2 and 3 into a short note, but never remove the executable prompt.

## Quality check before answering

Reject or revise a draft if:

- “cinematic”, “大片感”, “高级” are doing the work without observable details.
- It contains multiple unrelated camera moves or “then / next / cut to” inside a one-shot prompt.
- The main action has no start state, turn or endpoint.
- Lighting, color and environment contradict one another.
- The prompt changes the user's subject, identity, era or required object without saying so.
- It adds unsupported brand names, director imitation, invented camera tests or fake model guarantees.
- It promises continuity the target model cannot reliably control; mark it as a goal or use a multi-shot plan.
- Negative wording is mixed into the main prose in a way likely to summon the unwanted object; move it to `负面约束`.

## Local sample retrieval

Use the bundled JSON as a text/field reference library. The public Skill package intentionally does **not** bundle stills or GIFs; `representative_still` and `hover_gif` are null by policy. Do not claim to view media from this package. The internal source catalogue can map a cited entry back to its original media when available.

Use this deterministic retrieval order:

1. Normalize the request into intent tokens: subject, action, shot size, composition, camera motion, lighting, color, atmosphere, performance and continuity.
2. Rank exact or near matches in `title`, `normalized_prompt`, then the structured `fields`; prefer mechanism overlap over shared mood adjectives.
3. Keep at most three candidates. Require at least two meaningful field/mechanism matches, or one exact title/mechanism match.
4. Prefer `confidence: high`, then `medium`. Use `low` only when the output explicitly says the evidence is weak.
5. If the threshold is not met, do not cite a local sample; say `未找到足够相似的本地样本` and use marked general guidance.
6. Cite each selected sample by `sample_id`, `title`, `confidence`, source video/time and the specific mechanism transferred.

Useful mechanism examples:

- emotion + foreground obstruction + push-in → “鱼缸前的忧郁推进”
- handheld realism + city lights + isolation → “公交车内的手持孤独感”
- orbit + golden hour + dreamlike atmosphere → “黄金时刻花海环绕运镜”
- negative space / scale contrast → reviewed composition and science-fiction entries
- gothic / wuxia / wasteland / retro-agent → reviewed style entries

Do not paste a sample unchanged when the user's subject differs. Preserve the visual mechanism and replace the scene facts.

## References

- `references/model-adapters.md` — concise model/workflow adaptation notes.
- `references/rewrite-patterns.md` — reusable visual mechanisms distilled from the reviewed samples.
- `references/local-sample-library.json` — 92 reviewed-initial examples with source, still, GIF and structured fields.

## Skill boundary

This Skill rewrites and explains prompts. It does not claim that a prompt will generate a perfect result, does not generate media unless another tool is explicitly requested, and does not replace model-specific testing. When evidence is insufficient, say what is inferred and what should be tested.

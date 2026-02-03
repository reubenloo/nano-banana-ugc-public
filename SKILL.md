---
name: ugc-image-gen
description: Guide for generating consistent UGC-style character images with AI image models, using reference images and human QA.
---

# Consistent UGC Image Generation Guide

## Goal

Generate a set of images that look like **real UGC** (phone selfie / casual environment) while keeping the **same character identity** across multiple shots.

## Requirements

You need an image generation service that supports:

1. **Text-to-image** (baseline)
2. **Image-to-image / edits** with a **reference image** (recommended for consistency)

## API Examples

### Text-to-image

```bash
curl -s "$IMAGE_API_BASE_URL/v1/images/generations" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $IMAGE_API_KEY" \
  -d '{
    "model": "YOUR_IMAGE_MODEL",
    "prompt": "YOUR_PROMPT_HERE",
    "size": "1024x1792",
    "n": 1,
    "response_format": "b64_json"
  }' | jq -r '.data[0].b64_json' | base64 -d > output.jpg
```

### Image-to-image (Reference / Edits)

Use this when you need the **same person** across multiple shots.

```bash
curl -s "$IMAGE_API_BASE_URL/v1/images/edits" \
  -H "Authorization: Bearer $IMAGE_API_KEY" \
  -F "image=@/path/to/master-character.jpg" \
  -F "prompt=Same exact person as the reference photo. New pose/expression. Preserve identity." \
  -F "model=YOUR_IMAGE_MODEL" \
  -F "size=1024x1792" > response.json

jq -r '.data[0].b64_json' response.json | base64 -d > output.jpg
```

## Prompting Best Practices

### 1) Achieve a Real UGC Look

Add one or two of these to steer toward a phone-camera look:
- “Phone front camera selfie”
- “Natural skin texture, minimal/no makeup”
- “Slight grain / low compression artifacts”
- “Indoor ambient lighting, not studio lighting”

**Avoid**: “cinematic”, “studio”, “professional photoshoot”, “perfect skin”, “beauty filter”.

### 2) Framing

- Prefer upper-body or waist-up framing (arm’s-length phone distance).
- Close-ups exaggerate artifacts and make consistency harder.

### 3) Separate Identity from Scene

Keep **identity** stable and only vary **scene/pose**.
- **Identity block**: age range, general appearance, hairstyle, wardrobe baseline.
- **Scene block**: location, lighting, action, expression, camera angle.

## Consistency Workflow

### Phase 1: Create a Master Character
1. Generate a strong “master” image (good lighting, clean face, neutral expression).
2. Save it as `master-character.jpg`.
3. Generate 2–3 variants using reference-based edits, changing only pose/expression.
4. Pick the best as the final master.

### Phase 2: Generate the Shot List
For each shot:
1. Use the master as a reference.
2. Prompt only what’s changing (pose, environment, framing).
3. Verify identity consistency.

## Visual QA (Human-in-the-loop)

Use a two-step procedure for review:
1. **Neutral Description Pass**: Describe the subject (face, hair, clothing), hands, and environment factually.
2. **Reference Comparison Pass**: Compare against the master character. Note identity drift or geometry errors.

## Common Failure Modes

- **Identity Drift**: Face shape, hairstyle, or age impression changes across shots.
- **Anatomy Errors**: Extra fingers or distorted hands.
- **Over-stylization**: Image looks like a studio photoshoot rather than casual UGC.

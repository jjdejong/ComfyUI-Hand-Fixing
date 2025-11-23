# Flux Dev Camera & Lighting Control Guide

## Quick Start

This workflow provides **preset-based camera and lighting control** for Flux Dev generations on Apple Silicon.

### What You Get
- **8 camera positions** (cube faces: front, back, left, right, top, bottom + diagonals)
- **10+ lighting directions** (matching cube positions + artistic styles)
- **Apple Silicon optimized** (GGUF model, MPS-compatible settings)
- **Simple copy-paste interface** (no complex ControlNet setup needed)

---

## Important Limitation

⚠️ **Flux Dev has NO dedicated lighting direction ControlNet as of 2024.**

This workflow uses **prompt-based control**, which is the current state-of-the-art method for Flux lighting control. Results depend on prompt adherence and may vary.

For true geometric/depth control, see "Advanced: Depth ControlNet" section below.

---

## How to Use

### 1. Open the Workflow
Load `Flux_Dev_Camera_Lighting_Control.json` in ComfyUI

### 2. Set Your Subject
Edit the **"Base Subject Prompt"** node (green):
```
a red cube on a pedestal, photorealistic, 8k, detailed
```
Replace with your subject.

### 3. Choose Camera Position
Copy ONE preset from below into **"Camera Position"** node (blue):

#### Basic Positions (6 faces)
```
Front:   viewed from front center
Back:    viewed from behind, back view
Left:    viewed from left side
Right:   viewed from right side
Top:     viewed from above, top-down view, bird's eye view
Bottom:  viewed from below, worm's eye view, low angle
```

#### Diagonal Corners
```
Front Top-Left:      viewed from front top-left angle
Front Top-Right:     viewed from front top-right angle
Front Bottom-Left:   viewed from front bottom-left, low angle from left
Front Bottom-Right:  viewed from front bottom-right, low angle from right
Back Top-Left:       viewed from back top-left angle, elevated rear view
Back Top-Right:      viewed from back top-right angle, elevated rear view
```

#### Specialty Angles
```
Extreme Close-up:    extreme close-up, macro view
Wide Angle:          wide angle view, expansive perspective
3/4 View:            three-quarter view from front-right
Isometric:           isometric view, game asset style
```

### 4. Choose Lighting Direction
Copy ONE preset from below into **"Lighting Direction"** node (orange):

#### Basic Lighting (6 directions)
```
Front:   lit from front, flat even lighting, minimal shadows
Back:    backlit, rim lighting, silhouette effect, dramatic backlighting
Left:    lit from left side, side lighting, half in shadow
Right:   lit from right side, side lighting, half in shadow
Top:     lit from above, overhead lighting, strong vertical shadows
Bottom:  lit from below, underlighting, dramatic upward shadows, horror lighting
```

#### Diagonal Lighting (cube corners)
```
Front Top-Left:      lit from front top-left, 3/4 lighting, soft shadows to bottom-right
Front Top-Right:     lit from front top-right, 3/4 lighting, soft shadows to bottom-left
Back Top-Left:       lit from back top-left, rim light from upper-left, dramatic edge glow
Back Top-Right:      lit from back top-right, rim light from upper-right, dramatic edge glow
```

#### Artistic Styles
```
Rembrandt:   lit from front top-left, dramatic lighting, strong shadows to bottom-right, chiaroscuro
Split:       split lighting from hard left, one half lit one half shadow, dramatic contrast
Golden Hour: warm golden hour lighting from low right, soft warm glow, long shadows
Cinematic:   cinematic lighting from back-left, rim light with blue fill, teal and orange
Studio:      bright even studio lighting from front, minimal shadows, commercial look
```

### 5. Generate
Click **Queue Prompt**

---

## Example Combinations

### Portrait Photography
```
Subject:  a woman's face, photorealistic portrait, detailed skin
Camera:   viewed from front center
Lighting: lit from front top-left, 3/4 lighting, soft shadows to bottom-right
Result:   Classic portrait lighting
```

### Product Shot
```
Subject:  luxury watch on marble surface, commercial photography
Camera:   three-quarter view from front-right
Lighting: bright even studio lighting from front, minimal shadows, commercial look
Result:   Professional product photography
```

### Dramatic Character
```
Subject:  cyberpunk character in neon city, cinematic
Camera:   viewed from below, worm's eye view, low angle
Lighting: backlit, rim lighting, silhouette effect, dramatic backlighting
Result:   Epic hero shot
```

### Architectural Visualization
```
Subject:  modern building exterior, architectural render
Camera:   viewed from front top-right angle
Lighting: warm golden hour lighting from low right, soft warm glow, long shadows
Result:   Architectural sunset shot
```

---

## Technical Details

### Model Requirements
- **Flux Model**: `flux1-dev-Q4_K_S.gguf` (GGUF quantized for Mac)
  - Download from Hugging Face
  - Place in: `ComfyUI/models/unet/`

- **CLIP Models**:
  - `t5xxl_fp16.safetensors`
  - `clip_l.safetensors`
  - Place in: `ComfyUI/models/clip/`

- **VAE**: `ae.safetensors`
  - Place in: `ComfyUI/models/vae/`

### Generation Settings
```
Resolution:  1024×1024 (Flux native)
Steps:       20
CFG Scale:   1.0 (Flux uses guidance distillation)
Sampler:     euler
Scheduler:   simple
Denoise:     1.0
```

### Apple Silicon Performance
- **VRAM Usage**: ~8-12GB
- **Generation Time**: ~45-90s (M1/M2/M3 with 16GB+)
- **Compatibility**: Optimized for MPS backend
  - Uses GGUF quantization (70% faster model loading)
  - Euler sampler (best MPS compatibility)
  - No XLabs nodes (avoids MPS crashes)

---

## Advanced: Adding Depth ControlNet

For true spatial/geometric control beyond prompts:

### Option 1: Official Depth ControlNet
1. Download `FLUX.1-Depth-dev` from [black-forest-labs](https://huggingface.co/black-forest-labs/FLUX.1-Depth-dev)
2. Place in `ComfyUI/models/controlnet/`
3. Add nodes to workflow:
   ```
   ControlNetLoader → Load "FLUX.1-Depth-dev"
   DepthEstimator → Generate depth from reference image
   ControlNetApply → Conditioning scale: 0.5-0.7
   ```
4. Connect between CLIP encoding and KSampler

### Option 2: Union ControlNet (Multi-Mode)
1. Download `FLUX.1-dev-Controlnet-Union-Pro` from [Shakker-Labs](https://huggingface.co/Shakker-Labs/FLUX.1-dev-ControlNet-Union-Pro)
2. Supports: Depth, Canny, Pose, Soft Edge, Gray
3. Use `mode=depth` for spatial control
4. Conditioning scale: 0.4-0.6

**Benefits of Adding Depth:**
- Preserves spatial relationships from reference images
- Controls camera perspective geometrically
- Maintains consistent 3D structure
- Complements prompt-based lighting

**Combined Workflow:**
```
Depth ControlNet (0.6) → Geometric/spatial control
+ Camera prompts        → Viewpoint description
+ Lighting prompts      → Light direction/quality
= Precise control over entire scene
```

---

## Cube Position Reference

Imagine your subject at the center of a cube. Each position is one of:
- **6 faces**: front, back, left, right, top, bottom
- **8 corners**: front-top-left, front-top-right, etc.
- **12 edges**: combinations of adjacent faces

### Spatial Coordinates
```
        Top
         |
    Back | Front
         |
       Bottom

Left ← Subject → Right
```

### Lighting Direction Matches Camera Position
For example:
- **Camera**: front-top-left = you're standing at the front-top-left corner looking at the subject
- **Lighting**: front-top-left = light source is at the front-top-left corner shining on the subject

**Tip**: Opposite lighting to camera creates rim lighting/silhouettes.

---

## Troubleshooting

### "Model not found" error
- Download required models (see Technical Details)
- Check file paths in model loader nodes
- Ensure models are in correct ComfyUI folders

### Poor lighting control
- Lighting control is prompt-based; results vary
- Try increasing prompt detail: "strong dramatic shadows" vs "shadows"
- Combine multiple lighting keywords
- Consider adding Depth ControlNet for better spatial control

### MPS/Apple Silicon errors
- Ensure using GGUF model (not fp16/fp8)
- Use euler sampler (not dpmpp)
- Use simple scheduler (not karras)
- Update PyTorch to 2.3.1+ for latest MPS fixes

### Slow generation
- Normal for Flux on M1/M2: 45-90s
- Reduce steps to 15 for faster preview
- Consider ComfyUI-MLX extension for 35% speedup

### Image doesn't match camera/lighting
- Flux prompt adherence varies
- Try regenerating with different seed
- Make prompts more specific
- Add reference keywords: "professional photography", "studio shot"

---

## Credits & References

**Workflow Design**: ComfyUI-Hand-Fixing project
**Flux Model**: Black Forest Labs
**Optimization**: Community research on Apple Silicon compatibility

**Resources**:
- [Flux Official Models](https://huggingface.co/black-forest-labs)
- [ComfyUI Flux Guide](https://docs.comfy.org/tutorials/flux/)
- [Apple Silicon Optimization](https://github.com/thoddnn/ComfyUI-MLX)

---

## License

This workflow is part of the ComfyUI-Hand-Fixing project.
See repository LICENSE for details.

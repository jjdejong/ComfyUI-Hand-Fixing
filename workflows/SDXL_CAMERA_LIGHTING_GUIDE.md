# SDXL Camera & Lighting Control Guide

## Overview

This workflow provides **REAL lighting direction control** for SDXL using the **VIDIT-FAID ControlNet** - a dedicated lighting control model that uses depth maps to control where light falls in your scene.

### Key Advantage Over Flux
**SDXL has a dedicated lighting direction ControlNet** (VIDIT-FAID), unlike Flux which only has prompt-based control. This means more precise and consistent lighting control.

---

## What You Get

✅ **Real Lighting Control**: VIDIT-FAID ControlNet uses depth maps to control lighting direction
✅ **Camera Position Presets**: Cube-based positions (front, back, diagonals, etc.)
✅ **Depth-Based Lighting**: White areas = more lit, black areas = shadowed
✅ **Apple Silicon Compatible**: Works on M1/M2/M3/M4
✅ **Three Usage Modes**: Auto-depth, custom depth maps, or reference images

---

## Required Models

### 1. SDXL Checkpoint (Required)
Any SDXL model works. Example:
```
File: cyberrealisticXL_v70_fp32.safetensors
Location: ComfyUI/models/checkpoints/
```

### 2. VIDIT-FAID ControlNet (Required) ⭐
**This is the KEY model for lighting control!**

Download from: [SargeZT/controlnet-sd-xl-1.0-depth-faid-vidit](https://huggingface.co/SargeZT/controlnet-sd-xl-1.0-depth-faid-vidit)

Files (choose ONE):
- `controlnet-sd-xl-1.0-depth-faid-vidit.safetensors` (if available)
- OR `diffusion_pytorch_model.safetensors` (rename after download)

Location: `ComfyUI/models/controlnet/`

**What it does**: Controls lighting direction via depth map conditioning. Trained on VIDIT and FAID datasets specifically for lighting control.

### 3. ControlNet Auxiliary Preprocessors (Auto-installs)
ComfyUI Manager will prompt you to install `comfyui_controlnet_aux` for the MiDaS depth estimator.

---

## How It Works

### Lighting Control via Depth Maps

VIDIT-FAID interprets depth maps as lighting information:
- **White pixels** = Close to light source = **BRIGHTLY LIT**
- **Black pixels** = Far from light source = **IN SHADOW**
- **Gray pixels** = Mid-distance = **PARTIALLY LIT**

### Example: Light from Front Top-Left
```
Depth map gradient:
  Front-top-left corner: WHITE (most lit)
  → Diagonal gradient →
  Back-bottom-right corner: BLACK (shadowed)
```

### Example: Light from Above
```
Top surfaces: WHITE
Middle areas: GRAY
Bottom surfaces: BLACK
```

---

## Three Ways to Use

### Method 1: Auto-Generated Depth (Easiest)

**Best for**: Quick iterations, learning the workflow

1. Open `SDXL_Camera_Lighting_Control.json`
2. Set **Base Subject Prompt**: `"a red cube on pedestal, photorealistic"`
3. Set **Camera Position**: `"viewed from front center"`
4. Set **Lighting Direction**: `"dramatic lighting, strong shadows"`
5. Click **Queue Prompt**
6. On first run, a basic depth is generated
7. Output image feeds back to create better depth map
8. Re-run for improved lighting

**Note**: First generation uses auto-generated depth from the output image itself (feedback loop). Quality improves on subsequent runs.

### Method 2: Upload Custom Depth Map (Best Control)

**Best for**: Precise lighting control, matching specific lighting setups

#### Creating Depth Maps

**Option A: Manual in Photoshop/GIMP**
1. Create 1024×1024 grayscale image
2. Paint lighting pattern:
   - White brush = lit areas
   - Black brush = shadowed areas
   - Soft gradient = smooth lighting transition
   - Hard edges = sharp shadow boundaries

**Option B: 3D Software (Blender, etc.)**
1. Model your scene geometry
2. Render depth pass (Z-depth)
3. Normalize: closest object = white, farthest = black
4. Export as PNG

**Option C: Download Depth Map Templates**
- Many depth maps available online for common lighting setups
- Search "depth map template lighting"

#### Using Custom Depth Maps
1. Load depth map in **"Load Depth Map"** node
2. Set camera and lighting prompts to match your depth map
3. Generate

### Method 3: Use Reference Image

**Best for**: Matching lighting from existing photos

1. Load reference photo in **"Load Depth Map"** node
2. MiDaS automatically generates depth map
3. Set prompts describing the scene
4. Generate - lighting will match reference depth structure

**Example**:
```
Reference: Portrait with side lighting
→ MiDaS extracts depth
→ VIDIT-FAID applies that lighting pattern
→ Your subject gets same lighting as reference
```

---

## Camera Position Presets

Copy ONE preset into **"Camera Position"** node:

### Basic Positions (6 cube faces)
```
Front:   viewed from front center
Back:    viewed from behind, back view
Left:    viewed from left side
Right:   viewed from right side
Top:     viewed from above, top-down view, bird's eye view
Bottom:  viewed from below, worm's eye view, low angle
```

### Diagonal Corners (8 positions)
```
Front Top-Left:      viewed from front top-left angle
Front Top-Right:     viewed from front top-right angle
Front Bottom-Left:   viewed from front bottom-left, low angle from left
Front Bottom-Right:  viewed from front bottom-right, low angle from right
Back Top-Left:       viewed from back top-left angle, elevated rear view
Back Top-Right:      viewed from back top-right angle, elevated rear view
Back Bottom-Left:    viewed from back bottom-left, low rear angle from left
Back Bottom-Right:   viewed from back bottom-right, low rear angle from right
```

### Specialty Angles
```
Extreme Close-up:    extreme close-up, macro view
Wide Angle:          wide angle view, expansive perspective
Dutch Angle:         dutch angle, tilted perspective
3/4 View:            three-quarter view from front-right
Profile:             perfect side profile view from right
Isometric:           isometric view, game asset style
```

---

## Lighting Direction Presets

Copy ONE preset into **"Lighting Direction"** node:

### Lighting Quality (Refines the depth-controlled lighting)
```
Dramatic:    dramatic lighting, strong shadows, high contrast
Soft:        soft diffused lighting, gentle shadows, even exposure
Studio:      professional studio lighting, minimal shadows, clean
Cinematic:   cinematic lighting, moody atmosphere, dramatic shadows
Natural:     natural lighting, realistic shadows, photographic
Rembrandt:   Rembrandt lighting, chiaroscuro, dramatic side lighting
Rim Light:   rim lighting, edge glow, backlit highlights
Golden Hour: warm golden hour lighting, soft warm glow, long shadows
Hard:        hard direct lighting, sharp crisp shadows, high contrast
```

### Directional Hints (Optional - depth map is primary)
Add directional keywords to reinforce depth map:
```
lit from front
lit from back
lit from above
lit from side
backlit
top-down lighting
side lighting
```

### Combined Example
```
Lighting: dramatic lighting, strong shadows, lit from back top-right, rim lighting
```

**Important**: Depth map controls WHERE light falls. Prompts control HOW the light looks (quality, color, intensity).

---

## Understanding Depth Maps for Lighting

### Depth Map Patterns for Cube Positions

#### Front Lighting
```
Depth pattern:
  Front face: WHITE (close to light)
  Side faces: GRAY (angled)
  Back face: BLACK (far from light)
```

#### Back Lighting (Rim Light)
```
Depth pattern:
  Back edges: WHITE (lit rim)
  Front face: BLACK (silhouette)
  Creates dramatic backlit effect
```

#### Top Lighting
```
Depth pattern:
  Top surface: WHITE
  Vertical surfaces: GRAY (gradient top→bottom)
  Bottom surface: BLACK
```

#### Diagonal Top-Left Lighting
```
Depth pattern:
  Top-left corner: WHITE
  Diagonal gradient across object
  Bottom-right corner: BLACK
  Creates 3/4 lighting effect
```

### Tips for Depth Maps

- **Soft gradients** = Soft lighting transitions
- **Hard edges** = Sharp shadow boundaries
- **High contrast** (pure white/black) = Dramatic lighting
- **Low contrast** (grays) = Flat, even lighting
- **White highlights** = Specular reflections, glossy surfaces
- **Multiple white areas** = Multiple light sources

---

## Technical Settings

### ControlNet Strength
Adjust in the **"Apply VIDIT-FAID Lighting Control"** node:

```
0.4-0.5:  Subtle lighting, more creative freedom
0.6-0.7:  Balanced - RECOMMENDED ✅
0.8-0.9:  Strong depth adherence, precise lighting
```

**When to adjust**:
- Lighting too weak? → Increase to 0.75-0.85
- Want more creative variation? → Decrease to 0.5-0.6
- Perfect balance? → Keep at 0.65

### Generation Settings
```
Resolution:  1024×1024 (SDXL native)
Steps:       30 (range: 25-40)
CFG Scale:   7.0 (range: 5-10)
Sampler:     dpmpp_2m_sde
Scheduler:   karras
Denoise:     1.0 (full generation)
```

### Apple Silicon Performance
```
Hardware:     M1/M2/M3/M4 with 16GB+ RAM
VRAM Usage:   ~10-14GB
Time:         ~60-120s per generation
Compatibility: Native - no special models needed
```

**Optimization**:
- Uses standard SDXL (fp32 works fine)
- dpmpp_2m_sde sampler (best MPS compatibility)
- Karras scheduler (smooth sampling)

---

## Example Workflows

### Portrait with Rembrandt Lighting
```
Subject:  professional headshot of a woman, photorealistic portrait
Camera:   viewed from front center
Lighting: dramatic lighting, strong shadows, Rembrandt lighting, chiaroscuro

Depth Map: Front-top-left corner WHITE → back-bottom-right BLACK
Result:   Classic Rembrandt triangle of light on cheek
```

### Product Shot with Studio Lighting
```
Subject:  luxury watch on marble surface, commercial photography
Camera:   three-quarter view from front-right
Lighting: professional studio lighting, minimal shadows, clean, bright

Depth Map: Even white across front/top, gradual falloff to back
Result:   Clean commercial product shot
```

### Cinematic Character Lighting
```
Subject:  cyberpunk character in neon city
Camera:   viewed from below, low angle
Lighting: cinematic lighting, rim light, backlit, dramatic edge glow

Depth Map: Back edges WHITE (rim), front mostly BLACK (silhouette)
Result:   Epic backlit hero shot
```

### Architectural Golden Hour
```
Subject:  modern building exterior, architectural visualization
Camera:   viewed from front top-right angle
Lighting: warm golden hour lighting, soft warm glow, long shadows

Depth Map: Front-right surfaces WHITE, back-left BLACK, warm tone
Result:   Architectural sunset shot with dramatic shadows
```

---

## Comparison: SDXL vs Flux

| Feature | SDXL (This Workflow) | Flux Workflow |
|---------|---------------------|---------------|
| **Lighting Control** | ✅ Dedicated ControlNet (VIDIT-FAID) | ⚠️ Prompt-based only |
| **Precision** | ⭐⭐⭐⭐⭐ Very High | ⭐⭐⭐ Medium |
| **Consistency** | ⭐⭐⭐⭐⭐ Excellent | ⭐⭐⭐ Good |
| **Camera Control** | 📝 Prompt-based | 📝 Prompt-based |
| **Mac Compatible** | ✅ Yes (standard) | ✅ Yes (GGUF) |
| **Speed** | ~60-120s | ~45-90s |
| **Learning Curve** | Medium (depth maps) | Easy (prompts only) |
| **Best For** | Precise lighting | Fast iteration |

### When to Use Each

**Use SDXL** when:
- You need precise, consistent lighting control
- You have/can create depth maps
- Lighting is critical to the image
- You're matching reference lighting

**Use Flux** when:
- You want fast iteration
- Lighting is less critical
- No depth maps available
- Exploring creative variations

---

## Troubleshooting

### "Model not found" error
**Solution**:
1. Download VIDIT-FAID from [HuggingFace](https://huggingface.co/SargeZT/controlnet-sd-xl-1.0-depth-faid-vidit)
2. Place in `ComfyUI/models/controlnet/`
3. Restart ComfyUI
4. Update model name in ControlNetLoader node if needed

### Lighting doesn't match depth map
**Causes & Solutions**:
- ControlNet strength too low → Increase to 0.75-0.85
- Prompts conflict with depth → Remove directional keywords from prompt
- Depth map unclear → Increase contrast (more white/black, less gray)
- CFG too high → Lower to 5-6 for less rigid interpretation

### Depth map not loading
**Solution**:
1. Check image is grayscale or RGB (both work)
2. Verify resolution (1024×1024 recommended)
3. Ensure file format is PNG/JPG
4. Try re-saving in different image editor

### Results too dark/light
**Solutions**:
- Too dark: Increase white areas in depth map, add "bright" to prompt
- Too light: Increase black areas in depth map, add "shadows" to prompt
- Adjust CFG: Higher (8-9) = more literal, Lower (5-6) = more flexible

### MPS/Apple Silicon errors
**Solution**:
- Update PyTorch: `pip install --upgrade torch torchvision`
- Use dpmpp_2m_sde sampler (not euler_a)
- Ensure ComfyUI is updated to latest version
- Check VRAM: close other apps, reduce batch size to 1

### Slow generation on Mac
**Normal**: 60-120s is expected for SDXL on M1/M2
**Optimizations**:
- Reduce steps to 25
- Use smaller resolution (768×768) for testing
- Close other memory-intensive apps
- Consider ComfyUI-MLX extension for 30% speedup

---

## Advanced Tips

### Creating Depth Maps from 3D Software

**Blender Setup**:
1. Model your scene or import object
2. Add camera at desired position
3. Switch to Camera view
4. Compositor → Add Render Layers → Map Value → Normalize
5. Set Z-depth output range: 0 (closest) = white, max distance = black
6. Render → Save as PNG
7. Use in ComfyUI

### Combining Multiple Light Sources

Create depth maps with multiple white regions:
```
Example: Key + Fill + Rim lighting
- Front-top-left: Bright white (key light)
- Front-right: Medium gray (fill light)
- Back edges: White highlights (rim light)
```

VIDIT-FAID interprets this as multiple light sources.

### Matching Real-World Lighting

1. Take reference photo with known lighting
2. Load in "Load Depth Map" node
3. MiDaS extracts depth
4. Generate with your subject
5. Result: Your subject lit like the reference scene

### Iterative Refinement

1. First pass: Use auto-depth, generate
2. Check lighting: Too flat? Too dramatic?
3. Adjust ControlNet strength
4. Regenerate
5. Export current output as depth map
6. Manually edit depth map in Photoshop
7. Re-import, generate again

---

## Resources

### Download Links
- [VIDIT-FAID ControlNet](https://huggingface.co/SargeZT/controlnet-sd-xl-1.0-depth-faid-vidit)
- [Original Nahrawy VIDIT-FAID](https://huggingface.co/Nahrawy/controlnet-VIDIT-FAID) (SD1.5 version)
- [SDXL ControlNet Collection](https://huggingface.co/lllyasviel/sd_control_collection)

### Learning Resources
- [Stable Diffusion Art: 3 ways to control lighting](https://stable-diffusion-art.com/control-lighting/)
- [How to use ControlNet with SDXL](https://stable-diffusion-art.com/controlnet-sdxl/)
- [ComfyUI ControlNet Documentation](https://docs.comfy.org/)

### Depth Map Resources
- Search "depth map dataset" for examples
- Blender tutorials for Z-depth rendering
- Photoshop tutorials for manual depth painting

---

## Credits

**Workflow Design**: ComfyUI-Hand-Fixing project
**VIDIT-FAID ControlNet**: SargeZT (SDXL port), Nahrawy (original SD1.5)
**Research**: VIDIT Dataset, FAID Dataset
**SDXL**: Stability AI
**ComfyUI**: comfyanonymous and contributors

---

## License

This workflow is part of the ComfyUI-Hand-Fixing project.
See repository LICENSE for details.

---

## Summary

This SDXL workflow provides **true lighting direction control** via the VIDIT-FAID ControlNet, which interprets depth maps as lighting information. Unlike Flux's prompt-based approach, SDXL can precisely control where light falls in your scene through depth map conditioning.

**Quick Start**: Set prompts → Load/generate depth map → Adjust ControlNet strength → Generate

For questions or issues, see Troubleshooting section or check the ComfyUI-Hand-Fixing repository.

# ComfyUI Generation and Upscaling Workflows

High-quality image generation workflows with hand fixing and intelligent upscaling.

---

## Recommended Workflow

### `Generate with Hand Fix and Upscale.json` **COMPLETE PIPELINE**

**Best for:** Professional quality images with correct hand anatomy and high-resolution detail

**What it does:**
1. **Generate** base image at 1152x896
2. **Fix hands** on low-res image (before upscale) - 16x faster than post-upscale
3. **Upscale 4x** using Ultimate SD Upscale with tiled img2img (~4608x3584)
4. **Enhance faces** on high-res image (after upscale) - maximum detail

**Why this workflow is better:**
- Fixes hands BEFORE upscaling (16x faster than post-upscale fixing)
- Uses BBOX-only mode for hands (no tight masks that preserve malformed anatomy)
- Specialized prompts for each stage (hand fixing, face enhancement, body detail)
- Empirically tested and optimized parameters
- Comprehensive documentation embedded in workflow

**Tested optimal parameters:**
- Hand Fixer: denoise 0.5, cfg 6.0, bbox_crop_factor 2.5
- Face Enhancer: denoise 0.35, cfg 4.0, guide_size 1024
- Ultimate SD Upscale: 1024x1024 tiles, 32px padding, denoise 0.28

**Key features:**
- **BBOX-only hand fixing**: Rectangular bounding boxes give model freedom to regenerate correct anatomy
 - NO SAM for hands (SAM creates "tight glove" masks that preserve malformed shapes)
 - Prevents hallucinating extra hands with pose preservation prompts
- **SAM for faces**: Precise facial boundaries (appropriate for face enhancement)
- **Prompt structure**: Base prompts + enhancement terms for upscaling
 - Generation uses base prompts only
 - Upscale merges base + enhancement terms (body part details, texture preservation)
 - Hand/Face fixers use specialized anatomical prompts
- **Body part enhancement**: Specific terms for navel, nipples, areola, skin texture
- **Texture preservation**: Face enhancement avoids over-smoothing

**Requirements:**
- ComfyUI Impact Pack (FaceDetailer, SAMLoader, UltralyticsDetectorProvider)
- YOLOv8 hand detector: `bbox/hand_yolov8s.pt`
- YOLOv8 face detector: `bbox/face_yolov8m.pt`
- SAM model: `sam_vit_b_01ec64.pth`
- Upscale model: `4x-UltraSharp.pth`

**See workflow notes** (embedded in JSON) for complete parameter rationale and testing history.

---

## Alternative Workflows

### `Generate with Ultimate SD Upscale.json`

**Best for:** When hands are already correct, just need upscaling

**What it does:**
- Generate at 1152x896
- Upscale 4x with Ultimate SD Upscale
- No hand/face fixing

**Use when:**
- Generated hands are already correct
- You want faster processing (no detailer overhead)
- Testing upscale settings

**Parameters:**
- 1024x1024 tiles, 32px padding
- seam_fix_mode: None (20 tiles, fast)
- denoise: 0.28, steps: 40

---

### `Generate with 2x2x Upscale (Mac Compatible).json`

**Best for:** Low VRAM systems (Mac, older GPUs)

**What it does:**
- Generate at base resolution
- Upscale 2x, then 2x again (total 4x)
- Lower memory usage than single 4x pass

**Use when:**
- Running out of VRAM with standard upscale
- Using Mac with limited GPU memory
- Need to process on 6-8GB VRAM

---

### `Generate with ControlNet Tile Upscale.json`

**Best for:** Experimental - tile-based ControlNet guidance

**What it does:**
- Uses ControlNet Tile model for guided upscaling
- Can preserve composition better than pure img2img

**Use when:**
- You want to try ControlNet-guided upscaling
- Standard upscale changes composition too much

---

### `Generate with Latent Upscale (template).json`

**Best for:** Template for latent-space upscaling experiments

**What it does:**
- Upscales in latent space before VAE decode
- Faster but lower quality than pixel-space upscaling

**Use when:**
- Creating custom workflows
- Experimenting with latent upscaling
- Need faster preview iterations

---

### `PuLID_Ultimate_SD_Upscale_SDXL.json` ⭐ **[HIGHEST QUALITY]**

**Best for:** Maximum quality identity-preserving upscaling with SDXL

**What it does:**
- Loads reference image with face
- Uses PuLID to preserve facial identity (weight 0.8)
- Uses Ultimate SD Upscale for maximum quality
- 4x upscale with proper upscale model + tiled SD enhancement

**Why this is better:**
- **Much higher quality** than latent upscaling
- Pixel-space upscaling preserves detail
- Per-tile SD denoising adds realistic texture
- PuLID ensures identity preservation even at high denoise
- Suitable for production/final output

**Use when:**
- You want the absolute best quality upscaling
- Maintaining facial identity is critical
- Processing time is acceptable (3-8 minutes)
- Final render/production output

**Requirements:**
- **Custom Node**: [PuLID_ComfyUI](https://github.com/cubiq/PuLID_ComfyUI)
- **Custom Node**: [Ultimate SD Upscale](https://github.com/ssitu/ComfyUI_UltimateSDUpscale)
- **Model (CRITICAL)**: `ip-adapter_pulid_sdxl_fp16.safetensors`
  - Download: https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors
  - Save to: `ComfyUI/models/pulid/`
  - **DO NOT** use `pulid_v1.1.safetensors` - incompatible format!
- **Upscale Model**: `4x-UltraSharp.pth`
  - Download: https://huggingface.co/Kim2091/UltraSharp/resolve/main/4x-UltraSharp.pth
  - Save to: `ComfyUI/models/upscale_models/`
- InsightFace and EVA CLIP models (auto-download on first run)

**Parameters:**
- PuLID weight: 0.8 (high for strong identity preservation)
- PuLID mode: "fidelity" for photorealism
- Ultimate SD Upscale: 4x, 1024x1024 tiles, 32px padding
- Denoise: 0.35 (adds detail while preserving structure)
- Steps: 30, CFG: 6.5
- Sampler: dpmpp_2m_sde, Scheduler: karras

**Processing time:** 3-8 minutes (vs. 30 seconds for latent upscaling)

---

### `Multi-ControlNet_4x_Upscale_with_PuLID_SDXL.json`

**Best for:** Fast identity-preserving img2img upscaling with SDXL

**What it does:**
- Loads a base image with face reference
- Uses PuLID to preserve facial identity during upscaling
- Applies ControlNet Tile for structural guidance
- Upscales 4x using **latent upscaling** (faster, lower quality)

**Use when:**
- You need faster iteration speed
- Testing prompts/parameters
- Identity preservation is more important than maximum detail
- VRAM is limited

**vs. PuLID_Ultimate_SD_Upscale:**
- **10-20x faster** but lower quality
- Latent upscaling vs. pixel-space tiled upscaling
- Good for iteration, not final output
- See comparison section below

**Requirements:**
- **Custom Node**: [PuLID_ComfyUI](https://github.com/cubiq/PuLID_ComfyUI)
- **Model (CRITICAL)**: `ip-adapter_pulid_sdxl_fp16.safetensors`
  - Download from: https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors
  - Save to: `ComfyUI/models/pulid/`
  - **DO NOT** use `pulid_v1.1.safetensors` - incompatible format!
- **ControlNet**: TTPLANET_Controlnet_Tile_realistic_v2_fp16.safetensors
- InsightFace and EVA CLIP models (auto-download on first run)

**Parameters:**
- PuLID weight: 0.7 (adjust 0.5-0.9 for identity strength)
- PuLID mode: "fidelity" for photorealism
- ControlNet Tile strength: 1.0 (full structural guidance)
- Upscale: 4x latent upscaling (fast)
- Denoise: 0.55 (high enough to enhance while preserving identity)

**Processing time:** ~30 seconds

---

### `PuLID_ControlNet_Tile_4x_Upscale_SDXL.json` ⭐ **[RECOMMENDED]**

**Best for:** Stable, memory-efficient identity-preserving 4x upscaling

**What it does:**
- Loads reference image with face
- Uses PuLID to preserve facial identity (weight 0.8)
- Applies ControlNet Tile for structural guidance
- Uses VAEEncodeTiled/VAEDecodeTiled for memory efficiency
- 4x latent upscaling with KSampler refinement

**Why use this over UltimateSDUpscale:**
- **More stable** - no tensor view size errors
- **Memory efficient** - tiled VAE encoding/decoding
- **Compatible with all image dimensions**
- Combines PuLID + ControlNet Tile guidance
- Faster than UltimateSDUpscale, better than pure latent

**Use when:**
- UltimateSDUpscale gives tensor errors
- You want stable, predictable upscaling
- Memory efficiency is important
- You need both identity and structure preservation

**Requirements:**
- **Custom Node**: [PuLID_ComfyUI](https://github.com/cubiq/PuLID_ComfyUI)
- **Model (CRITICAL)**: `ip-adapter_pulid_sdxl_fp16.safetensors`
  - Download from: https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors
  - Save to: `ComfyUI/models/pulid/`
  - **DO NOT** use `pulid_v1.1.safetensors` - incompatible format!
- **ControlNet**: TTPLANET_Controlnet_Tile_realistic_v2_fp16.safetensors
- InsightFace and EVA CLIP models (auto-download on first run)

**Parameters:**
- PuLID weight: 0.8 (strong identity preservation)
- PuLID mode: "fidelity" for photorealism
- ControlNet Tile strength: 0.7 (structural guidance)
- VAE tile size: 512px (memory efficient)
- Upscale: 4x nearest-exact latent
- Denoise: 0.55, Steps: 30, CFG: 5.0

**Processing time:** 2-5 minutes

---

## PuLID Workflow Comparison

### Quality vs. Speed Tradeoff

| Aspect | PuLID_Ultimate_SD_Upscale | PuLID_ControlNet_Tile ⭐ | Multi-ControlNet_Latent |
|--------|---------------------------|--------------------------|-------------------------|
| **Quality** | ⭐⭐⭐⭐⭐ Production | ⭐⭐⭐⭐ Excellent | ⭐⭐⭐ Good |
| **Speed** | 3-8 minutes | 2-5 minutes | ~30 seconds |
| **Stability** | ⚠️ Tensor errors possible | ✅ Very stable | ✅ Stable |
| **Detail** | Best texture/detail | Great texture/detail | Smooth, less detail |
| **Method** | Pixel upscale + tiled SD | Tiled VAE + ControlNet | Latent space upscale |
| **Best for** | Maximum quality | Reliable quality | Iteration/testing |
| **VRAM** | 8GB+ | 6-8GB | 6GB+ |

### Technical Differences

**PuLID_Ultimate_SD_Upscale (High Quality):**
```
Input Image (1024x1024)
  ↓ (PuLID applied to model)
  ↓ (4x-UltraSharp upscale model)
Upscaled (4096x4096)
  ↓ (divide into 16x 1024x1024 tiles)
Tile 1 → img2img denoise 0.35 → Enhanced
Tile 2 → img2img denoise 0.35 → Enhanced
... (all tiles processed)
  ↓ (blend tiles seamlessly)
Final (4096x4096) with realistic detail
```

**Multi-ControlNet_Latent (Fast):**
```
Input Image (1024x1024)
  ↓ (PuLID applied to model)
  ↓ (ControlNet Tile guidance)
  ↓ (encode to latent)
Latent (128x128 @ 4 channels)
  ↓ (nearest-neighbor scale 4x)
Latent (512x512 @ 4 channels)
  ↓ (KSampler denoise 0.55 - ONE pass)
  ↓ (decode to pixels)
Final (4096x4096) smooth but less detail
```

### When to Use Each

**Use PuLID_Ultimate_SD_Upscale when:**
- ✅ You need the absolute maximum quality
- ✅ Your image dimensions work without tensor errors
- ✅ Production/portfolio work
- ✅ You have 3-8 minutes to wait
- ✅ Stability isn't an issue for you

**Use PuLID_ControlNet_Tile when:** ⭐ **RECOMMENDED**
- ✅ You want excellent quality with guaranteed stability
- ✅ UltimateSDUpscale gives you tensor errors
- ✅ Memory efficiency is important
- ✅ You want ControlNet structural guidance
- ✅ 2-5 minutes is acceptable
- ✅ Best balance of quality, speed, and reliability

**Use Multi-ControlNet_Latent when:**
- ✅ Testing different prompts/settings
- ✅ Need quick iteration
- ✅ Preview quality is sufficient
- ✅ Limited VRAM (6GB)
- ✅ Speed is critical

### Quality Examples

**Texture detail:**
- Ultimate SD: Visible skin pores, fabric weave, hair strands
- Latent: Smooth skin, simplified fabric, blended hair

**Sharpness:**
- Ultimate SD: Sharp edges, clear detail
- Latent: Slightly soft, less micro-detail

**Realism:**
- Ultimate SD: Photographic quality
- Latent: Good but more "AI-generated" look

### Recommended Workflow

1. **First pass**: Use Multi-ControlNet_Latent for quick testing
   - Test PuLID weight (0.6, 0.7, 0.8, 0.9)
   - Test different prompts
   - Find best reference image
   - Takes 30 seconds per test

2. **Final render**: Use PuLID_ControlNet_Tile ⭐ **RECOMMENDED**
   - Once you have optimal parameters
   - Excellent quality with guaranteed stability
   - Takes 2-5 minutes
   - Best for most use cases

3. **Maximum quality** (optional): Use PuLID_Ultimate_SD_Upscale
   - Only if ControlNet Tile isn't quite enough
   - If you need the absolute best and no tensor errors
   - Takes 3-8 minutes
   - Worth it for critical portfolio pieces

---

## Prompt Structure Explained

### How Prompts Work in the Main Workflow

The "Generate with Hand Fix and Upscale" workflow uses a sophisticated prompt structure:

#### 1. Base Prompts (Generation Pass)
- **Node 77**: Base positive prompt (e.g., "1girl, portrait, detailed face, photorealistic...")
- **Node 80**: Base negative prompt (e.g., "worst quality, low quality...")
- Used for initial 1152x896 generation

#### 2. Enhancement Terms (Upscale Pass)
- **Node 78**: Enhancement positive terms
 ```
 same person, consistent identity, natural skin texture, visible pores,
 detailed navel, realistic belly button, detailed nipples, realistic areola texture,
 detailed skin pores, natural body skin...
 ```
- **Node 81**: Enhancement negative terms
 ```
 smooth plastic skin, airbrushed, flat navel, undefined areola,
 blurry nipples, artificial nipples...
 ```

#### 3. Merged Prompts (Upscale Pass)
- **Nodes 84, 85**: StringConcatenate merges base + enhancement
- **Nodes 47, 76**: CLIPTextEncode creates conditioning
- Fed to Ultimate SD Upscale

**Why this works:**
- **Consistency**: Same base prompts = same subject/scene
- **Refinement**: Enhancement terms add detail without changing composition
- **Body parts**: Specific anatomical terms guide upscaler to enhance details it might miss

#### 4. Specialized Prompts (Detailers)

**Hand Fixer** (nodes 106, 107):
- Positive: "correct hand anatomy, five fingers, preserve hand pose, correct existing hand, single hand..."
- Negative: "malformed hands, extra fingers, multiple hands, extra hands, wrong hand pose..."
- **Purpose**: Fix anatomy while preventing hallucinations

**Face Enhancer** (nodes 108, 109):
- Positive: "sharp facial features, detailed skin pores, natural skin texture, preserved texture..."
- Negative: "smooth skin, plastic skin, airbrushed, over-smoothed, beauty filter..."
- **Purpose**: Enhance detail without over-smoothing

---

## Obsolete Workflows (Removed)

The following approaches were tested and found to be inferior:

### MeshGraphormer-based workflows
- **Problem**: Creates tight "glove-like" masks that follow malformed hand contours
- **Result**: Enhances bad hands instead of regenerating correct anatomy
- **Replaced by**: FaceDetailer with BBOX-only mode (rectangular boxes)

If you need MeshGraphormer workflows for reference, see git history (commit before removal).

---

## Getting Started

### Quick Start (Recommended Path)

1. **Install requirements**:
 ```
 - ComfyUI Impact Pack (via ComfyUI Manager)
 - Download models (see requirements above)
 ```

2. **Load workflow**:
 - Open ComfyUI
 - Load `Generate with Hand Fix and Upscale.json`

3. **Configure base generation**:
 - Set your checkpoint model
 - Write your generation prompts (nodes 77, 80)
 - Generate initial image

4. **Review embedded notes**:
 - Workflow JSON contains comprehensive parameter documentation
 - Explains why each parameter is set to its value
 - Includes testing history and rationale

5. **Generate and iterate**:
 - First run may take longer (model loading)
 - Review hand fixing results
 - Adjust parameters if needed

### Understanding Parameters

The main workflow has **tested optimal parameters** that were empirically validated:

**Hand Fixer**:
- cfg=7.0, denoise=0.65 → Too aggressive, fought model's range
- cfg=5.5-6.0, denoise=0.6 → Hallucinated extra hands
- cfg=6.0, denoise=0.5 → Well-formed hands in context

**bbox_crop_factor**:
- 3.0 → Too much context, increased hallucination risk
- 2.5 → Optimal - sufficient arm/wrist context without excess space

See workflow notes for complete testing history.

---

## Troubleshooting

### Hands still malformed after fixing

**Check:**
1. Is Hand Fixer using BBOX-only mode? (no SAM connection)
2. Is denoise at 0.5? (lower = preserves malformed structure)
3. Is cfg at 6.0? (lower = less anatomical guidance)
4. Are hand-specific prompts loaded? (check nodes 106, 107)

**Try:**
- Different seed (some seeds work better)
- Increase steps to 40
- Check bbox_threshold (0.5 is usually good)

### Extra hands hallucinated

**This indicates over-regeneration**:
1. Check denoise - should be 0.5, not higher
2. Check cfg - should be 6.0
3. Verify anti-hallucination prompts (node 107 should include "multiple hands, extra hands...")
4. Check bbox_crop_factor - should be 2.5, not 3.0

### Face looks over-smoothed

**Face Enhancer may be too aggressive**:
1. Reduce denoise from 0.35 to 0.20-0.25
2. Check prompts include texture preservation terms
3. Consider removing Face Enhancer entirely if upscaler does well enough

### Body parts not detailed enough

**Enhancement terms may not be applied**:
1. Check nodes 78, 81 include body part terms (navel, nipples, areola)
2. Verify merge nodes (84, 85) are connecting properly
3. Check Ultimate SD Upscale is using merged prompts (nodes 47, 76)

### SAM cuts off facial hair / mouth

**SAM mask too tight for faces**:
1. Increase sam_dilation from 0 to 20-25 pixels (Face Enhancer node 102)
2. Or remove SAM from Face Enhancer entirely (BBOX-only like Hand Fixer)

### PuLID Model Loading Error

**Error: "Missing key(s) in state_dict" for PulidModelLoader**

This happens when using the wrong model file format. The error looks like:
```
RuntimeError: Error(s) in loading state_dict for IDEncoder:
Missing key(s) in state_dict: "body.0.weight", "body.0.bias", ...
```

**Root cause:** ComfyUI's PuLID implementation requires IPAdapter-converted models, not the original PuLID model files.

**Solution:**
1. **Download the correct model file:**
   - File: `ip-adapter_pulid_sdxl_fp16.safetensors`
   - URL: https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors
   - Save to: `ComfyUI/models/pulid/`

2. **DO NOT use these files** (they are incompatible):
   - `pulid_v1.1.safetensors` (original format)
   - `pulid_flux_v0.9.1.safetensors` (for Flux, not SDXL)

3. **Verify installation:**
   - Check that `ComfyUI/models/pulid/ip-adapter_pulid_sdxl_fp16.safetensors` exists
   - Workflow node 22 should load this file
   - Restart ComfyUI after downloading

4. **If still failing:**
   - Verify PuLID_ComfyUI custom node is installed: `ComfyUI/custom_nodes/PuLID_ComfyUI`
   - Check file size: ~1.35 GB (incomplete download if smaller)
   - Try re-downloading the model

**Alternative models:**
- Main source: https://huggingface.co/huchenlei/ipadapter_pulid
- Backup: https://huggingface.co/Runzy/ip-adapter_pulid

---

## Workflow Customization

### Adapting for Your Needs

The main workflow can be customized:

**Change resolution:**
- Modify EmptyLatentImage dimensions (node for generation)
- Upscaling will still work (scales based on input)

**Change upscale factor:**
- Modify Ultimate SD Upscale upscale_by parameter
- Adjust tile_padding proportionally (32px for 4x, 24px for 3x, etc.)

**Skip hand fixing:**
- Disconnect Hand Fixer output
- Connect VAEDecode directly to Ultimate SD Upscale

**Skip face enhancement:**
- Disconnect Face Enhancer output
- Connect Ultimate SD Upscale directly to SaveImage

**Add your own detailers:**
- Copy FaceDetailer node structure
- Add YOLO detector for your target (e.g., body parts)
- Create specialized prompts
- Insert before or after existing detailers

---

## Performance Tips

**Faster generation:**
- Reduce steps (35 → 25 for testing)
- Use smaller tiles (1024 → 768)
- Skip face enhancement
- Use 2x2x workflow for low VRAM

**Higher quality:**
- Increase steps (35 → 40-50)
- Use larger tiles (1024 → 1536, needs more VRAM)
- Add seam_fix_mode (but slower: 20 tiles → 49 tiles)
- Increase upscale denoise slightly (0.28 → 0.32)

**Memory optimization:**
- Use 2x2x Upscale workflow
- Reduce tile size
- Lower guide_size in FaceDetailer
- Process fewer regions simultaneously

---

## Additional Resources

**Main documentation:**
- Workflow embedded notes (open JSON, check `extra.workflow_notes`)
- Git commit history for testing details

**Required downloads:**
- Hand detector: https://huggingface.co/Bingsu/adetailer/resolve/main/hand_yolov8s.pt
- Face detector: https://huggingface.co/Bingsu/adetailer/resolve/main/face_yolov8m.pt
- SAM model: https://huggingface.co/spaces/abhishek/StableSAM/resolve/main/sam_vit_b_01ec64.pth
- 4x-UltraSharp: https://huggingface.co/Kim2091/UltraSharp/resolve/main/4x-UltraSharp.pth

**ComfyUI nodes:**
- Impact Pack: https://github.com/ltdrdata/ComfyUI-Impact-Pack
- Install via ComfyUI Manager (search "Impact Pack")

---

## Version History

**Current version** (2024-11):
- FaceDetailer-based hand fixing with BBOX-only mode
- Tested optimal parameters (cfg 6.0, denoise 0.5)
- Specialized prompts for each stage
- Body part enhancement in upscale pass
- Comprehensive documentation

**Previous approaches** (deprecated):
- MeshGraphormer-based hand fixing (tight mask problem)
- Post-upscale hand fixing (16x slower)
- Generic prompts for all stages

---

Good luck creating amazing images! 

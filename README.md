# ComfyUI Workflows Collection

**Production-ready ComfyUI workflows with documented parameters, empirical testing, and optimization rationale**

This repository contains tested ComfyUI workflows for image generation and enhancement, with comprehensive documentation explaining parameter choices, testing results, success/failure rates, and lessons learned.

---

## Philosophy

**Why this repository exists:**

Most workflow collections show you *what* to do, but not *why*. This repository documents:

- ✅ **Parameter rationale** - Why each value is set (not just "set denoise to 0.5")
- ✅ **Testing history** - What we tried, what worked, what failed
- ✅ **Empirical results** - Success rates, processing times, quality comparisons
- ✅ **Lessons learned** - Common mistakes, optimization insights, troubleshooting

**Example:** Instead of "set cfg=6.0", you'll find:
> "cfg=7.0, denoise=0.65 → Too aggressive, fought model's range
> cfg=5.5-6.0, denoise=0.6 → Hallucinated extra hands
> cfg=6.0, denoise=0.5 → Well-formed hands in context ✓"

This is a living document - workflows and documentation evolve based on testing and community feedback.

---

## Workflow Categories

### 🎨 **Generation & Upscaling Pipelines**

Complete workflows from generation to high-resolution output:

| Workflow | Type | Output | Key Features |
|----------|------|--------|--------------|
| [Generate with Hand Fix and Upscale](workflows#generate-with-hand-fix-and-upscalejson-complete-pipeline) | **Complete Pipeline** | 4608x3584 | Generation → Hand Fix → Face Enhance → 4x Upscale |
| [Generate with Ultimate SD Upscale](workflows#generate-with-ultimate-sd-upscalejson) | Generation + Upscale | 4x output | Tiled SD upscaling, high quality |
| [Generate with ControlNet Tile Upscale](workflows#generate-with-controlnet-tile-upscalejson) | ControlNet-guided | 4x output | Structure-preserving upscale |
| [Generate with 2x2x Upscale (Mac Compatible)](workflows#generate-with-2x2x-upscale-mac-compatiblejson) | Multi-stage | 4x output | Two-pass 2x upscaling, low VRAM |
| [Generate with Latent Upscale (template)](workflows#generate-with-latent-upscale-templatejson) | Template | Variable | Fast latent space upscaling |

### 🆔 **Identity Preservation**

Workflows for maintaining facial identity during generation/upscaling:

| Workflow | Model | Use Case | Requirements |
|----------|-------|----------|--------------|
| [Multi-ControlNet 4x Upscale with PuLID SDXL](workflows#multi-controlnet_4x_upscale_with_pulid_sdxljson) | SDXL | Identity-preserving img2img upscaling | PuLID_ComfyUI, ip-adapter_pulid_sdxl_fp16.safetensors |

### 🖐️ **Specialized Enhancement**

Targeted fixes for specific issues:

| Workflow | Purpose | Success Rate | Documented Issues |
|----------|---------|--------------|-------------------|
| [hand_fix_bbox_inpaint](workflows/) | Hand correction | 70-90% | See [HAND_FIXING_GUIDE.md](HAND_FIXING_GUIDE.md) |

---

## Quick Start

### 1. Choose a Workflow

**For complete pipeline (generation to high-res):**
- Use: `Generate with Hand Fix and Upscale.json`
- Includes: Generation, hand fixing, face enhancement, 4x upscaling
- Best for: Production-quality outputs

**For upscaling existing images:**
- Use: `Generate with Ultimate SD Upscale.json` (no hand/face fixing)
- Or: `Multi-ControlNet_4x_Upscale_with_PuLID_SDXL.json` (with identity preservation)

**For low VRAM systems (Mac M-series):**
- Use: `Generate with 2x2x Upscale (Mac Compatible).json`
- Lower memory requirements per pass

### 2. Read the Documentation

Each workflow has embedded notes explaining:
- Why parameters are set to specific values
- What was tested and what worked/failed
- Common issues and solutions

**In ComfyUI:** Open workflow → Check Note nodes for documentation

**In repository:** Read [workflows/README.md](workflows/README.md) for comprehensive guides

### 3. Install Requirements

See [Prerequisites](#prerequisites) below for required custom nodes and models.

### 4. Run and Iterate

- First run may take longer (model loading)
- Review results and adjust parameters
- Check embedded notes for optimization tips

---

## Documentation Structure

### Workflow Documentation ([workflows/README.md](workflows/README.md))

**Detailed information for each workflow:**
- What it does and when to use it
- Required models and custom nodes
- Parameter explanations with testing history
- Troubleshooting common issues
- Performance tips and optimization

### Specialized Guides

**In-depth guides for specific topics:**

| Guide | Focus | Audience |
|-------|-------|----------|
| [HAND_FIXING_GUIDE.md](HAND_FIXING_GUIDE.md) | 4 different hand-fixing methods with comparisons | Anyone struggling with hand generation |
| [FIXING_HAND_ANATOMY.md](FIXING_HAND_ANATOMY.md) | Fixing wrong finger count and structural issues | Advanced users, anatomical problems |
| [INSTALLATION_TROUBLESHOOTING.md](INSTALLATION_TROUBLESHOOTING.md) | Installation help and common errors | Setup and troubleshooting |

### Embedded Workflow Notes

**Inside each workflow JSON:**
- Parameter rationale (why values are chosen)
- Testing history (what was tried)
- Optimization insights
- Common pitfalls

---

## Key Workflows Explained

### Generate with Hand Fix and Upscale ⭐ **[RECOMMENDED]**

**Complete production pipeline from generation to final high-res output**

**What it does:**
1. Generate base image (1152x896)
2. Fix hands before upscaling (16x faster than post-upscale)
3. Upscale 4x using Ultimate SD Upscale (~4608x3584)
4. Enhance faces after upscaling (maximum detail)

**Why this workflow is better:**
- Fixes hands BEFORE upscaling (much faster)
- BBOX-only mode for hands (no tight masks that preserve malformed anatomy)
- Specialized prompts for each stage
- Empirically tested optimal parameters

**Tested parameters:**
```
Hand Fixer: denoise=0.5, cfg=6.0, bbox_crop_factor=2.5
  → cfg=7.0 was too aggressive, cfg=5.5 hallucinated extra hands

Face Enhancer: denoise=0.35, cfg=4.0
  → Higher denoise over-smoothed skin texture

Ultimate SD Upscale: denoise=0.28, tile_size=1024
  → denoise=0.35 changed composition too much
```

See [workflows/README.md](workflows/README.md#generate-with-hand-fix-and-upscalejson-complete-pipeline) for complete documentation.

### Multi-ControlNet 4x Upscale with PuLID SDXL

**Identity-preserving img2img upscaling**

**What it does:**
- Loads reference image with face
- Uses PuLID to preserve facial identity during upscaling
- Applies ControlNet Tile for structural guidance
- Upscales 4x with high denoise while maintaining identity

**Why it's useful:**
- Maintain specific person's appearance during img2img
- High denoise values without losing identity
- Upscale photos while preserving the person

**Critical requirement:**
- Must use `ip-adapter_pulid_sdxl_fp16.safetensors` (IPAdapter format)
- NOT `pulid_v1.1.safetensors` (incompatible format causes loading errors)
- See [workflows/README.md - PuLID troubleshooting](workflows/README.md#pulid-model-loading-error)

---

## Prerequisites

### Required Software

- **ComfyUI** (latest version recommended)
- **Python 3.10+** (included with ComfyUI)
- **ComfyUI Manager** (highly recommended)
- **8GB+ VRAM** (6GB for basic workflows, 10GB+ for advanced)

### Essential Custom Nodes

Install via ComfyUI Manager:

1. **ComfyUI Impact Pack** - Hand/face detection, FaceDetailer
2. **Ultimate SD Upscale** - High-quality tiled upscaling
3. **ControlNet Auxiliary Preprocessors** - Various preprocessors

### Workflow-Specific Requirements

**For hand fixing workflows:**
- Impact Pack nodes
- Hand YOLO detector: `hand_yolov8s.pt`
- Face YOLO detector: `face_yolov8m.pt`
- SAM model: `sam_vit_b_01ec64.pth`

**For PuLID workflows:**
- PuLID_ComfyUI custom node
- Model: `ip-adapter_pulid_sdxl_fp16.safetensors` ([download](https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors))
- InsightFace models (auto-download)
- EVA CLIP models (auto-download)

**For upscaling workflows:**
- Upscale model: `4x-UltraSharp.pth` or `RealESRGAN_x4plus.pth`
- ControlNet Tile model (for ControlNet Tile workflow)

**See [INSTALLATION_TROUBLESHOOTING.md](INSTALLATION_TROUBLESHOOTING.md) for download links and setup help**

---

## Installation

### Quick Install (Recommended)

1. Open ComfyUI Manager
2. Search and install:
   - "Impact Pack"
   - "Ultimate SD Upscale"
   - "ControlNet Auxiliary"
3. Download required models (see workflows/README.md)
4. Restart ComfyUI

### Manual Installation

If ComfyUI Manager doesn't work:
- See [INSTALLATION_TROUBLESHOOTING.md](INSTALLATION_TROUBLESHOOTING.md) for manual ZIP downloads
- Browser-based model downloads
- Verification steps

---

## Workflow Comparison

### Upscaling Methods

| Method | Quality | Speed | VRAM | Best For |
|--------|---------|-------|------|----------|
| **Ultimate SD Upscale** | ⭐⭐⭐⭐⭐ | Medium | 8GB+ | High-quality detail-preserving upscale |
| **ControlNet Tile** | ⭐⭐⭐⭐ | Slow | 8GB+ | Structure preservation, controlled changes |
| **2x2x Multi-stage** | ⭐⭐⭐⭐⭐ | Slow | 6GB | Maximum quality, low VRAM systems |
| **Latent Upscale** | ⭐⭐⭐ | Fast | 6GB | Quick previews, fast iteration |

### Hand Fixing Methods

| Method | Wrong Fingers | Blurry Hands | Speed | Complexity |
|--------|---------------|--------------|-------|------------|
| **Impact Pack + BBOX** | 70-80% | 80-85% | Fast | Easy |
| **MeshGraphormer** | 85-90% | 80-85% | Slow | Advanced |

See [HAND_FIXING_GUIDE.md](HAND_FIXING_GUIDE.md#method-comparison) for detailed comparison.

---

## Common Issues & Solutions

### Workflow has red nodes / Missing nodes

**Solution:**
1. Install missing custom nodes via ComfyUI Manager
2. Check [workflows/README.md](workflows/README.md) for workflow-specific requirements
3. See [INSTALLATION_TROUBLESHOOTING.md](INSTALLATION_TROUBLESHOOTING.md)

### PuLID model loading error

**Error:** "Missing key(s) in state_dict for IDEncoder"

**Solution:**
- Download correct model: `ip-adapter_pulid_sdxl_fp16.safetensors`
- DO NOT use `pulid_v1.1.safetensors` (incompatible format)
- See [workflows/README.md - PuLID troubleshooting](workflows/README.md#pulid-model-loading-error)

### Hands still malformed after fixing

**Solution:**
1. Check denoise level (should be 0.5 for hand fixing)
2. Verify BBOX-only mode (no SAM for hands)
3. Review anti-hallucination prompts
4. See [workflows/README.md - Hands troubleshooting](workflows/README.md#hands-still-malformed-after-fixing)

### Out of VRAM

**Solution:**
- Use "2x2x Upscale (Mac Compatible)" workflow
- Reduce tile size in Ultimate SD Upscale
- Lower guide_size in FaceDetailer

---

## Performance Tips

**Faster generation:**
- Reduce steps (35 → 25 for testing)
- Use smaller tiles (1024 → 768)
- Skip face enhancement for testing
- Use latent upscale for quick previews

**Higher quality:**
- Increase steps (35 → 40-50)
- Use larger tiles (1024 → 1536, needs more VRAM)
- Add seam_fix_mode (slower but better)
- Increase upscale denoise (0.28 → 0.32)

**Memory optimization:**
- Use 2x2x multi-stage workflow
- Reduce tile size
- Lower guide_size in detailers
- Process fewer regions simultaneously

---

## Contributing

Found better parameters? Improved a workflow? Share your findings!

This repository thrives on empirical testing and community feedback. Contributions welcome:

- Better parameter settings with testing results
- New workflows with documented optimization
- Improved troubleshooting solutions
- Success/failure rate data

---

## Resources

### Model Downloads

**Essential:**
- [Hand/Face YOLO detectors](https://huggingface.co/Bingsu/adetailer)
- [SAM models](https://github.com/facebookresearch/segment-anything#model-checkpoints)
- [4x-UltraSharp upscaler](https://huggingface.co/Kim2091/UltraSharp)

**PuLID:**
- [ip-adapter_pulid_sdxl_fp16.safetensors](https://huggingface.co/huchenlei/ipadapter_pulid/resolve/main/ip-adapter_pulid_sdxl_fp16.safetensors) (REQUIRED - correct format)

**ControlNet:**
- [ControlNet models](https://huggingface.co/lllyasviel/ControlNet-v1-1)
- [ControlNet Tile](https://huggingface.co/lllyasviel/ControlNet-v1-1/resolve/main/control_v11f1e_sd15_tile.pth)

### Custom Nodes

- [Impact Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack)
- [Ultimate SD Upscale](https://github.com/ssitu/ComfyUI_UltimateSDUpscale)
- [ControlNet Aux](https://github.com/Fannovel16/comfyui_controlnet_aux)
- [PuLID_ComfyUI](https://github.com/cubiq/PuLID_ComfyUI)

---

## License

This repository contains workflows and guides for ComfyUI. Please refer to individual projects for their licenses:

- [ComfyUI License](https://github.com/comfyanonymous/ComfyUI)
- Custom nodes: See individual repositories

---

## Acknowledgments

Special thanks to the developers of:
- **ltdrdata** - ComfyUI Impact Pack
- **ssitu** - Ultimate SD Upscale
- **Fannovel16** - ControlNet Auxiliary Preprocessors
- **cubiq** - PuLID_ComfyUI

And the entire ComfyUI community for testing and feedback.

---

## Support

If you find these workflows helpful:
- ⭐ Star this repository
- 📢 Share with the ComfyUI community
- 💬 Provide feedback and testing results
- 🤝 Contribute improvements

---

**Good luck with your ComfyUI workflows!** 🎨

*For detailed workflow documentation, see [workflows/README.md](workflows/README.md)*
*For hand fixing specifically, see [HAND_FIXING_GUIDE.md](HAND_FIXING_GUIDE.md)*

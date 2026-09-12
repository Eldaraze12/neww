# FaceFusion NSFW/Content Analysis Reference Report

## Summary
- **NSFW models**: nsfw_1, nsfw_2, nsfw_3
- **Status**: NSFW detection is currently DISABLED (see content_analyser.py line 133-136, 194)
- **Total files importing content_analyser**: 16 files

---

## 1. CORE NSFW/CONTENT ANALYSER MODULE

### [facefusion/content_analyser.py](facefusion/content_analyser.py)

**Main Functions:**
- `analyse_image(image_path)` - Line 161-164: Analyzes single image for NSFW content
  - Returns bool: True if NSFW content detected
  - Uses LRU cache (@lru_cache decorator)
  
- `analyse_video(video_path, trim_frame_start, trim_frame_end)` - Line 167-190: Analyzes video frames
  - Returns bool: True if NSFW content detected in >10% of sampled frames
  - Checks frames at 1-second intervals (based on video FPS)
  - Uses progress bar for analysis feedback
  - Uses LRU cache

- `analyse_stream(vision_frame, video_fps)` - Line 149-154: Real-time stream analysis
  - Checks frames every 1 second based on FPS
  
- `analyse_frame(vision_frame)` - Line 157-158: Single frame analysis

- `detect_nsfw(vision_frame)` - Line 193-195: **CURRENTLY DISABLED** - Always returns False
  - Comment: "NSFW detection disabled"
  
- `detect_with_nsfw_1(vision_frame)` - Line 198-201: Detection using nsfw_1 model
  - Threshold: > 0.2 score
  
- `detect_with_nsfw_2(vision_frame)` - Line 205-208: Detection using nsfw_2 model
  - Threshold: > 0.25 score
  
- `detect_with_nsfw_3(vision_frame)` - Line 212-215: Detection using nsfw_3 model
  - Threshold: > 10.5 score

- `forward_nsfw(vision_frame, model_name)` - Line 219-231: Runs inference on model
  - Uses inference pool to get ONNX model
  - Thread-safe with conditional_thread_semaphore()

- `pre_check()` - Line 145-147: **DISABLED MODEL LOADING**
  - Returns True (models not actually downloaded due to commented-out code lines 134-136)
  - Lines 133-136 are commented out - NSFW model downloads disabled

**NSFW Models Defined (Lines 20-107):**
1. **nsfw_1** (Lines 23-50): EraX model (Apache-2.0, 2024)
   - Files: nsfw_1.hash, nsfw_1.onnx
   - Size: 640x640
   
2. **nsfw_2** (Lines 51-78): Marqo model (Apache-2.0, 2024)
   - Files: nsfw_2.hash, nsfw_2.onnx
   - Size: 384x384
   
3. **nsfw_3** (Lines 79-106): Freepik model (MIT, 2025)
   - Files: nsfw_3.hash, nsfw_3.onnx
   - Size: 448x448

**Model Download Configuration (Lines 33-44, 61-72, 89-100):**
- All models stored in `.assets/models/`
- Downloaded from 'models-3.3.0' release
- Hash verification included

**Disabled Code (Lines 133-136):**
```python
# NSFW detection disabled
# for content_analyser_model in [ 'nsfw_1', 'nsfw_2', 'nsfw_3' ]:
#     model_hash_set[content_analyser_model] = model_set.get(content_analyser_model).get('hashes').get('content_analyser')
#     model_source_set[content_analyser_model] = model_set.get(content_analyser_model).get('sources').get('content_analyser')
```

---

## 2. FILES THAT CALL analyse_image/analyse_video

### [facefusion/workflows/image_to_image.py](facefusion/workflows/image_to_image.py)
- Line 6: `from facefusion.content_analyser import analyse_image`
- Line 38: `if analyse_image(state_manager.get_item('target_path')): return 3`
  - **Purpose**: Checks if target image contains NSFW content during setup
  - **Return Code**: Returns error code 3 if NSFW detected
  - **Called in**: setup() function

### [facefusion/workflows/image_to_video.py](facefusion/workflows/image_to_video.py)
- Line 11: `from facefusion.content_analyser import analyse_video`
- Line 47: `if analyse_video(state_manager.get_item('target_path'), trim_frame_start, trim_frame_end): return 3`
  - **Purpose**: Checks if target video contains NSFW content during setup
  - **Return Code**: Returns error code 3 if NSFW detected
  - **Called in**: setup() function

---

## 3. FILES THAT IMPORT content_analyser MODULE

### [facefusion/core.py](facefusion/core.py)
- Line 8: `from facefusion import benchmarker, cli_helper, content_analyser, ...`
- Line 112: `content_analyser,` - List in common_modules
- Line 121-124: **Hash Validation** (Lines 121-124)
  ```python
  content_analyser_content = inspect.getsource(content_analyser).encode()
  content_analyser_hash = hash_helper.create_hash(content_analyser_content)
  return all(module.pre_check() for module in common_modules) and content_analyser_hash == 'b14e7b92'
  ```
  - **Purpose**: Calls content_analyser.pre_check() as part of common_pre_check()
  - **Validates**: Content_analyser code hasn't been modified (hash check)
  
- Line 137: `content_analyser,` - List in common_modules for force_download()
  - **Purpose**: Iterates through modules to force download models

### [facefusion/benchmarker.py](facefusion/benchmarker.py)
- Line 9: `from facefusion import content_analyser, core, state_manager`
- Line 65: `content_analyser.analyse_image.cache_clear()`
  - **Purpose**: Clears LRU cache for analyse_image in cold benchmark mode
  - **Called in**: cycle() function (line 60-68)
  
- Line 66: `content_analyser.analyse_video.cache_clear()`
  - **Purpose**: Clears LRU cache for analyse_video in cold benchmark mode

### [facefusion/uis/components/download.py](facefusion/uis/components/download.py)
- Line 6: `from facefusion import content_analyser, face_classifier, ...`
- **Purpose**: Imported for UI component management, likely for download progress tracking
- Line 6 is only import - no direct usage shown

### [facefusion/uis/components/execution.py](facefusion/uis/components/execution.py)
- Line 5: `from facefusion import content_analyser, face_classifier, ...`
- **Purpose**: Imported for UI component management
- Line 5 is only import - no direct usage shown

---

## 4. PROCESSOR MODULES THAT IMPORT content_analyser

All processor modules import content_analyser as part of common module list:

### [facefusion/processors/modules/age_modifier/core.py](facefusion/processors/modules/age_modifier/core.py)
- Line 10: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/face_editor/core.py](facefusion/processors/modules/face_editor/core.py)
- Line 10: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/lip_syncer/core.py](facefusion/processors/modules/lip_syncer/core.py)
- Line 9: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/face_debugger/core.py](facefusion/processors/modules/face_debugger/core.py)
- Line 8: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/expression_restorer/core.py](facefusion/processors/modules/expression_restorer/core.py)
- Line 10: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/frame_enhancer/core.py](facefusion/processors/modules/frame_enhancer/core.py)
- Line 9: `from facefusion import config, content_analyser, inference_manager, ...`

### [facefusion/processors/modules/deep_swapper/core.py](facefusion/processors/modules/deep_swapper/core.py)
- Line 11: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/frame_colorizer/core.py](facefusion/processors/modules/frame_colorizer/core.py)
- Line 10: `from facefusion import config, content_analyser, inference_manager, ...`

### [facefusion/processors/modules/background_remover/core.py](facefusion/processors/modules/background_remover/core.py)
- Line 10: `from facefusion import config, content_analyser, inference_manager, ...`

### [facefusion/processors/modules/face_enhancer/core.py](facefusion/processors/modules/face_enhancer/core.py)
- Line 8: `from facefusion import config, content_analyser, face_classifier, ...`

### [facefusion/processors/modules/face_swapper/core.py](facefusion/processors/modules/face_swapper/core.py)
- Line 11: `from facefusion import config, content_analyser, face_classifier, ...`

**Note**: These processor modules import content_analyser but don't show direct usage of analyse_image/analyse_video or pre_check() - likely included for consistency as part of common module handling.

---

## 5. TEST FILES

### [tests/test_inference_manager.py](tests/test_inference_manager.py)
- Line 6: `from facefusion import content_analyser, state_manager`
- Line 15: `content_analyser.pre_check()` - **Fixture setup**
  - Called in before_all() fixture (autouse=True)
  - Purpose: Pre-check before running inference tests
  
- Line 19: `model_names = [ 'nsfw_1', 'nsfw_2', 'nsfw_3' ]`
  - Defines test models
  
- Line 20: `_, model_source_set = content_analyser.collect_model_downloads()`
  - **Purpose**: Collects NSFW model download sources for testing
  - **Returns**: Model hash set and model source set
  
- Line 23: `get_inference_pool('facefusion.content_analyser', model_names, model_source_set)`
  - **Purpose**: Creates inference pool with NSFW models
  
- Line 25: `assert isinstance(INFERENCE_POOL_SET.get('cli').get('facefusion.content_analyser.nsfw_1.nsfw_2.nsfw_3.0.cpu').get('nsfw_1'), InferenceSession)`
  - Validates nsfw_1 model is properly loaded
  
- Line 30: Similar assertion for UI context
- Line 32: Asserts CLI and UI pools use same inference session

---

## 6. KEY FINDINGS

### ✓ NSFW Detection Status
- **CURRENTLY DISABLED**: Line 133-136 of content_analyser.py is commented out
- The detect_nsfw() function always returns False (line 194-195)
- No NSFW models are being downloaded or loaded

### ✓ Code Hash Validation
- core.py line 124 validates content_analyser code hash == 'b14e7b92'
- This check ensures no unauthorized modifications to NSFW detection

### ✓ Model Sources
- All NSFW models defined in content_analyser.py:
  - nsfw_1: EraX model (640x640)
  - nsfw_2: Marqo model (384x384)
  - nsfw_3: Freepik model (448x448)

### ✓ Error Code System
- When NSFW detected in analyse_image/analyse_video, returns error code 3
- Workflows check for this code and abort processing

### ✓ Cache Management
- Both analyse_image and analyse_video use @lru_cache for performance
- Benchmarker clears caches for cold runs

### ✓ Thread Safety
- forward_nsfw() uses conditional_thread_semaphore() for thread-safe inference

---

## 7. SUMMARY TABLE

| File | Line | Function | Purpose | Status |
|------|------|----------|---------|--------|
| content_analyser.py | 161-164 | analyse_image() | Image NSFW check | DISABLED |
| content_analyser.py | 167-190 | analyse_video() | Video NSFW check | DISABLED |
| content_analyser.py | 133-136 | pre_check() | Load models | COMMENTED OUT |
| content_analyser.py | 194-195 | detect_nsfw() | Detect NSFW | ALWAYS FALSE |
| image_to_image.py | 38 | setup() | Check image | CALLS analyse_image |
| image_to_video.py | 47 | setup() | Check video | CALLS analyse_video |
| core.py | 121-124 | common_pre_check() | Validate & init | CALLS pre_check |
| benchmarker.py | 65-66 | cycle() | Clear caches | COLD MODE |
| test_inference_manager.py | 15 | before_all() | Test setup | CALLS pre_check |

---

## 8. HOW TO RE-ENABLE NSFW DETECTION

To re-enable NSFW detection, uncomment lines 134-136 in content_analyser.py and modify detect_nsfw() to call one of the detection functions:

```python
# Current (disabled):
def detect_nsfw(vision_frame : VisionFrame) -> bool:
    # NSFW detection disabled
    return False

# To enable:
def detect_nsfw(vision_frame : VisionFrame) -> bool:
    # Choose one or combine multiple models:
    return detect_with_nsfw_1(vision_frame)  # or _2 or _3
```

And uncomment in pre_check():
```python
for content_analyser_model in [ 'nsfw_1', 'nsfw_2', 'nsfw_3' ]:
    model_hash_set[content_analyser_model] = model_set.get(content_analyser_model).get('hashes').get('content_analyser')
    model_source_set[content_analyser_model] = model_set.get(content_analyser_model).get('sources').get('content_analyser')
```

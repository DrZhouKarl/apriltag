# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AprilTag is a visual fiducial detection library written in C (C99 standard). It detects and identifies visual markers (tags) in images, primarily used in robotics and computer vision applications.

## Build Commands

```bash
# Standard release build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Install
cmake --build build --target install

# Debug build with AddressSanitizer
cmake -B build -GNinja -DCMAKE_BUILD_TYPE=Debug -DASAN=ON
cmake --build build

# Static libraries
cmake -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
cmake --build build
```

Key CMake options:
- `BUILD_SHARED_LIBS` (ON/OFF) — shared vs static
- `BUILD_EXAMPLES` (ON by default) — example executables
- `BUILD_PYTHON_WRAPPER` (ON by default) — Python bindings
- `BUILD_TESTING` (OFF by default) — enable tests
- `ASAN` (OFF by default) — AddressSanitizer for Debug builds

## Testing

```bash
# Enable and build tests
cmake -B build -DBUILD_TESTING=ON
cmake --build build

# Run all tests
cd build && ctest --verbose

# Run a specific test manually
./build/test_detection test/data/<image>.jpg
./build/test_tag_pose_estimation test/data/<image>.jpg test/data/<image>.txt
./build/test_quick_decode
```

Test data (images + ground truth `.txt` files) is in `test/data/`.

## Code Architecture

**Detection pipeline** (high level):
1. `apriltag_detector_detect()` in `apriltag.c` — entry point
2. `apriltag_quad_thresh.c` — quad boundary detection: image segmentation, union-find clustering, line fitting to find quadrilateral candidates
3. Back in `apriltag.c` — homography estimation, tag decoding with error correction
4. Optionally `apriltag_pose.c` — 3D pose estimation from homography

**Key source files:**
- `apriltag.h` / `apriltag.c` — public API and core detection logic
- `apriltag_quad_thresh.c` — the most complex component; quad detection via gradient/edge analysis
- `apriltag_pose.c` / `apriltag_pose.h` — pose estimation (orthogonal iteration)
- `tag*.c` / `tag*.h` — pre-generated tag family code tables (do not edit manually)
- `common/` — self-contained utility library: `matd` (matrix math), `image_u8` (image ops), `workerpool` (thread pool), `zarray`/`zhash` (containers), `homography`, `pjpeg` (JPEG decoder), etc.

**Main data structures** (defined in `apriltag.h`):
- `apriltag_detector_t` — detector config: `nthreads`, `quad_decimate`, `quad_sigma`, `refine_edges`, `decode_sharpening`
- `apriltag_detection_t` — result: `id`, `hamming`, `decision_margin`, corner points `p[4][2]`, center `c[2]`, homography `H`
- `apriltag_family_t` — tag family definition with codes, bit positions, hamming distance

**Recommended tag family:** `tagStandard41h12` (best balance of properties per README).

## Compiler Requirements

- C99 standard (`-std=c99`)
- Strict warnings: `-Wall -Wextra -Wpedantic -Werror`
- Links against `pthread` and `libm` on Unix
- No external runtime dependencies for core library; OpenCV only needed for the `opencv_demo` example

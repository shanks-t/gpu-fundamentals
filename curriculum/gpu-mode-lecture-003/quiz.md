# GPU Mode Lecture 003 — Conceptual Quiz

Purpose: build a precise mental model before running or modifying the code in `pmpp.ipynb`.

## Progress summary

Scores reflect demonstrated mastery of the prompt at the time it was completed; they are revision-friendly, not permanent judgments.

| Item | Type | Status | Attempts | Hints | Score | Gap tags | Next drill |
|---|---|---|---:|---:|---:|---|---|
| Q1 | Conceptual | Complete | 1 | 0 | 9 | tensor-layout | Re-derive channel offsets on a non-contiguous example. |
| Q2 | Conceptual | Complete | 1 | 0 | 6 | grid-block-thread | Distinguish a Python simulator from a CUDA launch. |
| Q3 | Conceptual | Complete | 1 | 0 | 8 | grid-block-thread | Explain block scheduling versus warp scheduling. |
| Q4 | Arithmetic/indexing | Complete | 1 | 1 | 6 | bounds-check | Practice ceiling division in Python and C++. |
| Q5 | Code reading | Complete | 1 | 0 | 6 | cuda-host-device | Trace shape, contiguity, and device transfer separately. |
| Q6 | Code reading | Complete | 1 | 0 | 8 | dtype-pointer-match | Explain wrapper validation from raw pointer through error check. |
| Q7 | Code reading | Complete | 1 | 0 | 8 | dtype-pointer-match | Contrast truncation with rounding to nearest. |
| Q8 | Performance | Complete | 1 | 0 | 10 | — | Retain for later review. |
| Q9 | Conceptual | Complete | 1 | 1 | 6 | matmul-work-count | Count outputs times work per output with a concrete matrix. |
| Q10 | Indexing | Complete | 1 | 1 | 7 | row-major-indexing | Map several 2-D blocks to output tiles. |
| Q11 | Broadcasting | In progress | 0 | 1 | — | broadcasting-shapes | Complete the shape drill in quiz.ipynb. |
| Q12–Q14 | Mixed | Pending | 0 | 0 | — | — | — |

## How we will use this file

- We will work one question at a time, from data layout through CUDA launch and performance.
- Each answer is recorded verbatim or lightly clarified, followed by feedback and any correction.
- Aim to explain *why* the code is shaped as it is, not merely recall syntax.

## Concept map to master

1. Image tensor shape, dtype, device, flattening, and contiguous memory.
2. RGB-to-grayscale indexing and the weighted luminance formula.
3. The meaning of a kernel; Python serial simulations versus actual GPU execution.
4. CUDA’s grid, blocks, threads, thread indices, bounds checks, warps, and SMs.
5. `load_inline`: Python → PyTorch C++ binding → CUDA compilation → callable extension.
6. Tensor validation, raw data pointers, output allocation, launch configuration, and error checks.
7. Matrix multiplication’s output mapping, indexing, 2-D launches, and work complexity.
8. Broadcasting/vectorized PyTorch versus Python loops, and what makes it faster on CPU.
9. Why the naïve CUDA matmul is correct but still far slower than `m1c @ m2c`.

## Quiz log

### Q1 — RGB layout and grayscale indexing

`img2` has shape `(c, h, w)` and contains RGB bytes. After `x = x.flatten()` and `n = h * w`, the notebook calculates pixel `i` as:

```python
0.2989*x[i] + 0.5870*x[i+n] + 0.1140*x[i+2*n]
```

Explain exactly why `i`, `i+n`, and `i+2*n` select the red, green, and blue values for the *same spatial pixel*. Include the assumed memory/layout order and the resulting output shape.

**Your answer:** After flattening the `(3, h, w)` image, the values are arranged channel by channel: all red pixel values, then all green values, then all blue values. For spatial pixel `i`, `x[i]` is its red value. Adding `n` skips the red channel to get that pixel's green value, and adding `2*n` skips both red and green to get its blue value.

**Feedback:** Correct. The useful precision is that `n = h*w` is the number of pixels in **one channel**, whereas the complete flattened RGB tensor has `3*n` elements. The result has shape `(h, w)`: one grayscale value per spatial pixel.

---

### Planned questions

Q2. Contrast `rgb2grey_py`, `rgb2grey_pyk`, and `rgb2grey_pybk`: which one runs in parallel, and what does each teach us?

**Your answer:** `rgb2grey_py` keeps all logic in one Python function. `rgb2grey_pyk` separates a kernel from a wrapper that calls it for each index. `rgb2grey_pybk` introduces GPU-oriented block and thread structure, with a wrapper receiving the block/thread configuration and calling the kernel for the mapped elements. This structure could be made parallel with CPU multiprocessing or, in CUDA, with threads.

**Feedback:** Strong structural explanation. Crucial correction: **none of the three Python versions executes in parallel as written.** `run_kernel` is a Python `for` loop; `blk_kernel` is nested Python `for` loops. The latter is a deliberately serial *simulation* of a CUDA launch so we can learn the mapping `i = blockidx * blockdim + threadidx` and the need for a final-block bounds check. A real CUDA launch later makes the logical threads eligible to execute concurrently on the GPU. The kernel itself handles one logical output index; the wrapper/simulator iterates over indices, blocks, and threads.

Q3. Define grid, block, thread, SM, and warp. Why does the notebook choose 256 threads per block and require a bounds check?

**Your answer:** A grid is a collection of blocks, and a block is a collection of threads. A group of threads is mapped to a streaming multiprocessor (SM). A warp is a collection of threads, believed to be 32 threads, providing an organizational/scheduling unit across SMs. The `i < n` guard is needed because ceiling division often launches more threads than data elements; with one data element per thread, extra threads must not access beyond flattened input/output bounds.

**Feedback:** Correct core model. More precisely: a **grid** is all blocks in one kernel launch; a **block** is the cooperating group scheduled onto one SM; a **thread** runs the kernel’s per-element instructions; an **SM** is GPU hardware that schedules and executes resident blocks/warps; and a **warp** is NVIDIA’s fixed 32-thread execution/scheduling group within a block. `256 = 8 × 32`, so it is a common warp-aligned block size that often offers good scheduling/resource trade-offs. It is not a promise that 256 threads run physically at the exact same instant. `cdiv(n, 256)` rounds up, so the final block has inactive excess threads and must guard before indexing.

Q4. Derive `i = blockIdx.x * blockDim.x + threadIdx.x` and explain what happens for a partially filled final block.

**Your answer:** `blocks = (n + threads - 1) / threads` gives 5 blocks for `n = 1000`, `threads = 256`, though four blocks produce 1,024 threads and are enough. There would be 24 inactive threads.

**Feedback:** The inactive-thread count is exactly right. The block count is **4**, because CUDA/C++ integer division truncates: `(1000 + 256 - 1) / 256 = 1255 / 256 = 4`. Block `b` owns the contiguous index range `b * 256` through `b * 256 + 255`; this is why its local thread offset is added to `blockIdx.x * blockDim.x`. Thus the final block (`blockIdx.x == 3`) covers `i = 768...1023`; `768...999` are active and `1000...1023` (24 threads) exit at the guard.

Q5. Walk the path from a Python CUDA source string to `module.rgb_to_grayscale(imgc)` executing on the GPU.

**Your answer:** `load_inline` creates C++ code from source strings, using a build system such as Ninja behind the scenes. `cuda_src` holds CUDA kernel code and `cpp_src` holds the C++ wrapper declaration. `__global__` indicates code invoked from host memory on the device. `<<<blocks, threads>>>` tells the GPU how many blocks and threads to create. `img.contiguous().cuda()` confirms the image is flattened and loaded onto the CUDA device.

**Feedback:** The main compilation/binding story is right. `load_inline` gives PyTorch C++/CUDA source strings, compiles them into a native extension (its build machinery may use Ninja), loads it, and exposes the named `functions` as Python-callable methods. `cuda_src` contains **both** the `__global__` kernel and the host-side C++ wrapper; `cpp_src` provides the wrapper’s C++ declaration for binding generation. `__global__` declares a CUDA kernel that the **host launches** but whose threads execute on the **GPU device**—it is not host-memory code. `<<<blocks, threads>>>` specifies the grid dimensions then block dimensions for that launch. Finally, `.contiguous()` ensures the original `(C,H,W)` tensor is packed in expected row-major contiguous storage; `.cuda()` copies that same-shaped tensor into GPU memory. Neither operation flattens it; the kernel calculates flat offsets from its raw pointer.

Q6. Explain the role of `CHECK_INPUT`, `.contiguous()`, `.cuda()`, `data_ptr<unsigned char>()`, `input.options()`, and `C10_CUDA_KERNEL_LAUNCH_CHECK()`.

**Your answer (partial):** `CHECK_INPUT` is a C++ macro wrapping `CHECK_CUDA` and `CHECK_CONTIGUOUS`, which in turn wrap PyTorch checks. `x.device().is_cuda()` verifies that `x` is a CUDA tensor—an n-dimensional array in device memory—and `x.is_contiguous()` checks that the data is contiguous in device memory.

**Your answer (continued):** `input.data_ptr<unsigned char>()` passes a pointer to the first address of the input array, typed as an 8-bit unsigned C++ character. `torch::empty({h,w}, input.options())` allocates space for the image's pixels, using the input's device (CUDA here). The braces appear to create a two-dimensional list from height and width.

**Feedback:** Correct core model. `data_ptr<unsigned char>()` returns a raw, device-accessible pointer to the first element; the template type must agree with the tensor dtype (`torch.uint8` here), so the kernel treats each element as one byte. `torch::empty` allocates an **uninitialized** output tensor: `{h, w}` is a C++ initializer list used as the shape argument (an `IntArrayRef`), meaning a 2-D tensor with dimensions `h` and `w`, not a nested list of pixel values. `input.options()` carries relevant tensor metadata—especially **device and dtype**, plus layout-related options—so output is CUDA `uint8`, matching the pointer type used by the kernel. Allocation is based on output shape; it does not depend on the input pointer.

**Your answer (final part):** `C10_CUDA_KERNEL_LAUNCH_CHECK()` is a PyTorch-provided macro that checks for errors after launching a kernel.

**Feedback:** Correct. It checks CUDA’s error state immediately after `<<<...>>>`, so invalid launch configuration, invalid device function, and related asynchronous CUDA errors are attributed to this kernel launch rather than only appearing during some later CUDA call. It reports the error; it does not make kernel execution synchronous by itself.

Q7. Why must the CUDA input and output types match the tensor dtype here? What numerical effect does assigning the weighted sum to `unsigned char` have?

**Your answer:** The pointer type needs to match the input tensor dtype because dtype determines how much contiguous memory each data element occupies. A mismatch would mean the kernel does not have the right memory allocation/interpretation. Assigning the floating-point grayscale result to the output converts it back to `uint8`, effectively rounding down to an integer.

**Feedback:** Correct, with a key refinement. Pointer type determines both the interpretation of bits **and pointer arithmetic**: `unsigned char*` advances one byte per `x[i]`; a `float*` advances four bytes and interprets four image bytes as one floating-point bit pattern. Thus the raw pointer type must match the tensor dtype; `input.options()` ensures the output tensor is also `uint8` for `unsigned char* out`. The weighted expression is evaluated in floating point, then conversion to `unsigned char` **truncates toward zero**. Since grayscale values here are nonnegative, that is equivalent to flooring, not rounding to nearest. The RGB range and near-unit weights keep normal image values in the representable 0–255 range.

Q8. Compare CPU timing and GPU timing fairly: which costs must be included or excluded depending on the question being asked?

**Your answer:** To measure the GPU kernel itself, exclude data loading to the device and transfer back to the host, isolating kernel execution. To measure a full CPU-image-to-CPU-result workflow, include both host-to-device and device-to-host transfers.

**Feedback:** Exactly. For kernel-only timing, keep input/output resident on the GPU, warm up the compiled kernel, and use CUDA events or explicit synchronization because GPU launches are ordinarily asynchronous. The notebook sets `CUDA_LAUNCH_BLOCKING=1`, which makes the host wait and simplifies its timing interpretation, but is not typical performance-mode behavior. For end-to-end timing, include H→D input copy, GPU computation, and D→H result copy (plus any required CPU-side setup), then compare against an equivalent CPU workflow. Compilation/loading costs are usually measured separately from steady-state inference/compute performance.

Q9. For `A[h, k] @ B[k, w]`, derive the output shape and the naïve operation count.

**Your answer:** `out[r,c]` computes the dot product of row `r` of `A` and column `c` of `B`, iterating over `i` up to `k - 1`. The total-count explanation needed help.

**Feedback:** Correct central operation. The loop begins at `i = 0`, not 1, and runs through `k - 1`, so it has exactly `k` iterations. For the count: output has `h*w` cells—`h` possible rows times `w` possible columns. Every one of those cells requires the same `k`-step dot product. Therefore total work is **number of outputs × work per output = `(h*w) * k = h*w*k`** multiply-accumulate iterations.

Q10. Explain the 2-D thread-to-output mapping for the Python and CUDA matmul kernels, including row-major flat indexing.

**Your answer (partial):** `out[r*w + c]` flattens the output into one dimension and assigns each dot product to its offset: row times width plus column identifies the location in a 1-D space.

**Feedback:** Correct. This is row-major indexing: each completed row contains `w` values, so `r*w` skips the preceding rows and `+c` selects the column in row `r`. The `r`/`c` coordinate mapping portion remains pending.

**Your answer (continued):** X coordinates map to output columns and Y coordinates map to output rows because the block is two-dimensional. The meaning of the block tile example was unclear.

**Feedback:** Correct. A 2-D block gives every thread an `(x,y)` location, and this kernel deliberately interprets x as horizontal column and y as vertical row. With `tpb = (16,16)`, block `(x=2, y=1)` begins at column `2*16 = 32` and row `1*16 = 16`. Its threads cover columns `32...47` and rows `16...31`—a 16×16 output tile, subject to the bounds guard at matrix edges.

Q11. Explain broadcasting in `a[i, :, None] * b`, all resulting dimensions, and why `.sum(dim=0)` yields one output row.

Q12. Why is that broadcasting formulation much faster than explicit Python loops on CPU, despite performing the same mathematical work?

Q13. Why is the simple CUDA matmul slower than PyTorch’s `@` in realistic cases, and what would tiling/shared memory change?

Q14. Contrast the 1-D and 2-D grayscale CUDA kernels: what changes and what stays conceptually identical?

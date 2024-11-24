# Video Filtering Example with WebGPU

## Overview
A WebGPU-based video filtering implementation of three different filters:
- No Filter (original video)
- Grayscale Filter
- Gaussian Blur Filter (7x7) with shared memory optimization (workgroup memory)
- Gaussian Blur Filter (7x7) without shared memory

## Implementation Details

### 1. Grayscale Filter
The grayscale filter converts RGB colors to grayscale using weighted color channels:
- **Approach**: Single-pass computation using weighted RGB values
- **Performance**: Efficient as it requires minimal computation per pixel
- **Key Features**:
  - Uses standard RGB to grayscale conversion weights
  - Preserves original alpha channel

### 2. Gaussian Blur Filter (Fast Implementation)
- **Optimization Techniques**:
  - Uses workgroup shared memory to reduce texture fetches
  - Implements 7x7 kernel using shared memory
  - Each thread loads multiple pixels into shared memory
- **Performance Benefits**:
  - Reduces global memory access
  - Reuses loaded pixels across multiple threads
  - Example of memory access reduction: From 49 global reads to ~4(up to 4) reads per thread

### 3. Gaussian Blur Filter (Slow Implementation)
- **Basic Approach**:
  - Direct texture sampling for each pixel in the 7x7 kernel
  - No memory optimization
  - Used as a baseline for performance comparison
- **Limitations**:
  - Higher number of texture fetches (49 reads per pixel)
  - More expensive global memory access

## Performance Analysis
| Filter Type | Global Memory Access Pattern |
|------------|----------------------|
| No Filter  | 1 read, 1 write     |
| Grayscale  | 1 read, 1 write     |
| Fast Gaussian | ~4 reads, 1 write per thread  |
| Slow Gaussian | 49 reads, 1 write per thread   |

## Technical Challenges and Solutions

1. **Shared Memory Management**
   - Challenge: Coordinating threads for shared memory access
   - Solution: Implemented synchronization b/w threads using `workgroupBarrier()`

2. **Border Handling**
   - Challenge: Managing edge cases in Gaussian blur
   - Solution: Implemented pixel address clamping at boundaries of texture
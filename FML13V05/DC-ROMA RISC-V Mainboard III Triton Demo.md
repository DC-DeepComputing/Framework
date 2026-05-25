# DC-ROMA RISC-V Mainboard III Triton Demo
### About Triton
Triton was originally an open-source domain-specific language and compiler framework for parallel programming created by OpenAI. It has now become the core tool for high-performance operator development in the PyTorch ecosystem. With a Python-native programming experience, it breaks through low-level hardware barriers — developers can write efficient deep neural network operators like FlashAttention without diving into low-level hardware details such as CUDA. The compiler automatically exploits the parallel potential of multiple hardware platforms including GPUs and RISC-V, balancing development efficiency with runtime performance.

### Significance of Triton on RISC-V
- Establishes an end-to-end PyTorch → Triton → RVV AI compilation chain, enabling native RISC-V support for large model inference with no CUDA dependency whatsoever
- Dramatically lowers the development barrier and cost of AI operators on RISC-V platforms
- Helps RISC-V become an integrated development platform for "general computing + AI"

## Deploying Triton on DC-ROMA RISC-V Mainboard III
### System Environment
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/8521478e-20b3-4d05-9eae-33c564a1de2c" />

### 1. Clone the Repository
```
git clone https://github.com/RuyiAI-Stack/triton-riscv.git triton-riscv

TRITON_RISCV_DIR="$(pwd)/triton-riscv"

git clone https://github.com/triton-lang/triton.git triton

cd triton && git checkout "$(cat "${TRITON_RISCV_DIR}/triton-hash.txt")"
```

### Apply Build Patches
```
cd ~/triton

"${TRITON_RISCV_DIR}/scripts/apply_patches.sh" "${TRITON_RISCV_DIR}/../triton"
```

### Build Buddy Compiler
- To build the Buddy Compiler, refer to: Getting Started
- Install dependencies:
```
# Install dependencies
sudo apt install flatbuffers-compiler libflatbuffers-dev libnuma-dev
```

- Clone the repository:
```
# Clone the repository
git clone git@github.com:buddy-compiler/buddy-mlir.git
cd buddy-mlir
git submodule update --init llvm
```
```
cd buddy-mlir
pip install -r requirements.txt
```

- Build and test LLVM/MLIR/CLANG:
```
# Build and test LLVM/MLIR/CLANG
cd buddy-mlir
mkdir llvm/build
cd llvm/build
cmake -G Ninja ../llvm \
    -DLLVM_ENABLE_PROJECTS="mlir;clang;openmp" \
    -DLLVM_TARGETS_TO_BUILD="host;RISCV" \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DOPENMP_ENABLE_LIBOMPTARGET=OFF \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DMLIR_ENABLE_BINDINGS_PYTHON=OFF

ninja -j2 check-clang check-mlir omp
```

- Build buddy-mlir:
```
# Build buddy-mlir
cd buddy-mlir
mkdir build
cd build
cmake -G Ninja .. \
    -DMLIR_DIR=$PWD/../llvm/build/lib/cmake/mlir \
    -DLLVM_DIR=$PWD/../llvm/build/lib/cmake/llvm \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DBUDDY_MLIR_ENABLE_PYTHON_PACKAGES=OFF
ninja
ninja check-buddy
```
### Create Virtual Environment
- Note: Python version must be Python 3.12 or 3.13.
- Compile Python 3.13:

```
# 1. Install build dependencies
sudo apt install build-essential libssl-dev libbz2-dev libreadline-dev libsqlite3-dev libffi-dev libffi-dev

# 2. Download and compile Python 3.13
cd /tmp
wget https://www.python.org/ftp/python/3.13.5/Python-3.13.5.tar.xz
tar xf Python-3.13.5.tar.xz
cd Python-3.13.5

./configure --prefix=/usr/local/python3.13 --enable-optimizations
make -j$(nproc)
sudo make install

```

- Create the environment:
```
/usr/local/python3.13/bin/python3.13 -m venv ~/triton-riscv/.venv --prompt triton-riscv
source .venv/bin/activate
```

### Load Environment and Build Triton
```
cd ~/triton-riscv

source .venv/bin/activate

export TRITON_DIR=/home/roma/triton
export BUDDY_DIR=/home/roma/buddy-mlir 
source scripts/triton-riscv-env.sh
#Install dependencies:
pip install pytest-xdist pybind11 setuptools
pip install torch --index-url https://community-ci.openruyi.cn/pypi/riscv64/dev/+simple/

cd ~/triton
pip install --no-build-isolation -e .
```

### Verify the Build
```
cd ~/triton-riscv

source .venv/bin/activate

export TRITON_DIR=/home/roma/triton
export BUDDY_DIR=/home/roma/buddy-mlir

source scripts/triton-riscv-env.sh

python -c "import triton; import triton.backends.triton_shared.compiler as c; print(triton.__version__); print(c.__file__)"
"$TRITON_SHARED_OPT_PATH" --version
Expected output:
3.4.0
/home/roma/triton-riscv/.venv/lib/python3.13/site-packages/triton/backends/triton_shared/compiler.py
LLVM (http://llvm.org/):
  LLVM version 22.0.0git
  Optimized build with assertions.
```

### Run Example Test Suite

```
pytest ../triton-riscv/python/examples/ \
    --ignore=../triton-riscv/python/examples/test_core.py \
    --ignore=../triton-riscv/python/examples/test_annotations.py \
    -v
```
- Example test output:
```
# Test example
(triton-riscv) roma@roma:~/triton-riscv$ pytest ../triton-riscv/python/examples/test_vec_add.py -v
==================================================================== test session starts ====================================================================
platform linux -- Python 3.13.5, pytest-9.0.3, pluggy-1.6.0 -- /home/roma/triton-riscv/.venv/lib/python3.13
cachedir: .pytest_cache
rootdir: /home/roma/triton-riscv
plugins: xdist-3.8.0
collected 1 item

python/examples/test_vec_add.py::test PASSED                                                                                                          [100%]

===================================================================== 1 passed in 5.10s =====================================================================
```


### Run Demo: Vector Addition + Matrix Multiplication


- Demo code:
```
"""Triton on RISC-V Demo – Vector Addition + Matrix Multiplication"""

import torch
import triton
import triton.language as tl
from triton.backends.triton_shared.driver import CPUDriver

# ── Step 1: Set up CPU driver (tell Triton to run on CPU) ──
triton.runtime.driver.set_active(CPUDriver())

# ── Step 2: Write a vector addition kernel ──
@triton.jit
def vec_add_kernel(x_ptr, y_ptr, out_ptr, n_elements, BLOCK_SIZE: tl.constexpr):
    pid = tl.program_id(0)
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
    mask = offsets < n_elements
    x = tl.load(x_ptr + offsets, mask=mask)
    y = tl.load(y_ptr + offsets, mask=mask)
    tl.store(out_ptr + offsets, x + y, mask=mask)

# ── Step 3: Write a matrix multiplication kernel ──
@triton.jit
def matmul_kernel(
    A, B, C, M, N, K,
    BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,
):
    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)
    offs_m = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_n = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    offs_k = tl.arange(0, BLOCK_K)
    a_ptrs = A + offs_m[:, None] * K + offs_k[None, :]
    b_ptrs = B + offs_k[:, None] * N + offs_n[None, :]
    acc = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
    for k in range(0, K, BLOCK_K):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        acc += tl.dot(a, b)
        a_ptrs += BLOCK_K
        b_ptrs += BLOCK_K * N
    c_ptrs = C + offs_m[:, None] * N + offs_n[None, :]
    tl.store(c_ptrs, acc)

# ── Step 4: Run vector addition ──
print("=" * 50)
print("Demo 1: Vector Add")
print("=" * 50)

N = 1024
x = torch.rand(N)
y = torch.rand(N)
out = torch.empty(N)

grid = lambda meta: (triton.cdiv(N, meta['BLOCK_SIZE']),)
vec_add_kernel[grid](x, y, out, N, BLOCK_SIZE=128)

print(f"x[:5]   = {x[:5]}")
print(f"y[:5]   = {y[:5]}")
print(f"x + y   = {(x + y)[:5]}")
print(f"Triton  = {out[:5]}")
print(f"Match:  {torch.allclose(out, x + y)}")

# ── Step 5: Run matrix multiplication ──
print("\n" + "=" * 50)
print("Demo 2: Matrix Multiplication")
print("=" * 50)

M, N_size, K_size = 64, 64, 64
A = torch.randn(M, K_size)
B = torch.randn(K_size, N_size)
C = torch.empty(M, N_size)

BLOCK = 16
grid = lambda meta: (
    triton.cdiv(M, BLOCK), triton.cdiv(N_size, BLOCK)
)
matmul_kernel[grid](A, B, C, M, N_size, K_size,
                    BLOCK_M=BLOCK, BLOCK_N=BLOCK, BLOCK_K=BLOCK)

torch_result = A @ B
print(f"Torch[0,0] = {torch_result[0, 0]:.4f}")
print(f"Triton[0,0] = {C[0, 0]:.4f}")
print(f"Match:  {torch.allclose(C, torch_result, atol=1e-3)}")

print("\n" + "=" * 50)
print("All demos complete! 🎉")
print("=" * 50)
```

- Run results:

```
(triton-riscv) roma@roma:~/triton-riscv$ python demo.py
==================================================
Demo 1: Vector Add
==================================================
x[:5]   = tensor([0.1691, 0.2057, 0.2796, 0.7611, 0.1705])
y[:5]   = tensor([0.0213, 0.7166, 0.8521, 0.5088, 0.7358])
x + y   = tensor([0.1904, 0.9223, 1.1317, 1.2699, 0.9063])
Triton  = tensor([0.1904, 0.9223, 1.1317, 1.2699, 0.9063])
Match:  True

==================================================
Demo 2: Matrix Multiplication
==================================================
Torch[0,0] = -1.4522
Triton[0,0] = -1.4522
Match:  True

==================================================
All demos complete! 🎉
==================================================
```
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/8e618da3-af5f-48e8-8ba4-02c4af784b54" />

- Activate Virtual Environment for Each Use
```
cd ~/triton-riscv
source .venv/bin/activate
export TRITON_DIR=/home/roma/triton
export BUDDY_DIR=/home/roma/buddy-mlir
source scripts/triton-riscv-env.sh
```
### FAQ
Building on RISC-V often encounters dependency issues. You can seek help from the community: https://github.com/RuyiAI-Stack/triton-riscv
### References
- https://github.com/RuyiAI-Stack/triton-riscv
- https://github.com/buddy-compiler/buddy-mlir?tab=readme-ov-file#getting-started

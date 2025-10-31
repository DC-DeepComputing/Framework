# DC-ROMA RISC-V AI PC, RISC-V Mainboard II NPU Memory Adjustment Instructions

 
System Declaration 
- Default NPU memory allocation is **6GiB**, which is insufficient for running **Deepseek-7B** on a single Die.  
- Requires expansion to **≥11GiB** to run Deepseek-7B.

- Kernel Memory Allocation Logs
```
[    0.000000] Reserved memory: created CMA memory pool at 0x0000002260000000, size 512 MiB
[    0.000000] OF: reserved mem: initialized node linux,cma, compatible id shared-dma-pool
[    0.000000] OF: reserved mem: 0x0000002260000000..0x000000227fffffff (524288 KiB) map reusable linux,cma
[    0.000000] OF: reserved mem: 0x0000000059000000..0x00000000593fffff (4096 KiB) nomap non-reusable sprammemory@59000000
[    0.000000] OF: reserved mem: 0x0000000079000000..0x00000000793fffff (4096 KiB) nomap non-reusable sprammemory@79000000
[    0.000000] OF: reserved mem: 0x0000000080000000..0x000000008007ffff (512 KiB) nomap non-reusable mmode_resv0@80000000
[    0.000000] OF: reserved mem: 0x00000000dffe0000..0x00000000dfffffff (128 KiB) nomap non-reusable lpcpures@dffe0000
[    0.000000] OF: reserved mem: 0x00000000e0000000..0x00000000e1ffffff (32768 KiB) nomap non-reusable region@e0000000
[    0.000000] OF: reserved mem: 0x00000000fff00000..0x00000000ffffffff (1024 KiB) nomap non-reusable ramoops@fff00000
[    0.000000] OF: reserved mem: 0x00000001fe000000..0x00000001ffffffff (32768 KiB) nomap non-reusable g2d_8GB_boundary_reserved_4k
[    0.000000] OF: reserved mem: 0x00000002fffff000..0x00000002ffffffff (4 KiB) nomap non-reusable g2d_12GB_boundary_reserved_4k
[    0.000000] Reserved memory: created mmz_nid_0_part_0 eswin reserve memory at 0x0000000300000000, size 6144 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_0_part_0, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x0000000300000000..0x000000047fffffff (6291456 KiB) nomap non-reusable mmz_nid_0_part_0
[    0.000000] OF: reserved mem: 0x0000002040000000..0x00000020403fffff (4096 KiB) nomap non-reusable nid_1_zero_device_simu@2040000000
[    0.000000] OF: reserved mem: 0x00000020e0000000..0x00000020e1ffffff (32768 KiB) nomap non-reusable region@20,e0000000
[    0.000000] OF: reserved mem: 0x00000020fffff000..0x00000020ffffffff (4 KiB) nomap non-reusable d1_g2d_4GB_boundary_reserved_4k
[    0.000000] OF: reserved mem: 0x00000021fe000000..0x00000021ffffffff (32768 KiB) nomap non-reusable d1_g2d_8GB_boundary_reserved_4k
[    0.000000] Reserved memory: created mmz_nid_1_part_0 eswin reserve memory at 0x0000002280000000, size 6144 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_1_part_0, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x0000002280000000..0x00000023ffffffff (6291456 KiB) nomap non-reusable mmz_nid_1_part_0
```
## NPU Memory Adjustment Steps

### For **32GiB DC-ROMA AI PC (RISC-V Mainboard II for Framework Laptop)** 

1. **U-Boot Configuration**
``` 
U-Boot 2024.01 (Oct 13 2025 - 03:38:17 +0000)
CPU: rv64imafdc_zba_zbb
Model: Deepcomputing FML13V03
DRAM: 16 GiB (effective 32 GiB)
```

2. **Die0 NPU Memory Adjustment**
```
# Execute in U-Boot:

#Constraint: start_address + memory_size = 0x480000000 (End of Die0 16GiB)
setenv fdt0_cmd "fdt mmz mmz_nid_0_part_0 <start_address> <memory_size>"
setenv bootcmd "run fdt0_cmd;${bootcmd}"
saveenv

#Example (11GiB):
setenv fdt0_cmd "fdt mmz mmz_nid_0_part_0 0x1c0000000 0x2c0000000"    
setenv bootcmd "run fdt0_cmd;${bootcmd}"
saveenv
```

- Post-Adjustment Kernel Log:

```
[    0.000000] Reserved memory: created mmz_nid_0_part_0@1,c0000000 eswin reserve memory at 0x00000001c0000000, size 11264 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_0_part_0@1,c0000000, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x00000001c0000000..0x000000047fffffff (11534336 KiB) nomap non-reusable mmz_nid_0_part_0@1,c0000000
```

3. **Die1 NPU Memory Adjustment**

```
#Execute in U-Boot:

#Constraint: start_address + memory_size = 0x2400000000 (End of Die1 16GiB)
setenv fdt1_cmd "fdt mmz mmz_nid_1_part_0 <start_address> <memory_size>"
setenv bootcmd "run fdt1_cmd;${bootcmd}"
saveenv 

#Example (11GiB):
setenv fdt1_cmd "fdt mmz mmz_nid_1_part_0 0x2140000000 0x2c0000000 "
setenv bootcmd "run fdt1_cmd;${bootcmd}"
saveenv 
```

- Post-Adjustment Kernel Log:

```
[    0.000000] Reserved memory: created mmz_nid_1_part_0@21,40000000 eswin reserve memory at 0x0000002140000000, size 11264 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_1_part_0@21,40000000, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x0000002140000000..0x00000023ffffffff (11534336 KiB) nomap non-reusable mmz_nid_1_part_0@21,40000000
```


## For **64GiB DC-ROMA AI PC (RISC-V Mainboard II for Framework Laptop)** 

1. **U-Boot Configuration**

```
U-Boot 2024.01 (Oct 13 2025 - 03:38:17 +0000)
CPU:   rv64imafdc_zba_zbb 
Model: Deepcomputing FML13V03 
DRAM:  32 GiB (effective 64 GiB)
```

2. **Die0 NPU Memory Adjustment**

```
#Execute in U-Boot:

#Constraint: start_address + memory_size = 0x880000000 (End of Die0 32GiB)
setenv fdt0_cmd "fdt mmz mmz_nid_0_part_0 <start_address> <memory_size>"
setenv bootcmd "run fdt0_cmd;${bootcmd}"
saveenv 

#Example (11GiB):
setenv fdt0_cmd "fdt mmz mmz_nid_0_part_0 0x5c0000000 0x2c0000000"
setenv bootcmd "run fdt0_cmd;${bootcmd}"
saveenv 
```

- Post-Adjustment Kernel Log:

```
[    0.000000] Reserved memory: created mmz_nid_0_part_0@5,c0000000 eswin reserve memory at 0x00000005c0000000, size 11264 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_0_part_0@5,c0000000, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x00000005c0000000..0x000000087fffffff (11534336 KiB) nomap non-reusable mmz_nid_0_part_0@5,c0000000
```

3. **Die1 NPU Memory Adjustment**

```
# Execute in U-Boot:

#Constraint: start_address + memory_size = 0x2800000000 (End of Die1 32GiB)
setenv fdt1_cmd "fdt mmz mmz_nid_1_part_0 <start_address> <memory_size>"
setenv bootcmd "run fdt1_cmd;${bootcmd}"
saveenv

#Example (11GiB):
setenv fdt1_cmd "fdt mmz mmz_nid_1_part_0 0x2540000000 0x2c0000000"
setenv bootcmd "run fdt1_cmd;${bootcmd}"
saveenv
```

- Post-Adjustment Kernel Log:
```
[    0.000000] Reserved memory: created mmz_nid_1_part_0@25,40000000 eswin reserve memory at 0x0000002540000000, size 11264 MiB
[    0.000000] OF: reserved mem: initialized node mmz_nid_1_part_0@25,40000000, compatible id eswin-reserve-memory
[    0.000000] OF: reserved mem: 0x0000002540000000..0x00000027ffffffff (11534336 KiB) nomap non-reusable mmz_nid_1_part_0@25,40000000
```

### Note: All address calculations use hexadecimal notation. Ensure alignment with physical memory boundaries to avoid conflicts.

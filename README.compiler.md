● What is arm-none-eabi-gcc?

  It's a cross-compiler — it runs on your x86_64 Linux machine but produces binaries for ARM processors. The name is a GCC target triple:

  arm  -  none  -  eabi
   │        │        └─ Embedded ABI (calling convention, struct layout, etc.)
   │        └─ no operating system (bare-metal)
   └─ ARM architecture


  Version installed: GCC 13.2.1 (Debian package 13.2.rel1-2).

  ---
  How it works

  It is a standard GCC with the backend configured to emit ARM machine code instead of x86. The full pipeline is the same: C/C++ → preprocessor →
  compiler → assembler → linker. The only difference is the output runs on an ARM CPU, not the host.

  Key flags that control what it emits:

  ┌──────────────┬──────────────────────────────────────────────────────────────────────────────────┐
  │     Flag     │                                     Meaning                                      │
  ├──────────────┼──────────────────────────────────────────────────────────────────────────────────┤
  │ -mcpu=       │ Target CPU (e.g. cortex-m33) — sets arch + tune together                         │
  ├──────────────┼──────────────────────────────────────────────────────────────────────────────────┤
  │ -march=      │ Target architecture (e.g. armv8-m.main)                                          │
  ├──────────────┼──────────────────────────────────────────────────────────────────────────────────┤
  │ -mthumb      │ Emit Thumb-2 ISA (compact 16/32-bit instructions, used by all Cortex-M)          │
  ├──────────────┼──────────────────────────────────────────────────────────────────────────────────┤
  │ -mfloat-abi= │ soft (software FP), softfp (HW FP registers, soft ABI), hard (HW FP all the way) │
  ├──────────────┼──────────────────────────────────────────────────────────────────────────────────┤
  │ -mfpu=       │ FPU type (e.g. fpv5-sp-d16 for CM33 single-precision)                            │
  └──────────────┴──────────────────────────────────────────────────────────────────────────────────┘

  ---
  Supported target platforms (multilib)

  The compiler ships pre-built runtime libraries (libgcc, libc, etc.) for each combination. The relevant ones for Pico:

  thumb/v6-m/nofp        ← RP2040 / Cortex-M0+  (pico, pico_w)
  thumb/v8-m.main/nofp   ← RP2350 / Cortex-M33  (pico2, pico2_w, no FPU)
  thumb/v8-m.main+fp/softfp  ← RP2350 with FPU, soft ABI
  thumb/v8-m.main+fp/hard    ← RP2350 with FPU, hard ABI


  The Pico SDK sets these flags automatically based on PICO_BOARD. For pico2_w it uses -mcpu=cortex-m33 -mthumb which maps to armv8-m.main.

  ---
  Why ARM_CM0 port runs (but isn't ideal) on RP2350

  The Cortex-M33 is a superset of Cortex-M0+. It executes all the same Thumb-2 instructions, so code compiled for armv6s-m (CM0+)
  runs on CM33 — it just misses CM33-specific features like:
  - Hardware divide instructions (CM33 has SDIV/UDIV)
  - DSP extensions
  - Optional TrustZone (not used on Pico 2)
  - Better FPU support

  This is exactly why RP2040-FreeRTOS-yyk compiles and runs on pico2_w with the ARM_CM0 FreeRTOS port — it works, but it's suboptimal. The correct
  port for RP2350 is ARM_CM33_NTZ (Non-TrustZone), which generates proper CM33 context-switch code.

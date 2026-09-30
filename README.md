# Crash Dump Decoder

Architecture-aware firmware crash analysis for Cortex-M and RISC-V targets.

The decoder turns raw exception frames, fault registers, stack bytes, frame-pointer memory, and ELF-derived symbols into actionable reports, backtraces, stack candidates, Markdown, and SARIF.

## Quick start

```bash
npm install
npm test
node src/cli.js examples/cortex-m-crash.json report.json \
  --markdown report.md --sarif report.sarif
```

```js
import {
  buildCrashReport, decodeCortexMFrame,
  decodeCortexMFaults, decodeRiscVTrap,
  unwindFramePointers
} from '@opencircuitlabs/crash-dump-decoder';

const report = buildCrashReport({
  architecture: 'cortex-m',
  frame,
  faultRegisters,
  symbols,
  stack,
  memory,
  framePointer
});
```

Symbol arrays can come directly from `elf-inspector` by mapping `value`, `size`, and `name` fields.

## Cortex-M decoding

- Basic exception stack frame: R0–R3, R12, LR, PC, and xPSR.
- IPSR exception names including external IRQ numbers.
- Thumb-state and stack-alignment flags.
- MemManage, BusFault, UsageFault, HardFault, and DebugFault status bits.
- Valid MMFAR/BFAR address gating.
- Symbolized PC and LR with Thumb-bit clearing.

## RISC-V decoding

RV32 and safe-range RV64 `mcause`, `mepc`, `mtval`, and `mstatus` values are supported. The interrupt bit and standard synchronous exception or interrupt code are decoded separately. RISC-V symbol lookup preserves address bit zero.

## Backtrace recovery

`MemoryReader` provides bounded reads from captured memory segments. Frame-pointer unwinding validates alignment, available memory, cycles, direction, and a configurable maximum depth. Every return address is symbolized.

When frame pointers are unavailable, stack scanning identifies unique words that fall within known symbols. Candidates retain their stack address and symbol offset and are explicitly not presented as a guaranteed call chain.

## Reports

The normalized JSON report includes architecture data, device/firmware identity, decoded exception, fault reasons, frame-pointer backtrace, stack candidates, and a concise summary. Markdown is suitable for issue attachments; SARIF creates a firmware-crash diagnostic for CI ingestion.

## License

MIT

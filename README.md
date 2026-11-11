# AXI Protocol Examples

Worked, annotated explanations of the **AMBA AXI protocols** — AXI4, AXI4-Lite, and AXI4-Stream —
focused on the part that actually confuses people: the **VALID/READY handshake** and the five
independent channels. Every SoC IP you integrate speaks AXI, so getting the handshake right once
pays off forever.

## Contents

| Doc | Covers |
|-----|--------|
| [The VALID/READY handshake](examples/01-valid-ready.md) | the one rule that governs every AXI channel |
| [AXI4-Lite](examples/02-axi4-lite.md) | the five channels, a register slave, byte strobes |
| [AXI4 vs AXI4-Lite vs AXI-Stream](examples/03-variants.md) | when to use which, bursts, and streaming |

## The 30-second model
AXI is **five independent channels**, each a point-to-point stream governed by one handshake rule:

```
Write: AW (address) ─┐
       W  (data)    ─┼─► slave ──► B (write response)
Read:  AR (address) ─┘        └──► R (read data + response)
```
Each channel transfers one beat when **both** VALID (from source) and READY (from destination) are
high on the same clock edge. That's the whole protocol's heartbeat.

## Why AXI feels hard at first
- The channels are **independent** — write-address and write-data can arrive in either order.
- VALID must **not wait** for READY (a source can't deadlock itself), but READY *may* wait for VALID.
- Once VALID is asserted, the payload must stay **stable** until the handshake completes.
Get those three rules and the rest is detail.

## Companion material
- Build one for real → [btech-vlsi-projects #05 (AXI4-Lite slave)](https://github.com/chipxpertofficial/btech-vlsi-projects)
- Verify one with UVM → [uvm-testbench-cookbook](https://github.com/chipxpertofficial/uvm-testbench-cookbook)
- Verified RTL building blocks → [vlsi-verilog-examples](https://github.com/chipxpertofficial/vlsi-verilog-examples)
- SoC integration training → [ChipXpert](https://chipxpert.in/)

## License
MIT — see [LICENSE](LICENSE). Maintained by the ChipXpert team.

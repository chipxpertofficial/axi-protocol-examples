# AXI4 vs AXI4-Lite vs AXI4-Stream — which, and why

The AXI family has three members. They share the VALID/READY handshake but serve very different
jobs. Pick the lightest one that does what you need.

## Quick comparison
| | **AXI4-Lite** | **AXI4 (full)** | **AXI4-Stream** |
|---|---|---|---|
| Use | control/status registers | high-throughput memory-mapped | data pipelines (no addresses) |
| Addresses | yes, single | yes, with bursts | **none** |
| Bursts | no (1 beat) | yes (up to 256 beats) | continuous |
| IDs / reordering | no | yes (`AxID`) | no |
| Channels | 5 | 5 | 1 (just T) |
| Typical | a peripheral's CSR block | a DMA ↔ DDR path | video/DSP sample streams |

## AXI4 full — what's added over Lite
- **Bursts:** one address starts a run of beats (`AWLEN` = beats−1, `AWSIZE` = bytes/beat,
  `AWBURST` = FIXED/INCR/WRAP). Amortises address overhead for memory transfers.
- **Transaction IDs:** multiple outstanding transactions, responses can come back out of order.
- **WLAST/RLAST:** mark the final beat of a burst.
Use full AXI4 when you move bulk data (DMA, memory controllers). It's overkill for a few registers.

## AXI4-Stream — the odd one out
No addresses at all — just a one-directional data flow:
```
TDATA, TVALID, TREADY, TLAST (packet boundary), TKEEP (byte qualifier), TUSER (sideband)
```
It's for **pipelines**: a filter feeding a FIFO feeding a packer. `TLAST` marks the end of a packet
(e.g. a video line/frame). Same VALID/READY handshake; no concept of read vs write.

## How to choose
- A few control registers → **AXI4-Lite**. Simplest; don't over-build.
- Moving blocks of data to/from memory → **AXI4 full** (bursts earn their keep).
- A streaming datapath (DSP, video, networking) → **AXI4-Stream**.

## Interview-worthy summary
"Lite for control, full for memory bursts, Stream for data pipelines — all three ride the same
VALID/READY handshake." (See [vlsi-interview-prep](https://github.com/chipxpertofficial/vlsi-interview-prep).)

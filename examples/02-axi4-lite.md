# AXI4-Lite

AXI4-Lite is the stripped-down AXI used for register interfaces (control/status). No bursts, no
IDs — just single 32/64-bit transfers. It's the first AXI anyone should implement.

## The five channels
| Channel | Signals (core) | Direction |
|---------|----------------|-----------|
| **AW** write address | `AWADDR`, `AWVALID`, `AWREADY` | master → slave |
| **W** write data | `WDATA`, `WSTRB`, `WVALID`, `WREADY` | master → slave |
| **B** write response | `BRESP`, `BVALID`, `BREADY` | slave → master |
| **AR** read address | `ARADDR`, `ARVALID`, `ARREADY` | master → slave |
| **R** read data | `RDATA`, `RRESP`, `RVALID`, `RREADY` | slave → master |

Each uses the VALID/READY handshake from doc 01.

## A write transaction, step by step
1. Master drives `AWADDR` + `AWVALID`; **independently** drives `WDATA`+`WSTRB`+`WVALID`.
2. Slave accepts AW (AWREADY) and W (WREADY) — **in either order**, or together.
3. Once both are accepted, slave performs the write and raises `BVALID` with `BRESP`.
4. Master accepts the response (`BREADY`). Done.

Because AW and W are independent, a correct slave must handle "W arrives before AW" — a common
bug is assuming AW always comes first.

## `WSTRB` — byte strobes
`WSTRB[n]` = "byte n of WDATA is valid." Lets a master write a single byte/halfword into a 32-bit
register without read-modify-write. A correct slave applies only the strobed bytes.

## Response codes (`BRESP`/`RRESP`)
- `2'b00 OKAY` — success.
- `2'b10 SLVERR` — slave error (e.g. address out of range, write to a read-only reg).
- `2'b11 DECERR` — no slave at that address (from the interconnect).
Return `SLVERR` for an unmapped register rather than silently ignoring it.

## A register-slave recipe
- Decode `AWADDR[hi:2]` / `ARADDR[hi:2]` → register index.
- Hold `AWREADY`/`WREADY` logic so you commit the write only when **both** AW and W have handshaked.
- For reads, register the addressed word and raise `RVALID` next cycle with `OKAY`.
- Out-of-range → `SLVERR`.

A full buildable version is [btech-vlsi-projects #05](https://github.com/chipxpertofficial/btech-vlsi-projects);
verify it with the [UVM cookbook](https://github.com/chipxpertofficial/uvm-testbench-cookbook).

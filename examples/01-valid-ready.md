# The VALID/READY handshake

Every AXI channel — all five of them — transfers data using the exact same rule. Learn it once.

## The rule
A **beat** (one transfer) happens on a rising clock edge when **both** `VALID` and `READY` are high.

```
        ___     ___     ___     ___
clk  __/   \___/   \___/   \___/   \__
VALID ________/‾‾‾‾‾‾‾‾‾‾‾\_________      (source: "I have data")
READY ____________/‾‾‾‾‾\___________      (dest: "I can take it")
                      ^
                 transfer here (both high)
```
Source drives `VALID` + payload; destination drives `READY`. Transfer occurs only where they
overlap.

## The three laws (memorise these)
1. **A source must not wait for READY to assert VALID.** Once it has data, it raises VALID. (If
   VALID waited for READY and READY waited for VALID, both sides deadlock.)
2. **A destination may assert READY before or after VALID.** It can pre-raise READY (always ready)
   or wait until it sees VALID — both legal.
3. **Payload must stay stable while VALID is high and READY is low.** You asserted VALID with some
   ADDR/DATA; you may not change it until the beat completes.

## The three legal timing cases
- **VALID before READY** — source has data, destination not yet ready; source holds everything
  stable and waits.
- **READY before VALID** — destination is pre-ready; transfer happens the instant VALID rises.
- **Same cycle** — both high together; immediate transfer.
All three are correct. A good testbench exercises all three.

## A minimal source (skeleton)
```verilog
// assert VALID when we have data; deassert after the handshake
always @(posedge clk) begin
  if (!rst_n)               valid <= 1'b0;
  else if (have_data)       valid <= 1'b1;        // raise, don't wait for ready
  else if (valid && ready)  valid <= 1'b0;        // handshake done
end
// payload held stable while valid && !ready (don't change it mid-wait)
```

## A minimal destination (skeleton)
```verilog
assign ready = can_accept;                 // may be a constant 1 (always ready)
always @(posedge clk)
  if (valid && ready) capture <= data;     // take the beat
```

## The #1 bug
Making `VALID` depend on `READY` (law 1). It passes a simple directed test where the other side is
always ready, then deadlocks the moment a real slave applies back-pressure. Always test with a
destination that **delays** READY randomly.

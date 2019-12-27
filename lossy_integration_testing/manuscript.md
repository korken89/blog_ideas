# Integration testing lossy connections / Testing framework for RF devices

## Issue

Integration testing multiple state machines that transmit events over a lossy medium over each other.


## Test framework design

Each connected unit communicates over a serial.

```rust
let serial1 = "/dev/ttyACM0";
let serial2 = "/dev/ttyACM1";

test.exptect(serial1, "Send abc").expect(serial2, "Receive abc").pass()
```



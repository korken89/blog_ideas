# Integration testing lossy connections / Testing framework for RF devices

## Issue

Integration testing multiple state machines that transmit events over a lossy medium over each other.


## Test framework design

Each connected unit communicates over a serial.

```rust
let firmware1 = Firmware::new("tests/test_1_send");
let firmware2 = Firmware::new("tests/test_1_rec");

let serial1 = Serial::new("/dev/ttyACM0");
let serial2 = Serial::new("/dev/ttyACM1");

firmware1.flash();
firmware2.flash();

let test = exptect(&serial1, "Send abc").expect(&serial2, "Received abc").pass();

firmware1.reset();
firmware2.reset();

test.run()
```



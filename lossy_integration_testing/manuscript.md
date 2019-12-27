# Integration testing lossy connections / Testing framework for RF devices

## Issue

Integration testing multiple state machines that transmit events over a lossy medium over each other.


## Test framework design

Each connected unit communicates over a serial.

```rust
// Firmware folders
let firmware1 = Firmware::new("tests/test_1_send");
let firmware2 = Firmware::new("tests/test_1_rec");

// Flashing firmware
firmware1.flash();
firmware2.flash();



// Serials
let serial1 = Serial::new("/dev/ttyACM0");
let serial2 = Serial::new("/dev/ttyACM1");

//
// Somehow match serial to firmware...
//
let firmware1_link = ...;
let firmware2_link = ...;


//
// Somehow define tests...
//
let test = exptect(&firmware1_link, "Send abc %i").expect(&firmware2_link, "Received abc %i").pass();


// Reset and run
firmware1.reset();
firmware2.reset();

// Link firmware and serial
link_firmware(...);

// Run the test
test.run()
```



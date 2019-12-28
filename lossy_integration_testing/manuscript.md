# Integration testing lossy connections / Testing framework for RF devices

## Issue

When testing RF communication there is a lot that needs to be tested, as sending and receiving RF
packets at different times, and sometimes with collisions, can cause drivers to get into strange
states and hang. Also we want to do regression testing of the functionality when adding new
features as it is easy to break the timing requirements of the network.

### TLDR

We want to integration test the communication of an RF device together with timing requirements of
common communication patterns, i.e. integration testing multiple state machines that transmit
events over a lossy medium to each other.

## How can a rudimentary test protocol look like?

Bellow follows a simple example of the features that could be tested:

### Basic tests

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Enter/exit sleep mode     | Enter/exit sleep mode, settings should be persistent over the changes.        | **FAIL** |

### Receive tests

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Receive package                       | Receive packages with some time between them (so the chip has time to idle).                                  | **FAIL** |
| Receive package                       | Receive packages as fast as possible.                                                                         | **FAIL** |
| Receive package multicast             | Receive multicast packages (with matching and multicast PAN ID).                                              | **FAIL** |
| Receive package with delay            | Receive packages using the delay feature. Delay min to max time and receive a package.                        | **FAIL** |
| Receive package with timeout          | Receive packages using the timeout feature. Should both be done with and without failures.                    | **FAIL** |
| Receive package with double-buffer    | Receive packages using the double-buffer feature.                                                             | **FAIL** |
| Receive package with frame-filtering  | Receive packages using the frame-filtering feature, both rejection and accepting (remember PAN ID rejection). | **FAIL** |


### Transmit tests

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Send package                          | Send packages with some time between them (so the chip has time to idle).                     | **FAIL** |
| Send package                          | Send packages as fast as possible.                                                            | **FAIL** |
| Send package multicast                | Send multicast packages (with normal and multicast PAN ID).                                   | **FAIL** |
| Send package with delay               | Send packages with a delay of 1-8500 ms.                                                      | **FAIL** |
| Send package with delay error         | Send packages with a delay that causes Half Period Warning.                                   | **FAIL** |
| Send package with TPU error           | Send packages with a delay that causes Transmit power up time error.                          | **FAIL** |
| Send package with different MAC types | Send packages with different MAC types (in conjunction with Receive using frame-filtering).   | **FAIL** |


### Receive / Transmit tests

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Send / receive                    | Send and receive packages with some time between them (so the chip has time to idle).     | **FAIL** |
| Send / receive (with auto RX)     | Send and receive packages using auto RX, else same as last.                               | **FAIL** |


### Clock stability tests

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Synchronize to another clock                      | Receive packages with that include the current clock value and synchronize the clocks.                                                                | **FAIL** |
| Synchronize to another clock (with clock tune)    | Receive packages with that include the current clock value and synchronize the clocks. Use the clock tune register to get rid of most of the error.     | **FAIL** |

### Frame synchronization

| **Test** | **Description** | **Pass / Fail** |
| -------- | --------------- | :-------------: |
| Synchronize to super frames and send/receive      | Receive packages with that include the current frame time synchronize the sending to a free slot, while receiving in the other slot.    | **FAIL** |


## Test framework design

Each connected unit communicates over a serial.

```rust
// Firmware folders
let firmware1 = Firmware::new("tests/test_1_send");
let firmware2 = Firmware::new("tests/test_1_rec");

// Serials
let serial1 = Serial::new("/dev/ttyACM0");
let serial2 = Serial::new("/dev/ttyACM1");

// Somehow define devices...
let devices = Devices::new().with_device(firmware1, serial1).with_device(firmware2, serial2).done();

// Flash and run
devices.flash();

// Somehow define tests...
// I think that %s would be some string that must match on both sender and receiver. Or something...
let test = Test::new().exptect(&devices[0], "Send abc %s").expect(&devices[1], "Received abc %s").done();

// Run the test
test.run()
```



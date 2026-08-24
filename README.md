# openlogi-async-hid

The OpenLogi-maintained fork of [async-hid](https://github.com/sidit77/async-hid), a Rust library for asynchronously interacting with HID devices.

Version `0.5.3-openlogi.1` is based on upstream `0.5.3` and includes the macOS report-write fix from [upstream PR #46](https://github.com/sidit77/async-hid/pull/46). The package keeps the `async_hid` Rust library name, so downstream users can adopt it without source changes:

```toml
async-hid = { package = "openlogi-async-hid", version = "=0.5.3-openlogi.1" }
```

This crate aims to be a replacement for [hidapi-rs](https://github.com/ruabmbua/hidapi-rs) without the baggage that comes from being a wrapper around a C library.

This crate generally offers a simpler and more streamlined api while also supporting async as best as possible. 

## Example

```rust
use async_hid::{AccessMode, DeviceInfo, HidResult};
use simple_logger::SimpleLogger;
use futures_lite::StreamExt;

#[pollster::main]
async fn main() -> HidResult<()> {
    SimpleLogger::new().init().unwrap();

    let device = DeviceInfo::enumerate()
        .await?
        //Steelseries Arctis Nova 7X headset
        .find(|info: &DeviceInfo | info.matches(0xFFC0, 0x1, 0x1038, 0x2206))
        .await
        .expect("Could not find device")
        .open(AccessMode::ReadWrite)
        .await?;

    device.write_output_report(&[0x0, 0xb0]).await?;
    let mut buffer = [0u8; 8];
    let size = device.read_input_report(&mut buffer).await?;
    println!("{:?}", &buffer[..size]);
    Ok(())
}
```


## Platform Support

| Operating System | Underlying API                                 |
|------------------|------------------------------------------------|
| Windows          | Win32 (`Windows.Win32.Devices`)                |
| Windows          | WinRT (`Windows.Devices.HumanInterfaceDevice`) |
| Linux            | hidraw                                         |
| MacOs            | IOHIDManager                                   |

Under Windows this crate uses either `win32` (default) or `winrt` feature for backend.


## Async
The amount of asynchronicity that each OS provides varies. The following table gives a rough overview which calls utilize async under the hood.

|                 | `enumerate`  | `open` | `read_input_report` | `write_output_report` |
|-----------------|--------------|--------|---------------------|-----------------------|
| Windows (Win32) | ❌️           | ️️ ❌️    | ✔️                  | ✔️                    |
| Windows (WinRT) | ✔️           | ✔️     | ✔️                  | ✔️                    |
| Linux           | ❌            | ❌      | ✔️                  | ✔️                    |
| MacOS           | ❌            | ✔️     | ✔️                  | ✔️                     |

Under Linux this crate uses either `async-io` (default) or `tokio` feature for the async functionality. 

## Planned Features
- Redesign how feature reads and write are integrated exposed.
  - Allow feature reads for read-only devices?
  - Allow opening read/write/feature handles all at once?
- Add support for WASM via WebHID


## License
MIT License

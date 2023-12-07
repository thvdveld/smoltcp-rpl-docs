# Using smoltcp

## 1. Adding `smoltcp` to your `Cargo.toml`

To use `smoltcp` in your project, you must add it as a dependency in the `Cargo.toml` file.
`smoltcp` uses a lot of feature flags for configuration and therefore the correct ones need to be added.

Depending on the medium that is used, at least one of the following features must be enabled:
- `medium-ethernet` for Ethernet devices;
- `medium-ieee802154` for IEEE802.15.4 devices. Enabling this feature, also enables `proto-sixlowpan`;
- `medium-ip` for devices without medium.
 

For the network layer, at least one of the following features must be enabled:
- `proto-ipv4` for IPv4;
- `proto-ipv6` for IPv6;
- `proto-sixlowpan` for 6LoWPAN.

There are many more feature flags that can be enabled.
For more information, see the [documentation](https://docs.rs/smoltcp/latest/smoltcp/#feature-flags).

~~~admonish example title="Example: IEEE802.15.4 device using 6LoWPAN and RPL"
For a IEEE802.15.4 device using 6LoWPAN and RPL, the following is added in the `Cargo.toml` file:

```toml
[dependencies.smoltcp]
version = "0.10"
default-features = false
features = [
	"medium-ieee802154",
	"proto-sixlowpan",
	"rpl-mop-2"
]
```

This will use RPL with MOP 2 (Storing Mode of Operation) and 6LoWPAN with the IEEE802.15.4 medium.
These are the available modes of operation: `rpl-mop-0`, `rpl-mop-1`, `rpl-mop-2` and `rpl-mop-3`.
~~~


## 2. Implementing `smoltcp::phy::Device` for your platform

In order for `smoltcp` to effectively handle incoming and outgoing packets, 
a connection must be established between the TCP/IP stack and the underlying hardware.
This essential linkage is achieved by implementing the [`smoltcp::phy::Device`](https://docs.rs/smoltcp/latest/smoltcp/phy/trait.Device.html)
trait for the hardware:

```rust
# extern crate smoltcp;
# use smoltcp::phy::DeviceCapabilities;
pub trait RxToken {
    fn consume<R, F>(self, f: F) -> R
    where
        F: FnOnce(&mut [u8]) -> R;
}

pub trait TxToken {
    fn consume<R, F>(self, len: usize, f: F) -> R
    where
        F: FnOnce(&mut [u8]) -> R;
}

pub trait Device {
    type RxToken<'a>: RxToken
    where
        Self: 'a;
    type TxToken<'a>: TxToken
    where
        Self: 'a;

    fn receive(&mut self, timestamp: Instant) -> Option<(Self::RxToken<'_>, Self::TxToken<'_>)>;

    fn transmit(&mut self, timestamp: Instant) -> Option<Self::TxToken<'_>>;

    fn capabilities(&self) -> DeviceCapabilities;
}
```

## 3. Setting up the `smoltcp` stack

Once you have the `Device` trait implemented, an [`Interface`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html) can be created.
The `Interface` is the main entry point for the `smoltcp` stack.
It is responsible for handling incoming and outgoing packets, as well as managing the sockets.
The `Interface` is created by passing a [`Config`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Config.html)
and a `Device` to the [`new`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html#method.new) function.
The `Config` struct contains all the configuration options for the stack.
The `MyDevice` is the hardware abstraction layer that was implemented in the previous step.


```rust
# extern crate smoltcp;
use smoltcp::wire::*;
use smoltcp::iface::{Interface, Config};
use smoltcp::socket::{SocketSet, SocketStorage};

fn main() {
    // Get the MAC address from the device:
    let mac_adddr = 
        Ieee802154Address::Extended([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

    // Create the device:
    let mut device = MyDevice::new(mac_adddr);

    // Create the configuration for the stack:
    let mut config = Config::new(mac_addr.into());
    config.pan_id = Some(Ieee802154Pan(0xbeef));

    // Create the sockets:
    let mut sockets_buffer = [SocketStorage::EMPTY; 1];
    let mut sockets = SocketSet::new(&mut sockets_buffer[..]);

    // Create the interface:
    let mut iface = Interface::new(config, device)?;
}
```
## 4. Polling the `smoltcp` stack

The interface is now ready to be used.
Polling the interface will handle incoming and outgoing packets.
For the interface to continue working, it must be polled regularly.
This is done by calling the [`poll`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html#method.poll)
function on the interface.
If a packet is received, it is queued by the device and can be handled by the stack.
After processing the packet, the stack might have to send a packet.

Calling `poll_at` on the interface returns the time when the next `poll` should be called.
Calling `poll` before that time is only wasting energy, but is not harmful for the stack.
Calling `poll` after that duration might be harmful for the stack.

```rust
loop {
	iface.poll(now, device, sockets);

	match iface.poll_at(Instant::now()) {
		Some(Instant::ZERO) => continue,
		Some(d) => sleep(d),
		None => sleep_until_new_packet(),
	}
}

```

## 5. You're all set 🎉 (for now)

That's it! There is nothing more that needs to be done!
The `smoltcp` stack will now handle all incoming and outgoing packets.
The next step is to create sockets and use them to send and receive data.

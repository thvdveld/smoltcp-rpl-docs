# Using smoltcp

> For now, we assume that the final setup is going to be a stack using the RPL routing protocol, using 6LoWPAN over an IEEE802.15.4 network.

## 1. Adding `smoltcp` to your `Cargo.toml`
To be able to use `smoltcp` in your project, you must add it as a dependency in your `Cargo.toml` file.
However, `smoltcp` uses a lot of feature flags for configuration and therefore the correct ones need to be added.
For a IEEE802.15.4 device using 6LoWPAN and RPL, the following is added in the `Cargo.toml` file:

```toml
[dependencies.smoltcp]
version = "0.9"
default-features = false
features = [
	"medium-ieee802154",
	"proto-sixlowpan",
	"proto-rpl"
]
```

<!--More information about the feature flags can be found [here](./feature_flags.md).-->

## 2. Implementing `smoltcp::phy::Device` for your platform

`smoltcp` needs to be able to accept incoming packets and transmit packets.
To be able for `smoltcp` to do this, a connection needs to be made between the TCP/IP stack and the hardware.
This is done by implementing the [`smoltcp::phy::Device`](https://docs.rs/smoltcp/latest/smoltcp/phy/trait.Device.html) trait for the hardware:

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

Once you have the `Device` trait implemented, an [`Interface`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html) can be created:

```rust
#extern crate smoltcp;
#use smoltcp::wire::Ieee802154Address;
#fn main() {
let ll_addr = Ieee802154Address::Extended([0x1, 0x2, 0x3, 0x4, 0x5, 0x6 0x7, 0x8]);

// Create the device:
let mut device = MyDevice::new(ll_addr);

// Create the config for RPL:
let mut rpl_config = smoltcp::iface::RplConfig::new();

// Create the configuration for the stack:
let mut config = smoltcp::iface::Config::new();
config.hardware_addr =	Some(ll_addr.into());
config.pan_id = Some(Ieee802154Pan(0xbeef));

// Add the RPL config:
config.rpl = rpl_config;

// Create the sockets:
let mut sockets_buffer = [SocketStorage::EMPTY; 1];
let mut sockets = SocketSet::new(&mut sockets_buffer[..]);

// Create the interface:
let mut iface = smoltcp::iface::Interface::new(config, device)?;

#}
```

## 4. Polling the `smoltcp` stack

After the interface is created, the stack is ready to be polled (by calling the [`poll`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html#method.poll) function on the interface).
Polling the stack transmits packets that were queued or handles received packets queued by the device.
If a packet is processed by the stack, the readiness of sockets might have changed.
Therefore, it is possible that the stack needs to be polled multiple times.
The [`poll_delay`](https://docs.rs/smoltcp/latest/smoltcp/iface/struct.Interface.html#method.poll_delay) function returns an _advisory wait_ time for calling `poll` the next time. 
Calling `poll` before that time is only wasting energy, but is not harmful for the stack,
but might be when `poll` is called after that duration.

```rust
loop {
	iface.poll(Instant::now(), device, sockets);

	match iface.poll_at(Instant::now()) {
		Some(Instant::ZERO) => continue,
		Some(d) => sleep(d),
		None => sleep_until_new_packet(),
	}
}
```

## 5. You're all set 🎉 (for now)

That's it! There is nothing more that needs to be done, if you want a TCP/IP stack without sockets.
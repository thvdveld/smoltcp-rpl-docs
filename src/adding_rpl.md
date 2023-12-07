# Adding RPL

To enable RPL, one of the following feature flags must be enabled:
- `rpl-mop-0`: Enables RPL in only upward messages (MOP0);
- `rpl-mop-1`: Enables RPL in non-storing mode (MOP1);
- `rpl-mop-2`: Enables RPL in storing mode (MOP2);
- `rpl-mop-3`: Enables RPL in storing mode with multicast (MOP3).

When enabling one of these feature flags, it is required to add a RPL configuration to the the interface.

```rust
use smoltcp::iface::{Interface, Config, RplConfig, RplModeOfOperation};

let mut config = Config::new(mac_addr.into());
config.rpl = RplConfig::new(RplModeOfOperation::StoringMode);
```

For a RPL root node, the root configuration must be added:

```rust
let mut config = Config::new(mac_addr.into());
config.rpl = RplConfig::new(RplModeOfOperation::StoringMode).add_root_config(
    RplRootConfig::new(
        RplInstanceId::new(30),                         // Instance ID
        Ipv6Address::new(0xfd00, 0, 0, 0, 0, 0, 0, 1),  // DODAG ID
    )
);
```

## That's it! The interface is now using RPL 🎉

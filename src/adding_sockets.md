# Adding sockets

In the previous section, we created device with an interface.
We already used a `SocketSet`, however, this one was empty.
Let's create a `TcpSocket` and add it to the `SocketSet`:
```rust
let mut sockets = SocketSet::new(vec![]);

let tcp_socket = TcpSocket::new(
    TcpSocketBuffer::new(vec![0;1500]),
    TcpSocketBuffer::new(vec![0;1500]),
);

let tcp_handle = sockets.add(tcp_socket);
```

Now, we can use the `TcpSocket` to connect to a remote host:
```rust
let remote_addr = Ipv6Address::new(0xfe80, 0, 0, 0, 0, 0, 0, 2);
let remote_port = 4242;
let host_port = 4343;

let socket = socket.get_mut::<tcp::Socket>(tcp_handle);
socket.connect(iface.context(), (remote_addr, remote_port), host_port).unwrap();
```

The `TcpSocket` will now try to connect to the remote host.
However, we still need to poll the interface to make sure that the `TcpSocket` can send and receive packets.

```rust
loop {
    if let Err(e) = iface.poll(&mut sockets, Instant::now()) {
        println!("Network error: {:?}", e);
        continue;
    }
}
```

The `TcpSocket` is now ready to send and receive data.

```rust
loop {
    if let Err(e) = iface.poll(&mut sockets, Instant::now()) {
        println!("Network error: {:?}", e);
        continue;
    }

    let mut socket = sockets.get::<TcpSocket>(tcp_handle);
    
    if !socket.is_open() {
        println!("Connection closed");
        break;
    }
    
    if socket.can_send() {
        socket.send_slice(b"Hello world!").unwrap();
    }
    
    if socket.can_recv() {
        let data = socket.recv(|data| {
            println!("Received data: {:?}", data);
            (data.len(), data)
        }).unwrap();
    }
}
```

## Other socket types

The [`TcpSocket`](https://docs.rs/smoltcp/latest/smoltcp/socket/tcp/struct.Socket.html) is not the only socket type available.
There are also the following types:
- [`Dhcpv4Socket`](https://docs.rs/smoltcp/latest/smoltcp/socket/dhcpv4/struct.Socket.html): a DHCPv4 client socket;
- [`UdpSocket`](https://docs.rs/smoltcp/latest/smoltcp/socket/udp/struct.Socket.html): a UDP socket;
- [`DnsSocket`](https://docs.rs/smoltcp/latest/smoltcp/socket/dns/struct.Socket.html): a DNS client socket;
- [`IcmpSocket`](https://docs.rs/smoltcp/latest/smoltcp/socket/icmp/struct.Socket.html): an ICMP socket.
  This socket type is used to send and receive ICMP packets and might listen for specific ICMP error messages related to a UDP/TCP port;
- [`RawSocket`](https://docs.rs/smoltcp/latest/smoltcp/socket/raw/struct.Socket.html): a raw socket.
  This socket type is used to send and receive raw packets.

```admonish note title="Feature flags"
Every socket type requires to enable the corresponding feature.
```

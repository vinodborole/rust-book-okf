---
type: Web Page
title: IpAddr in std::net - Rust
description: An IP address, either IPv4 or IPv6.
resource: https://doc.rust-lang.org/stable/std/net/enum.IpAddr.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

```
pub enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```
## Expand description

An IP address, either IPv4 or IPv6.

This enum can contain either an `Ipv4Addr` or an `Ipv6Addr`, see their
respective documentation for more details.

## §Examples

```
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr};
let localhost_v4 = IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1));
let localhost_v6 = IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1));
assert_eq!("127.0.0.1".parse(), Ok(localhost_v4));
assert_eq!("::1".parse(), Ok(localhost_v6));
assert_eq!(localhost_v4.is_ipv6(), false);
assert_eq!(localhost_v4.is_ipv4(), true);
```
## Variants§

## Implementations§

Source§### impl IpAddr

 

### impl IpAddr

1.12.0 (const: 1.50.0) · Source#### pub const fn is_unspecified(&self) -> bool

 

#### pub const fn is_unspecified(&self) -> bool

Returns `true` for the special ‘unspecified’ address.

See the documentation for `Ipv4Addr::is_unspecified()` and
`Ipv6Addr::is_unspecified()` for more details.

##### §Examples

1.12.0 (const: 1.50.0) · Source#### pub const fn is_loopback(&self) -> bool

 

#### pub const fn is_loopback(&self) -> bool

Returns `true` if this is a loopback address.

See the documentation for `Ipv4Addr::is_loopback()` and
`Ipv6Addr::is_loopback()` for more details.

##### §Examples

Source#### pub const fn is_global(&self) -> bool

 🔬This is a nightly-only experimental API. (`ip` #27709)

#### pub const fn is_global(&self) -> bool

`ip` #27709)Returns `true` if the address appears to be globally routable.

See the documentation for `Ipv4Addr::is_global()` and
`Ipv6Addr::is_global()` for more details.

##### §Examples

1.12.0 (const: 1.50.0) · Source#### pub const fn is_multicast(&self) -> bool

 

#### pub const fn is_multicast(&self) -> bool

Returns `true` if this is a multicast address.

See the documentation for `Ipv4Addr::is_multicast()` and
`Ipv6Addr::is_multicast()` for more details.

##### §Examples

Source#### pub const fn is_documentation(&self) -> bool

 🔬This is a nightly-only experimental API. (`ip` #27709)

#### pub const fn is_documentation(&self) -> bool

`ip` #27709)Returns `true` if this address is in a range designated for documentation.

See the documentation for `Ipv4Addr::is_documentation()` and
`Ipv6Addr::is_documentation()` for more details.

##### §Examples

Source#### pub const fn is_benchmarking(&self) -> bool

 🔬This is a nightly-only experimental API. (`ip` #27709)

#### pub const fn is_benchmarking(&self) -> bool

`ip` #27709)Returns `true` if this address is in a range designated for benchmarking.

See the documentation for `Ipv4Addr::is_benchmarking()` and
`Ipv6Addr::is_benchmarking()` for more details.

##### §Examples

1.16.0 (const: 1.50.0) · Source#### pub const fn is_ipv4(&self) -> bool

 

#### pub const fn is_ipv4(&self) -> bool

Returns `true` if this address is an `IPv4` address, and `false`
otherwise.

##### §Examples

1.16.0 (const: 1.50.0) · Source#### pub const fn is_ipv6(&self) -> bool

 

#### pub const fn is_ipv6(&self) -> bool

Returns `true` if this address is an `IPv6` address, and `false`
otherwise.

##### §Examples

1.75.0 (const: 1.75.0) · Source#### pub const fn to_canonical(&self) -> IpAddr

 

#### pub const fn to_canonical(&self) -> IpAddr

Converts this address to an `IpAddr::V4` if it is an IPv4-mapped IPv6
address, otherwise returns `self` as-is.

##### §Examples

```
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr};
let localhost_v4 = Ipv4Addr::new(127, 0, 0, 1);
assert_eq!(IpAddr::V4(localhost_v4).to_canonical(), localhost_v4);
assert_eq!(IpAddr::V6(localhost_v4.to_ipv6_mapped()).to_canonical(), localhost_v4);
assert_eq!(IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1)).to_canonical().is_loopback(), true);
assert_eq!(IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0xffff, 0x7f00, 0x1)).is_loopback(), false);
assert_eq!(IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0xffff, 0x7f00, 0x1)).to_canonical().is_loopback(), true);
```
Source§### impl IpAddr

 

### impl IpAddr

Source#### pub fn parse_ascii(b: &[u8]) -> Result<IpAddr, AddrParseError>

 🔬This is a nightly-only experimental API. (`addr_parse_ascii` #101035)

#### pub fn parse_ascii(b: &[u8]) -> Result<IpAddr, AddrParseError>

`addr_parse_ascii` #101035)Parse an IP address from a slice of bytes.

```
#![feature(addr_parse_ascii)]
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr};
let localhost_v4 = IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1));
let localhost_v6 = IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1));
assert_eq!(IpAddr::parse_ascii(b"127.0.0.1"), Ok(localhost_v4));
assert_eq!(IpAddr::parse_ascii(b"::1"), Ok(localhost_v6));
```
## Trait Implementations§

1.17.0 (const: unstable) · Source§### impl From<[u16; 8]> for IpAddr

 

### impl From<[u16; 8]> for IpAddr

1.17.0 (const: unstable) · Source§### impl From<[u8; 16]> for IpAddr

 

### impl From<[u8; 16]> for IpAddr

Source§#### fn from(octets: [u8; 16]) -> IpAddr

 

#### fn from(octets: [u8; 16]) -> IpAddr

Creates an `IpAddr::V6` from a sixteen element byte array.

##### §Examples

```
use std::net::{IpAddr, Ipv6Addr};
let addr = IpAddr::from([
    0x19u8, 0x18u8, 0x17u8, 0x16u8, 0x15u8, 0x14u8, 0x13u8, 0x12u8,
    0x11u8, 0x10u8, 0x0fu8, 0x0eu8, 0x0du8, 0x0cu8, 0x0bu8, 0x0au8,
]);
assert_eq!(
    IpAddr::V6(Ipv6Addr::new(
        0x1918, 0x1716, 0x1514, 0x1312,
        0x1110, 0x0f0e, 0x0d0c, 0x0b0a,
    )),
    addr
);
```

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/net/enum.IpAddr.html

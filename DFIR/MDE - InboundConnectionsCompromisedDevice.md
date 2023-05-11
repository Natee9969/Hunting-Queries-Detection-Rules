# (Public) Inbound connections to a compromised device

## Query Information

#### Description
This query can be used to get a quick overview of all the inbound connections that have been accepted to a compromised device. This could indicate that the RemoteIP is used to gain access to the device itself. The query contains a filter to filter private IP addresses and to only search for public (RFC1918) addresses. 

#### References
- https://www.microsoft.com/en-us/security/blog/2020/12/28/using-microsoft-365-defender-to-coordinate-protection-against-solorigate/

## Defender For Endpoint
```
// Add the device you are investigating in the CompromisedDevice variable
let CompromisedDevice = "test.domain.tld";
let SearchWindow = 10d; //Customizable h = hours, d = days
let IPRegex = '[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}';
DeviceNetworkEvents
| where Timestamp > ago(SearchWindow)
| where DeviceName == CompromisedDevice
// Only list accepted inbound connections
| where ActionType == "InboundConnectionAccepted"
// Add column wich displays if the remote IP is private
| extend FourToSixPrivate = extract(IPRegex, 0, RemoteIP)
| extend RemoteIPIsPrivate = iff(ipv4_is_private(RemoteIP), 1, iff(ipv4_is_private(FourToSixPrivate), 1, 0))
// Remove comment below if you only want to see inbound connections from public IP addresses.
//| where RemoteIPIsPrivate == 0
| project Timestamp, DeviceName, RemoteIP, RemotePort, RemoteIPIsPrivate, FourToSixPrivate, LocalIP, LocalPort
```
## Sentinel
```
// Add the device you are investigating in the CompromisedDevice variable
let CompromisedDevice = "test.domain.tld";
let SearchWindow = 10d; //Customizable h = hours, d = days
let IPRegex = '[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}';
DeviceNetworkEvents
| where TimeGenerated > ago(SearchWindow)
| where DeviceName == CompromisedDevice
// Only list accepted inbound connections
| where ActionType == "InboundConnectionAccepted"
// Add column wich displays if the remote IP is private
| extend FourToSixPrivate = extract(IPRegex, 0, RemoteIP)
| extend RemoteIPIsPrivate = iff(ipv4_is_private(RemoteIP), 1, iff(ipv4_is_private(FourToSixPrivate), 1, 0))
// Remove comment below if you only want to see inbound connections from public IP addresses.
//| where RemoteIPIsPrivate == 0
| project TimeGenerated, DeviceName, RemoteIP, RemotePort, RemoteIPIsPrivate, FourToSixPrivate, LocalIP, LocalPort
```

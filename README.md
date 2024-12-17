# Ansible wireguard
Ansible role for wireguard.
This ansible role utilizes Netplan to generate the configuration for wireguard.
You can use it to configure multiple wireguard interfaces.

## Setup
use wireguard to generate a public and private key. You can do this graphically in windows/mac
or in linux with cli:
```
wg genkey | tee privkey | wg pubkey > pubkey
```
This will create privkey and pubkey files containing the private and the public key respectively.

> Note: The wg genkey command just outputs the private key. The wg pubkey similary just outputs the public key for the private key it's being fed (you must pipe the private key to the wg pubkey command)

Once you have your keys ready, you can supply the public key to anyone who will be connecting.
Clients will need to generate their own keys, and send you the public key.

> Note on security: Sharing public key does not present a security risk. Private keys however should be kept safe. This is why clients generating their own keys is safer than having them generated centrally and having them distributed via email - for example.

## Configuration values
The whole configuration is kept in one single structure:
- wireguard_tunnels: Contains a list of wireguard tunnels
  - name: Single string without spaces or special characters representing the interface to be created e.g. "wg1"
    private_key: Private key to be used. Recommended to use ansible vault and call the value here
    public_key: Public key of the interface
    addresses: List of addresses for the interface.
    port: Port for wireguard to listen on.
    peers: List of objects - peer
      - name: String - user friendly name for the peer
        public_key: Public key of the peer
        allowed_ips: List of IPs to be allowed from peer - which in most cases means: "specify peer IP"

### Example
In the following example a single wireguard tunnel is specified (wg1).
it uses the internal IP of 192.168.100.1 - peers use the same subnet.
It listens on port 12345 - peers will use this and the host public IP to connect.
The "wireguard_private_key_wg1_vault" variable is stored in ansible vault and referenced here.
```
wireguard_tunnels:
  - name: wg1
    private_key: "{{ wireguard_private_key_wg1_vault }}"
    public_key: "9m5/Iol92aZLj39IRtj86/Jwa/93J4l796tdAaeUZ38="
    addresses:
      - 192.168.100.1/24
    port: 12345
    peers:
      - name: user1
        public_key: oFA+lBJLgPD253XfQLOABIG26M1Fdp+3/ZAnxVXYu04=
        allowed_ips:
          - 192.168.100.11/32
      - name: user2
        public_key: IUbmqgHzOFhW1R5vdwtFJGujUY7kRE2COvC+k1i2wFI=
        allowed_ips:
          - 192.168.100.12/32
```

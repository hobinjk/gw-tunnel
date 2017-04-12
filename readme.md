# Gateway remote access 

## Actors

The system relies on 5 entities:
* A DNS server which is authoritative and only resolves $name.box.$domain names.
* A registration server, keeping track of the known boxes and their local IPs.
* A tunnel server ('frontend' in pagekite terms)
* A registration client, to orchestrate the name registration, get the https
  certificates from Let's Encrypt and periodically transmit the local IP to
  the registration server.
* A tunnel client ('kite' in pagekite terms) only relaying port 443.

## Initial registration

1. The registration client notifies the registration server that it will own a subdomain name
   which matches the MAC address of the gateway.
2. The registration client starts the LE DNS challenge for the MAC endpoint, cooperating with
   the registration server to configure the DNS server TXT record appropriately.
3. Once the LE challenge succeeds, the registration client and server set up the MAC tunnel.
4. The user navigates to http://mozbox.local and is redirected to https://$MAC.box.knilxof.org to
   complete the next steps.
5. The registration client chooses a friendly name, used to build the public gateway URL
   (eg. fabrice.box.knilxof.org). It tries names until the registration server
   returns success and a unique token associated to this name.
6. The registration client initiates the LE DNS challenge, cooperating with the registration
   server to configure the DNS server TXT record appropriately.
7. Once the LE challenge succeeds, the registration client and server set up the tunnel and the
   browser is redirected to https://$FRIENDLY_NAME.knilxof.org/.

## Normal operation mode

After a restart of the gateway, the registration client re-establishes the tunnel and
sends periodic updates of its local IP to the registration server, using the token
provided during the initial registration.

On the server side, if the client hasn't registered for a while, the tunnel is shut down
and the local IP for the token is null-ified. All DNS request for the name associated
to this token will fail.

# Implementation

1. The DNS server will be PowerDNS used with a Remote Backend (https://doc.powerdns.com/md/authoritative/backend-remote/).
The Remote Backend will either be the same http server as the registration server, or use zeroMQ to not expose the PowerDNS
protocol publicly.

2. The http registration server is a standalone https server, implemented either in Node.js or in Rust. The database can be
initially SQLite and then move to eg. PostgreSQL.

3. The registration client will be a node module, reusable by the other pieces of the gateway.

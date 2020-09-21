# Decentralized DNS (dDNS) Whitepaper
#### By: Jared Rice Sr.
#### Authored On: September 19th, 2020
#### Last Updated On: September 20th, 2020

## Abstract
Since the dWeb is not dependant on centralized domain names, compatible with the Internet's tree-structured name space or with the Internet's "Domain Name System" (as defined in RFC 1034 and 1035) for that matter - the dWeb needed its own domain name system that was capable of functioning alongside the dWeb's family of protocols and frameworks, as well as dWeb's tree-structured name space. The dWeb could never be used by everyday people without decentralized domain names or a directory-like service that's compatible with dWeb-based discovery keys. This paper breaks down "The Decentralized Domain Name System" and how it allows a decentralized web browser to resolve the dWeb key for a particular "Resource Record" (RR), related to any decentralized top-level domain (dTLD).

## Background
The Domain Name System (DNS) (RFC 1034 and RFC 1035) is a directory lookup service that provides a mapping between the name of a host on the Internet and its numerical address. DNS is essential to the functioning of the Internet. The Decentralized Domain Name System (dDNS), like DNS, is a directory lookup service but it provides a mapping between a name of a dDrive, peer or device on the dWeb and a dWeb's "discovery keys". dDNS is essential to the functioning of the Decentralized Web.

## dDNS Operation Overview
1. A user requests a dWeb key for a dTLD. 
2. A resolver module queries the local "NameDrive" in the same domain as the resolver (typically on the user's computer) to see if a record is being kept in a local database or cache, and if so, returns the dWeb key to the requestor.
3. If the record cannot be found locally, the dWeb's root system is queried via the ARISEN blockchain for the "authoritative NameDrive(s)" (ND records) for the domain name. An ND record like other record types, points to a dWeb key, that usually points to a NameDrive (a dDrive containing distributed databases of resource records) where the domain's records are stored in a distributed database like a [dAppDB](https://github.com/distributedweb/dappdb) or a [dWebTrie](https://github.com/distributedweb/dwebtrie) (a decentralized and distributed key-value store).
4. The database within the NameDrive(s) is queried for the requested record. If found, the record is returned to the requestor and cached by the requestor in their local NameDrive for the time specified in the time-to-live (TTL) field of the retrieved Resource Record (RR).
5. The user's program or web browser is given the dWeb key or an error message. (See [Error Messages](#error-messages)).

### Pseudo Representation Of The dDNS Operation Overview

```
[User System]
        ^
        |
        |
 User Program
        |
   Local ND  => cache
        |
     dWeb <=> [Root System] <=> [ND Record For Domain] <=> [Domain's NameDrive] <=> [Resource Record]
```

## The Root Network
The dWeb utilizes the [ARISEN Blockchain](https://arisen.network) and the [dNames Smart Contract](https://github.com/peepsx/dnames-contract) as its root dDNS network. This means ARISEN is responsible for the management of the dWeb's domain name space and each domain's authoritative ND records. New dTLDs can be added to the root of the domain tree through dTLD auctions. As an example, the dTLD ".dnet" was won by the @peeps user on the ARISEN network. @peeps can now sell .dnet domains and at the very same time, increased the size of the dWeb's domain name space at the very same time. There is no limit to the amount of dTLDs that can be won at auction.

### Root Database Actions
The root system's database is managed by the dNames contract, which brings to life the following actions:

#### _ADD_:
The `add` action allows a user to add a ND record for a domain. The user must provide the following parameters:
- `domain` - This is a form of authentication, since domains possess their own keypairs. The domain's keys are used to sign the transaction related to running the `add` action, proving that the user can modify ND records for the domain.
- `record_name` - The record name for the ND record (ex. nd1.domain.dcom)
- `type` - This is autofilled as `ND`. See other [Resource Record Types](#resource-record-types).
- `class` - This autofills as `DW` (dWeb) - similar to how regular DNS uses `IN` for Internet. See other [Record Record Classes](#resource-record-classes).
- `ttl` - Typically when an RR is retrieved from a name server or a NameDrive in this case, the retriever will cache the RR so that there is no need to query the NameDrive or root network repeatedly. This field specifies the time interval that the resource record can be cached before the NameDrive or root network should be queried once more. Format should be in seconds.
- `rdata` - Similar to the `Rdata` field with regular DNS systems, only instead of 32 bit IPV4 addresses, this field stores 64-bit dWeb addresses.

*NOTE:* There is no need for `Rdata Field Length` since dWeb discovery keys always have the same length (in octets).

#### _UPDATE_:
The `update` action allows a user to update a particular ND record for a domain. Only the `record_name`, `ttl` and `rdata` can be modified and all modifications must be signed by the domain's keys. The user must provide the following parameters with the `modify` action:
- `domain` - Domain associated with the ND record. Used for authentication and locating record.
- `record_name` - The ND record name
- `ttl`
- `rdata`

#### _DELETE_: 
The `delete action allows users to remove an ND record.
The user must provide the following parameters with the `delete` action:
- `domain` - Domain associated with the ND record. Used for authentication and locating record.
- `record_name` - The ND record name being deleted.

Many tools are available for interaction with the root system and managing a domain's ND records, like: 
- [ARISEN's Open RPC API](https://docs.arisen.network)
- [arisecli](https://arisen.network/arisecli)
- [dDNS CLI](https://github.com/peepsx/ddns-cli)

## Decentralized Top Level Domains (dTLDs)
As mentioned in the previous section, dTLDs can be put up for auction and won by anyone. While the winner has the sole power to determine who can sell the dTLDs they win, they cannot control the domain names associated with a dTLD after they have been registered by someone. As an example, @peeps can sell .dnet domains since they won the .dnet dTLD, but they cannot control the domains they register for others, nor do they ever see the keys associated with the domains that are registered.

This is because a domain name, regardless of the dTLD within the root name space it belongs to, is no different than a blockchain account. Unlike typical blockchain accounts, each domain operates around a permissions system, where different ND records can each be managed by different permission-levels and their associated keypairs. Changes to or the creation of ND records associated with a domain name, must be signed with the private key associated with the permission level that is ultimately associated with a domain name's ND record. All permission levels of a domain, fall under the domain's root permission level known as `owner`. Therefore, domain names are forever controlled by the user who possesses the `owner` keypair, which can reset the keypairs related to any permission level existing beneath it. 

For example, a domain name can have unlimited ND records, which means its RRs can be stored across many NameDrives, each of which contain a distributed database that stores the RRs, just liike RRs  can be stored across many nameservers with traditional DNS. With this setup, one ND record could be under the control of one permission level and another ND could be under the control of an entirely different permission level. This further protects ND records from being altered by hackers or manipulated by hackers. As an added bonus, like accounts, domains can hold a multi-currency balance, while also being able to send and receive payments.

## NameDrives
A NameDrive (ND) is the dWeb's version of a "Nameserver" (NS) and is where a domain's "Resource Records" (RRs) are stored. This relieves the "Root Network" from having to manage every single RR from every single domain on the dWeb, while also further decentralizing the domain name system itself amongst domain holders. NameDrives contain a distributed database, like a [dDatabase](https://github.com/distributedweb/ddatabase), [dAppDB](https://github.com/distributedweb/dappdb) or a [dWebTrie](https://github.com/distributedweb/dwebtrie), which contain a domain or multiple domain's RRs. 

When multiple domain-based databases are created, a dDNS management software should utilize [dWebStore](https://github.com/distributedweb/dwebstore) for managing multiple databases within a single NameDrive. This enables the dDNS management software to easily keep both databases in sync with one another, while also insuring that both databases are derived from the same "default" [dDatabase](https://github.com/distributedweb/ddatabase) and the default's master keypair (not to be confused with the ARISEN keys used for interfacing with the Root Network). dWebStore exposes a dDatabase factory and a set of functions for managing generated dDatabases, where all writable dDatabase keys are derived from a single master keypair.

## dDNS Database
Each database within a NameDrive is based on a hierarchical database containing RRs that include the `rdata`, `ttl`, `class` and `type` type for each of a domain's records (`record_name`). As mentioned in the previous section, a NameDrive could be used by multiple domains (also called zones) just like a name server is. Similar to traditional DNS management systems, dDNS management systems manage domains via separate zones, in the form of separate databases within a NameDrive. A NameDrive should store the RRs of multiple domains within totally separate databases that derive from the "default" database. Each database should be named after the domain it represents. For example, the database for domainA.dcom should be named `domainA.dcom.db`.

Below is an easy to understand diagram of what a NameDrive looks like, that manages multiple domains:

```
NameDrive (nd1.domain.dcom) (ND record points to discovery key of `default`)
- Default dDatabase (master key [used for ND record on Root System])
== domainA.dcom.db (derived from default)
== domainB.dcom.db (derived from default)
```

Using dWebStore, all the databases that derive from the "default" can be easily managed. For example, the database associated with domainA.dcom, on the nd1.domain.dcom NameDrive, can easily be synced with domainA.dcom's database on the nd2.domain.dcom NameDrive, since one dDatabase can easily replicate with another, almost instantaneously.

### Distributed P2P Database
Every NameDrive's discovery key is broadcasted to the dWeb's Distributed Hash Table (DHT) where requestors on the dWeb can easily discovery the NameDrive and make requests on its databases with simple `get` commands via the [dDatabase API](https://github.com/distributedweb/ddatabase).

When creating an ND record for a domain name on the root network, the `rdata` field should have the discovery key of the NameDrive, so that when the root system is queried for a domain's ND record(s), it will return the location of the NameDrive, allowing the user's client to query the records it's seeking from a particular domain, via its dedicated dDatabase within the NameDrive. Those who are accessing a domain's resources and choose to seed a particular NameDrive, do not need to cache records or rely on TTLs, since changes to any of the dDatabases within the NameDrive are live-replicated to the seeder's computer.

In other words, a particular NameDrive, holding the dDNS records of 3 different popular domains, could be distributed by and replicated amongst hundreds if not thousands of seeders across the world. Compare that to a few stagnant name servers holding the DNS records of 3 different popular domains and you will understand the clear difference between dDNS and DNS.

## Resource Records (RRs)
A domain's collection of RRs are organized in a distributed database that are shared amongst the peers who frequently access the NameDrive a domain's distributed database resides within. It's important to note that a "NameDrive" is simply a [dDrive](https://github.com/distributedweb/ddrive), that contains distributed databases each of which contain the RRs of the domain it's named after.

### Resource Record Format
An RR must have the following format:

- `record_name` - (Variable length) - This is the name of the record. For example, if trying to create a record "test.domainA.dcom", the record_name would simply be "test". Since the name of the database is "domainA.dcom", there is no reason to place this in the database. A dDNS system should already realize what domain it's dealing with, far before it ever queries for a record.
- `type` - Type should be one of many [Resource Record Types](#resource-record-types)
- `class` - Class should be one of many [Resource Record Classes](#resource-record-classes)
- `ttl` - Typically when an RR is retrieved from a NameDrive, the retriever will cache the RR so that it doesn't have to repeatedly query the domain's distributed database within the NameDrive. Browsers like [dBrowser](https://dbrowser.com) honor the TTL in an RR, which should specify the time interval that the RR should be cached, before the source NameDrive should be consulted once again.
- `rdata` - Similar to the `Rdata` field with regular DNS systems. The only difference is that instead of 32 bit IPV4 addresses, the rdata field can only store 64 bit addresses.

### Resource Record Types
| Type | Description |
| --- | --- | 
| D | A dDrive address. This RR type typically routes the name of a website or an app to the dDrive they're located in. |
| CNAME | Canonical name. Specifies an alias name for another RR and maps this to the canonical (true) name. | 
| MINFO | Decentralized mailbox or mail list information. Maps a mailbox or mail list name to a P2P mail host. |
| DMAIL | Mail exchange. Identifies the system(s) via which mail to the queried domain name should be relayed. | 
| ND | Authoritative NameDrive for this domain. | 
| SRV | Provides the name of a P2P service that is mapped to the service's discovery key. | 
| TXT | Arbitrary text. Provides a way to add text comments to the database. |

### Resource Record Classes
| Type | Description | 
| --- | --- | 
| DW | Identifies the dWeb protocol family | 
| DM | Identifies the dMesh protocol family |

## Name Resolution
Each query begins at a name resolver located in the user's host system. Each resolver is configured to know the name and address of a local dDNS NameDrive. The local NameDrive is used to cache RRs from various domains that the user accesses, only for the specified TTL for each RR that's cached.

Like with typical NameDrives, cached RRs for different domains are stored in separate databases within the local NameDrive, just as they are within public authoritative NameDrives.

If a domain or RR for the specified domain are not found within the local NameDrive, the root system is queried for a domain's ND record(s). Most domains only need one ND. Since a NameDrive is not a server, it's completely distributed amongst peers and will probably never go offline - especially if an ND uses an "always-online" [Peer Service](https://github.com/peepsx/peer-service). With that in mind, for most use-cases, one NameDrive will do - but for the ultimately redundancy - two will work just as well. 

### Querying The Root System
Querying the Root System for a domain's ND servers is quite easy if you use the [@ddns/lookup](https://github.com/peepsx/ddns-lookup) module. Using this Node module, you can perform the lookup like so:

```
const import { Lookup } from '@ddns/lookup'

const lookup = new Lookup (domainName);

lookup.on('found', key => {
  console.log(key); // displays key to NameDrive
});
```

This should return ND records like this:

```js
{
queriedRoot: "https://greatchain.arisennodes.io",
NDs: [ {recordName, discoveryKey, TTL}, ...]
}
```

### Locating Peers Swarming The NameDrive
Once the discovery key for an ND record has been successfully queried via the Root Network, a lookup for the discovery key should be performed on the dWeb via dWeb's DHT, to locate the peers whom the specified NameDrive can be downloaded from. Using the [@dwebswarm/dht](https://github.com/distributedweb/dwebswarm-dht) library, we can easily get a list of peers, like so:

```js
const dht = require ('@dwebswarm/dht')

const node = dht ({
  ephemeral: true
})

const lookup = node.lookup(ndDiscoveryKey)
  .on('data', console.log)
  .on('end', () => {
    node.destroy();
  });
```

The returned stream looks like this:

```js
{
node: { host, port },
//List of peers
peers: [ { host, port }, ...],
// List of LAN peers
localPeers: [ { host, port }, ...]
}
```

If you pass the callback the stream will be error handled and buffered.

### Joining The NameDrive Swarm
You could also use [dWebSwarm](https://github.com/distributedweb/dwebswarm) to simply join the NameDrive's "swarm" of peers, like so:

```js
const dwebswarm = require('dwebswarm')

const swarm = dwebswarm();

swarm.join(ndDiscoveryKey, {
  lookup: true, // find and connect to peers of NameDrive
  announce: true // become a seed/point of connection for other peers
});

swarm.on('connection', (socket, details) => {
  console.log('New Connection!', details);
  // You can now use the socket as a stream
  process.stdin.pipe(socket).pipe(process.stdout);
});
```

### Downloading The NameDrive
Probably the simplest way of all to simply download the NameDrive and consume its data, would be running [dDrive Daemon](https://github.com/distributedweb/ddrive-daemon) in the background (which dBrowser does by default) and use [dDrive Daemon Client](https://github.com/distributedweb/ddrive-daemon-client) to communicate with the daemon's gRPC-based API like so:

```js
const { DDriveClient } = require('ddrive-daemon-client')
const client = new DDriveClient('localhost:3101', your_access_token')
// If the client and daemon are on the same machine, the endpoint/token can be read
// from a common location (by default, at "~./ddrive")

const namedrive = "nd1.domainA.dcom"
let getND = client.fuse.mount(domain, {opts: opts},  () => {
  console.log("NameDrive Downloaded")
});
```

The above fetches the NameDrive and its contents and stores it at ~DDrive/nd1.domainA.dcom/

### Querying The Domain Database
Once the NameDrive (dDrive) has been downloaded, the databases within it can be queried, like so:

```js
var dappdb = require('dappdb')
var db = dappdb('./nd1.domainA.dcom/domainA.db', { valueEncoding: 'utf-8' })
db.get ('/www', function (err, nodes) {
  if (err) throw err
  console.log('www keys' + nodes[i].value)
});
```

The located key points to another dDrive or decentralized service where the requestor's journey will end. Essentially a requestor performs an ND lookup at the Root Network, downloads a given NameDrive and queries a domain's database held within the NameDrive for a specified record. At that point, the client can resolve the dWeb key that the actual record points to. Compare that to a tradition DNS lookup:

#### 1. Local Lookups
- Regular DNS: Performs a lookup on the user's local nameserver. If record is found, content is downloaded from the IP address related to the queried record.
- dDNS: Performs a lookup on the user's local NameDrive. If record is found, dDrive or content is downloaded that relates to the dWeb discovery key, related to the queried record.

#### 2. Root Lookup
- Regular DNS: Queries the Internet's Root DNS servers for the domain's authoritative name server(s)
- dDNS: Queries the dWeb's Root dDNS database for the domain's authoritative name servers (ND record(s))

#### 3. Name Resolution
- Regular DNS: Queries the nameserver for the `rdata` related to the record, which returns and IP address.
- dDNS: Downloads the NameDrive (few kilobytes on average) and queries the domain's database for the `rdata` related to the record, which returns a dWeb key. User has the option to seed the NameDrive and the domain's records to others.

#### 4. Resolving Content
- Regular DNS: Browser downloads content over a protocol like HTTP, which derives from a single server or network of servers.
- dDNS - Browser downloads dDrive or data related to the record's dWeb discovery key via possibly thousands of peers that are seeding the dDrive or data related to the record.

Not only is a NameDrive and its RRs further decentralized amongst those who seed the NameDrive, but the dDrives or distributed content/data those RRs point to are also seeded by those who consume them. That creates a truly decentralized web from end-to-end.

## Database Schema:
Without a database schema for storing Resource Records in a key-value store (KV store), one browser may have a difficult time querying data from one NameDrive-based database and one that doesn't follow the below schema.

RRs should take on the following standard format:
| Key | Value | Example |
| --- | --- | --- |
| /{recordName}/ | Record name | /www/, www |
| /{recordName}/ttl | Specified TTL | /www/ttl, 30 |
| /{recordName}/type | RR Type | /www/type, D |
| /{recordName}/class | RR Class Type | /www/class, DW |
| /{recordName}/rdata | dWeb discovery key related to record | /www/rdata, {dWeb key} |

The above standard must be used to be compatible with dWeb browsers like [dBrowser](https://dbrowser.com).

## Conclusion
dDNS brings many of the components of a traditional DNS system into a truly peer-to-peer and decentralized domain name system, so that the learning curve for developers is truly minimal. dDNS allows clients to easily resolve a decentralized domain to a dWeb key, by resolving resource records via distributed NameDrives that are in-turn seeded and live-replicated amongst its peers. dDNS is far more decentralized and distributed than a traditional DNS systems and is the icing on the cake when it comes to the dWeb achieving true end-to-end decentralization. It also makes the dWeb and its' websites and apps easily accessible by anyone through the use of easy-to-remember names, rather than 64 digital hexadecimal discovery keys.

### Implementations
- [@ddns/resolve](https://github.com/peepsx/ddns-resolve) - Module for resolving dDNS records within a NameDrive.
- [@ddns/lookup](https://github.com/peepsx/ddns-lookup) - Module for resolving a NameDrive key for a domain. 
- [dNames Contract](https://github.com/peepsx/dnames-contract) - Contract used for managing the ND database on ARISEN, as well as related actions.
- [@ddns/cli](https://github.com/peepsx/ddns-cli) - Used for setting ND records on ARISEN, creating NameDrives and creating dDNS records for a domain.

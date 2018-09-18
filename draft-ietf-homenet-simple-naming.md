---
ipr: trust200902
docname: draft-ietf-homenet-simple-naming-03
cat: info
obsoletes: ''
updates: ''
pi:
  toc: 'yes'
  compact: 'yes'
  symrefs: 'yes'
title: Simple Homenet Naming and Service Discovery Architecture
abbrev: Simple Homenet Naming/SD Arch
date: 2018-07-02
author:
- ins: T. Lemon
  name: Ted Lemon
  org: Nibbhaya Consulting
  street: P.O. Box 958
  city: Brattleboro
  region: Vermont
  country: United States of America
  code: '05301'
  email: mellon@fugue.com
- ins: D. Migault
  name: Daniel Migault
  org: Ericsson
  street: 8400 boulevard Decarie
  city: Montreal, QC H4P 2N2
  country: Canada
  facsimile: ''
  email: daniel.migault@ericsson.com
  uri: ''
- ins: S. Cheshire
  name: Stuart Cheshire
  org: Apple Inc.
  street: 1 Infinite Loop
  city: Cupertino
  region: California
  code: '95014'
  country: USA
  phone: "+1 408 974 3207"
  email: cheshire@apple.com


normative:
  RFC1034: 
  RFC1035: 
  RFC1918: 
  RFC2132: 
  RFC2136: 
  RFC3397:
  RFC3646:
  RFC6303:
  RFC6762: 
  RFC6763: 
  RFC7228:
  RFC7336:
  RFC7368: 
  RFC7556: 
  RFC7788:
  RFC8106: 
  I-D.ietf-dnssd-hybrid: 
  I-D.sctl-dnssd-mdns-relay: 
  I-D.sctl-service-registration: 
  I-D.ietf-mif-mpvd-ndp-support: 
  I-D.ietf-tokbind-https: 
  RFC8375: 
  I-D.sctl-discovery-broker: 
informative:
  I-D.ietf-mboned-ieee802-mcast-problems:

--- abstract




This document describes how names are published and resolved on
homenets, and how hosts are configured to use these names to discover
services on homenets. It presents the complete architecture, and
describes a simple subset of that architecture that can be used in
low-cost homenet routers.


--- middle


# Introduction


This document is a homenet architecture document. The term 'homenet'
refers to a set of technologies that allow home network users to have
a local-area network (LAN) with more than one physical link and,
optionally, more than one internet service provider. Home network
users are assumed not to be knowledgeable in network operations, so
homenets automatically configure themselves, providing connectivity
and service discovery within the home with no operator intervention.
This document describes the aspect of homenet automatic configuration
that has to do with service discovery and name resolution.


The homenet naming architecture consists of two parts: the simple
naming architecture, and the advanced naming architecture. The
advanced architecture provides approximate parity of features with a
managed network, including the ability to publish services on the
internet. The simple architecture provides a minimal set of features
required to enable seamless service discovery on a multi-link home
network, but does not attempt to provide feature parity with a managed
LAN.


This document begins by presenting a motivational list of requirements
and considerations, which should give the reader a clear idea of the
scope of the problem being solved. It then explains how each
requirement is addressed, and provides references for relevant
standards documents describing the details of the implementation. Some
requirements are not satisfied by the simple architecture; these are
discussed in this document, but explained in more detail in the
Advanced Homenet Naming Architecture document, which is to follow.




# Requirements


Name service on a local area network (LAN) requires the following:


* Name: a forward domain under which information about local services will
  be published


* Authority: a name server that is authoritative for at least one
  forward domain and one or two reverse domains that are applicable to
  that network and is capable of signing and publishing the zones
  using DNSSEC


* Resolution: a full-service caching DNS resolver that fully supports EDNS(0)
  and queries with the DO bit set


* Publication: a mechanism that


  * allows services on the LAN to publish information about the services they
    provide


  * allows services to publish information on how to reach them


  * manages the lifetime of such information, so that it persists long
    enough to prevent spoofing, but protects end users from seeing
    stale information




* Host configuration: one or more automatic mechanisms (e.g. DHCP or RA) that
  provide:


  * caching resolver information to hosts on the LAN


  * information about how services on the LAN can publish information




* Trust: some basis for trusting the information that is provided by
  the service discovery system




## Managed LAN versus Homenet


A managed network is one that has a (human) manager, or operator.  The
operator has authority over the network, and the authority to publish
names in a forward DNS tree, and reverse names in the reverse tree.
The operator has the authority to sign the respective trees with
DNSSEC, and acquire TLS certificates for hosts/servers within the
network.


On a managed LAN, many of these services can be provided by
operators. When a new printer is added to the network, it can be added
to the service discovery system (the authoritative server)
manually. When a printer is taken out of service, it can be
removed. In this scenario, the role of "publisher" is filled by the
network operator.


In many managed LANs, establishment of trust for service discovery is
simply on the basis of a belief that the local resolver will give a
correct answer. Once the service has been discovered and chosen, there
may be some security (e.g., TLS) that protects the connection to the
service, but the trust model is often just "you're connected to a
network you trust, so you can trust the printer that you discovered on
this network."


A homenet does not have an operator, so functions that would normally
be performed by the operator have to happen automatically. This has
implications for trust establishment—since there is no operator
controlling what services are published locally, some other mechanism
is required for basic trust establishment. 


Additionally, whereas in a managed LAN with multiple links to the Internet, the network operator can configure the network so that multihoming is handled seamlessly, in a homenet, multihoming must be handled using multiple provisioning domains {{RFC7556}}.


This is an issue that needs to be addressed in the naming architecture because Content Delivery Networks (CDNs) often use the DNS as a way to encourage hosts to use nearby content servers ({{RFC7336}} Section 2.1.1).   This information is only valid in the context of a particular ISP; using the a DNS response from one ISP to contact a server through another ISP can result in poor performance or even failure to access the referenced resource.




## Homenet-specific considerations


A naming architecture for homenets therefore adds the following considerations:


* All of the operations mentioned here must reliably function automatically,
  without any user intervention or debugging.


* Because user intervention cannot be required, naming conflicts must
  be resolved automatically, and, to the extent possible,
  transparently.


* Devices that provide services must be able to publish those services
  on the homenet, and those services must be available from any part
  of the homenet, not just the link to which the device is attached.


* Homenets must address the problem of multiple provisioning domains,
  in the sense that the DNS may give a different answer depending on
  whether caching resolvers at one ISP or another are queried.




An additional requirement from the Homenet Architecture {{RFC7556}} is
that hosts are not required to implement any homenet-specific
capabilities in order to discover and access services on the
homenet. This architecture may define optional homenet-specific
features, but hosts that do not implement these features must work on
homenets.






# Terminology


This document uses the following terms and abbreviations:


HNR
: Homenet Router




SHNR
: Homenet Router implementing simple homenet naming architecture




AHNR
: Homenet Router implementing advanced homenet naming architecture




ISP
: Internet Service Provider




Forward Mapping
: A mapping between a host name or service name and some information
  about that host or service.




Reverse Mapping
: A mapping between an IP address and the host that has that IP address.






# Name


In order for names to be published on a homenet, it is necessary that
there be a set of domain names under which such names are
published. These domain names, together, are referred to as the "local
domains." By default, homenets use the reserved domain 'home.arpa.' {{RFC8375}}
for publishing names for forward lookups. So a host called 'example'
that published its name on the homenet would publish its records on
the domain name 'example.home.arpa.'. Because 'home.arpa.' is used by
all homenets, it has no global meaning, and names published under the
domain 'home.arpa' cannot be used outside of the homenet on which they
are published.


Homenet routers that implement may also be
configured with a global domain. Support for delegation of a global
name is optional for routers that implement authoritative name service,
and not recommended for routers that don't.


In addition to the name, which defaults to 'home.arpa.', names are
needed for reverse lookups. These names are dependent on the IP
addressing used on the homenet. If the homenet is addressed with IPv4,
a reverse domain corresponding to the IPv4 subnet {{RFC1034}} section
5.2.1 should be constructed. For example, if the homenet is allocating
local IP addresses out of net 10 {{RFC1918}}, a domain,
'10.in-addr.arpa' would be required. Like 'home.arpa.',
'10.in-addr.arpa' is a locally-served zone, and has no validity
outside of the homenet.


If the homenet is addressed with IPv6, it is expected to have a unique
local address prefix; subsets of this prefix will be advertised on
every link on the homenet. Every service on the homenet that supports
IPv6 is expected to be reachable at an address that is configured
using the ULA prefix. Therefore there is no need for any IPv6 reverse
zone to be populated other than the ULA zone.  So for example if the
homenet's ULA prefix is fd00:2001:db8::/48, then the reverse domain
name for the homenet would end in '8.b.d.0.1.0.0.2.0.0.d.f.ip6.arpa'.




# Authority


The authority role is provided by a name server that is authoritative
for each of the local domains. SHNRs provide authoritative service for
the homenet using DNSSD Discovery Broker
{{I-D.sctl-discovery-broker}}. SHNRs also provide Discovery Relay
service {{I-D.sctl-dnssd-mdns-relay}}. On a homenet that has only
SHNRs, each SHNR individually provides authoritative service for the
whole homenet by using Discovery relays to discover services off the
local link.


The Discovery Proxy model relies on each link having its own
name. However, homenets do not actually have a way to name local links
that will make any sense to the end user.  Consequently, this
mechanism will not work without some tweaks. In order to address this,
homenets will use Discovery Brokers {{I-D.sctl-discovery-broker}}. The
discovery broker will be configured so that a single query for a
particular service will be successful in providing the information
required to access that service, regardless of the link it is on.


Artificial link names will be generated using HNCP {{RFC7788}}.  These should only
be visible to the user in graphical user interfaces in the event that
the same name is claimed by a service on two links.  Services that are
expected to be accessed by users who type in names should use
{{I-D.sctl-service-registration}} if it is available.


It is possible that local services may offer services available on IP addresses
in public as well as ULA prefixes.
Homenet hybrid proxies MUST filter out global IP addresses, providing
only ULA addresses, similar to the process described in section 5.5.2
of {{I-D.ietf-dnssd-hybrid}}.


This filtering applies to queries within the homenet; it is
appropriate for non-ULA addresses to be used for offering services,
because in some cases end users may want such services to be reachable
outside of the homenet.  Configuring this is however out of scope for
this document.




# Resolution


Name resolution is provided by a local DNS cache or proxy on the
homenet, henceforth the "local resolver."  All host queries are sent
to this local resolver.  The local resolver may either act as a
full-service caching resolver, or as a DNS proxy.  Its responsibility
with respect to queries on the homenet is to notice queries for names
for which the local authoritative server is authoritative.  Queries
for such names are handled through the local authoritative server.
Queries for all other names are resolved either by forwarding them to
an ISP-provided full service resolver, or by providing the full
service resolver function locally.




# Publication


Put in some diagrams here.


## DNS Service Discovery Registration Protocol


Support for the DNSSD Service Registration Protocol (DNSSD SRP) {{I-D.sctl-service-registration}} is not required for homenets.   However, HNRs that are capable of supporting DNSSD SRP SHOULD support it.


Support for SRP provides several advantages over DNSSD Discovery Proxy.   First, DNSSD SRP provides a secure way of claiming service names.   Second, a claimed name is valid for the entire network covered by the SRP service, not just an individual link, as is the case with mDNS.   Third, SRP does not use multicast, and is therefore more reliable on links with constrained multicast support {{I-D.ietf-mboned-ieee802-mcast-problems}}.


Support for the DNSSD SRP service is not sufficient to achieve full deployment of DNSSD SRP: it is also necessary that services advertise using DNSSD SRP.   Requiring such support is out of scope for this document; our goal is simply to specify path for upgrading to this service so that as adoption of SRP increases on devices providing service, it can actually be used.


DNSSD SRP requires stateful authoritative DNS service; this may be beyond the capability of the minimal Homenet router.  However, more capable Homenet routers should provide this capability.


Homenet routers that provide DNSSD SRP support MUST also provide DNS Discovery Broker service {{I-D.sctl-discovery-broker}}.   The Discovery Broker can combine answers from Discovery Proxies with answers from the DNSSD SRP service to provide a single DNSSD service.


## Configuring Service Discovery


Clients discovering services using DNS-SD {{RFC6763}} follow a
two-step process.  The first step is for the client device to
determine in which domain(s) to attempt to discover services.  The
second step is for the client device to then seek desired service(s)
in those domain(s).  For an example of the second step, given the
desired service type "IPP Printing", and the domains "local" and
"meeting.ietf.org", the client device forms the queries
"_ipp._tcp.local. PTR ?" (resolved using Multicast DNS) and
"_ipp._tcp.meeting.ietf.org PTR. ?" (resolved using Unicast DNS) and
then presents the combined list of results to the user.


The first step, determining in which domain(s) to attempt to discover
services, is performed in a variety of ways, as described in Section
11 of the [DNS-Based Service Discovery specification](#RFC6763).


The domain "local" is generally always in the set of domains in which
the client devices attempt to discover services, and other domains for
service discovery may be configured manually by the user.


The device also learns additional domains automatically from its
network environment.  For this automatic configuration discovery,
special DNS queries are formulated.  To learn additional domain(s) in
which to attempt to discover services, the query string
"lb._dns_sd._udp" is prepended onto three different kinds of
"bootstrap domain" to form DNS queries that allow the device to learn
the configuration information.


One of these bootstrap domains is the fixed string "local".  The
device issues the query "lb._dns_sd._udp.local. PTR ?" (resolved using
Multicast DNS), and if any answers are received, then they are added
to the set of domains in which the client devices attempt to discover
services.


Another kind of these bootstrap domains is name-based, derived from
the [DHCPv4 "domain name" option (code 15)](#RFC2132) (for IPv4) or
the [DNS Search List (DNSSL) Router Advertisement option](#RFC8106)
(for IPv6).  If a domain in the DNSSL is "example.com", then the
device issues the query "lb._dns_sd._udp.example.com. PTR ?" (resolved
using Unicast DNS), and if any answers are received, then they are
likewise added to the set of domains in which the client devices
attempt to discover services.


Finally, the third kind of bootstrap domain is address-based, derived
from the device's IP address(es) themselves.  If the device has IP
address 192.168.1.100/24, then the device issues the query
"lb._dns_sd._udp.0.1.168.192.in-addr.arpa. PTR ?" (resolved using
Unicast DNS), and if any answers are received, then they are also
added to the set of domains in which the client devices attempt to
discover services.


Since there is an HNR on every link of a homenet, automatic
configuration could be performed by having HNRs answer the
"lb._dns_sd._udp.local. PTR ?" (Multicast DNS) queries.  However,
because multicast is slow and unreliable on many modern network
technologies like Wi-Fi, we prefer to avoid using it.  Instead we
require that a homenet be configured to answer the name-based
bootstrap queries.  By default the domain in the DNSSL communicated to
the client devices will be "home.arpa", and the homenet will be
configured to correctly answer queries such as
"lb._dns_sd._udp.example.com. PTR ?", though client devices must not
assume that the name will always be "home.arpa". A client could be
configured with any valid DNSSL, and should construct the appropriate
bootstrap queries derived from the name(s) in their configured DNS
Search List.


HNRs will answer domain enumeration queries against every IPv4 address
prefix advertised on a homenet link, and every IPv6 address prefix
advertised on a homenet link, including prefixes derived from the
homenet's ULA(s).  Whenever the "\<domain>" sequence appears in this
section, it references each of the domains mentioned in this
paragraph.


Homenets advertise the availability of several browsing zones in the
"b._dns_sd._udp.\<domain>" subdomain.  By default, the 'home.arpa'
domain is advertised.  Similarly, 'home.arpa' is advertised as the
default browsing and service registration domain under
"db._dns_sd._udp.\<domain>", "r._dns_sd._udp.\<domain>",
"dr._dns_sd._udp.\<domain>" and "lb._dns_sd._udp.\<domain>".


In order for this discovery process to work, the homenet must provide
authoritative answers for each of the domains that might be queried.
To do this, it provides authoritative name service for the 'ip6.arpa'
and 'in-addr.arpa' subdomains corresponding to each of the prefixes
advertised on the homenet.  For example, consider a homenet with the
192.168.1.0/24, 2001:db8:1234:5600::/56 and fc01:2345:6789:1000::/56
prefixes.  This homenet will have to provide a name server that claims
to be authoritative for 1.168.192.in-addr.arpa,
6.5.4.3.2.1.8.b.d.0.1.0.0.2.ip6.arpa and
0.0.9.8.7.6.5.4.3.2.1.0.c.f.ip6.arpa.


An IPv6-only homenet would not have an authoritative server for a
subdomain of in-addr.arpa.  These public authoritative zones are
required for the public prefixes even if the prefixes are not
delegated.  However, they need not be accessible outside of the
homenet.


It is out of the scope of this document to specify ISP behavior, but
we note that ISPs have the option of securely delegating the zone, or
providing an unsigned delegation, or providing no delegation.  Any
delegation tree that does not include an unsigned delegation at or
above the zone cut for the ip6.arpa reverse zone for the assigned
prefix will fail to validate.


Ideally, an ISP should provide a secure delegation using a
zone-signing key provided by the homenet.  However, that too is out of
scope for this document.  Therefore, an ISP that wishes to support
users of the simple homenet naming architecture will have to provide
an unsigned delegation.  We do not wish, however, to discourage
provisioning of signed delegations when that is possible.






# Host Configuration


Hosts on the homenet receive a set of resolver IP addresses using
either DHCP or RA.  IPv4-only hosts will receive IPv4 addresses of
resolvers, if available, over DHCP.  IPv6-only hosts will receive
resolver IPv6 addresses using either stateful (if available) or
stateless DHCPv6, or through the Recursive DNS Server Option
({{RFC8106}}, Section 5.1) in router advertisements.


All Homenet routers provide resolver information using both stateless
DHCPv6 and RA; support for stateful DHCPv6 and DHCPv4 is optional,
however if either service is offered, resolver addresses will be
provided using that mechanism as well.




# Globally Unique Name


Automatic configuration of a globally unique name for the homenet is
out of scope for this document.  However, homenet servers MUST allow
the user to configure a globally unique name in place of the default
name, 'home.arpa.'  By default, even if configured with a global name,
homenet routers MUST NOT answer queries from outside of the homenet
for subdomains of that name.




# DNSSEC Validation


DNSSEC Validation for the 'home.arpa' zone and for the locally-served 'ip6.arpa and 'in-addr.arpa' domains is not possible without a trust anchor.  Each HNR on a homenet generates its own private/public key pair that can serve as a trust anchor.  These keys are shared using HNCP {{RFC7788}}. HNRs MUST NOT use pre-installed keys: each HNR MUST generate its own key.   TODO: write an HNCP update that describes this.   Do we need key rollover?


## How trust is established


Every homenet that publishes naming and service discovery information using the 'home.arpa' special-use domain will have its own set of keys that are used to sign responses.   Homenets can therefore be differentiated by noticing that the set of keys published in the DNSKEY RRset on 'home.arpa' is different.   Since homenets share keys using HNCP, it should never be the case that the same key is present on more than one homenet, and it should never be the case that some keys are not known.


In order to establish trust, each homenet agrees on a set of keys that will be published.   Whenever a key is added to the set, every participating HNR generates a new RRSIG record that covers the keys.   These RRSIG records are shared among the HNRs using HNCP.   In principle, every key is signed and published.


There is no upper bound other than cost and practicality on the number of HNRs that may be present on a homenet.  Because each HNR has its own key and generates its own RRSIG, in principle the set of DNSKEYs and RRSIGs published in the zone could be quite large.   However, because not every RRSIG must be validated every time, this is really the only practical constraint.


HNRs MAY have a user setting, which we will call NOSIGN,  that disables key publication for that HNR.   When NOSIGN is enabled, if there are more than three HNRs present on the network for which NOSIGN is not disabled, those HNRs for which NOSIGN is disabled are not required to publish their keys in the DNSKEY RRset at 'home.arpa.'


HNRs that provide DNSSD SRP service MUST by default disable NOSIGN.   HNRs that do not provide DNSSD SRP service MAY enable NOSIGN by default.   On a network where fewer than three HNRs have NOSIGN disabled, but more than three HNRs are present, HNCP will be used to elect an HNR for which NOSIGN will be disabled.   This will repeat until there are three HNRs on the network for which NOSIGN is disabled.   If the number of HNRs on the network for which NOSIGN is disabled is more than three, HNRs for which NOSIGN was enabled by default may re-enable NOSIGN; the router that disabled NOSIGN most recently re-enables it first.


The reason for this process is to avoid truly excessive responses, while ensuring that there is a stable set of signers that persists over time.   HNRs that are acting as signers may from time to time be removed from the homenet; when this happens, the proposed process should result in the remaining keys being as old as possible, which makes it more likely that hosts that connect to a homenet infrequently will still successfully identify it.


Homenets have more than one zone: not only the home.arpa zone, but also the zones for each of the individual links in the homenet.   Some links are served by only one HNR; some are served by several.   Each HNR that provides such a zone will have a DS record for its key on the delegation for that zone in home.arpa, whether it is a signer for home.arpa or not.


A homenet must have one or more name servers that are authoritative for home.arpa.   If DNSSD SRP service is present, the domain that SRP updates is home.arpa, and so the primary SRP server is responsible for the content of the home.arpa zone.   However, many homenets will not have an SRP server; on these homenets, each HNR that is a signer also acts as an authoritative server for home.arpa.   If there is more than one SRP server on the homenet, one of the SRP servers will act as primary and the other(s) will act as secondaries.   (Describe the plumbing).


## Records published under home.arpa


This section should describe all the records published in the 'home.arpa' zone on HNRs that do not support SRP.   This is a subset of the records published when SRP is available, since services will be published directly under 'home.arpa.' when registered using SRP.


XXX actually describe what's published under home.arpa! Also, where should this section actually be? XXX


## Validating Host Behavior for DNSSEC


Hosts are not required to implement any special behavior for homenets.   If a host implements no special behavior, it will not be able to validate DNS responses for home.arpa.   But because the delegation of 'home.arpa' in the DNS is not signed, this will not prevent names in home.arpa from being resolved.   Hosts MAY implement homenet-specific trust anchor behavior.   Doing so will allow hosts both to identify specific homenet instances and to validate names under home.arpa on those homenet instances.


Hosts that wish to validate names on the homenet must keep track of the set of homenets to which they have connected.   Each homenet is identified by the set of keys published in the DNSKEY RRset on the name 'home.arpa.'   These keys will change over time as HNRs are added to or removed from the homenet. 


When a host connects to a particular homenet for the first time, it issues a query for the DNSKEY RRset published on 'home.arpa.'   It searches its list of homenets, checking to see if any of the keys in the response are present in the list of keys known to belong to any previously-visited homenet.


If it finds that there is no remembered homenet for which there is a matching key, then it is connected to a new homenet, and it creates a new record for that homenet and adds the keys it has been given to that record.   If appropriate, the host should prompt the user to indicate that it has discovered a new homenet and ask the user to supply a mnemonic name for that homenet.   Otherwise, it may construct a name for the homenet.


When a host connects to a particular homenet to which it has previously connected, it knows this because it finds a homenet record containing at least one key that matches a key in the answer to the 'home.arpa IN DNSKEY' query.   However, it is possible that the set of keys for that homenet has changed since the last time the host did this query; in that case, some keys may no longer be present in the current RRset, and some new keys may have been added.


   In order for the new DNSKEY response to be treated as belonging to the same homenet, the host must find a RRSIG record that covers the DNSKEY RRset and that is signed using a key that the host remembers.  This need not be one of the keys that was previously a signing key—any remembered DNSKEY from the home.arpa DNSKEY RRset is sufficient.    


If no such RRSIG record exists, 


The host marks keys that are missing as stale.   Any new keys that are seen must be validated using a previously-remembered key.   This validation can be done lazily: when some RRSIG under home.arpa needs to be validated, the chain of trust is followed up to 'home.arpa' according to normal DNSSEC rules.   The link in the chain of trust prior to validating 'home.arpa' will be signed with an RRSIG; if this RRSIG uses a key that has not been validated, then the host validates the key using some key from the homenet record that had previously been validated.   If no such key exists, the record fails validation.


Although validation of the top-level DNSKEY RRs in the DNSKEY RRset 
Hosts MAY maintain lists of keys that are known to exist on a homenet.   These
keys can then be used to validate DNSSEC-signed answers in the homenet.   Each
homenet is identified by the set of mutually-authenticated keys that is returned
by an IN DNSKEY query to the home.arpa name.   Each DNSKEY record returned in a response MUST be validated using each of the RRSIGs that apply to that key.
The entire set of keys is then associated with a single homenet; any one of these
keys can then be used for DNSSEC validation of records published in that particular
homenet.   Keys must not be assumed to be stable—HNRs may come and go from
a homenet, and so new keys may be added, and old keys may be lost.   A new key
can only be added by being signed by at least one pre-existing HNR key.


Note: the rationale for doing this is that there's no way that HNRs can have a single
secret key, because if they all know it, it's not a secret.   However, the work of validating
all of the keys is a bit of a combinatorial explosion; it might be good to think of a less
cross-product-y solution to this problem.   And of course I can think of lots of ways of
attacking this; should we insist on at least two known keys signing a new key, for
example?


Homenets that have been configured with a globally unique domain MUST
support DNSSEC signing of local names, and must provide a way to
generate a KSK that can be used in the secure delegation of the
globally unique domain assigned to the homenet.




# Support for Multiple Provisioning Domains


Homenets must support the Multiple Provisioning Domain Architecture
{{RFC7556}}.  Hosts connected to the homenet may or may not support
multiple provisioning domains.  For hosts that do not support multiple
provisioning domains, the homenet provides one or more resolvers that
will answer queries for any provisioning domain.  Such hosts may
receive answers to queries that either do not work as well if the host
chooses a source address from a different provisioning domain, or does
not work at all.  However, the default source address selection
policy, longest-match (CITE), will result in the correct source
address being chosen as long as the destination address has a close
match to the prefix assigned by the ISP.


Hosts that support multiple provisioning domains will be provisioned
with one or more resolvers per provisioning domain.  Such hosts can
use the IP address of the resolver to determine which provisioning
domain is applicable for a particular answer.


Each ISP has its own provisioning domain.  Because ISPs connections
cannot be assumed to be persistent, the homenet has its own separate
provisioning domain.


Configuration from the IPv4 DHCP server are treated as being part of
the homenet provisioning domain.  The case where a homenet advertises
IPv4 addresses from one or more public prefixes is out of scope for
this document.  Such a configuration is NOT RECOMMENDED for homenets.


Configuration for IPv6 provisioning domains is done using the Multiple
Provisioning Domain RA option (CITE).




# Using the Local Namespace While Away From Home


This architecture does not provide a way for service discovery to be
performed on the homenet by devices that are not directly connected to
a link that is part of the homenet.


# Expected Host Behavior


It is expected that hosts will fall into one of two categories: hosts that are able to discover
DNS browsing domains, and hosts that are not.   Hosts that can discover DNS browsing
domains can be expected to successfully use service discovery across the entire homenet.
Hosts that do not will only be able to discover services on the particular local subnet of the
homenet to which they happen to be attached at any given time.


This is not considered to be a problem, since it is understood by the authors that the vast
majority of hosts that are capable of doing mDNS discovery are also capable of doing DNSSD
discovery as described in {{RFC6763}}.


# Implementation


## Required Functions


Each HNR implements a minimum set of necessary functions.   Some HNRs may also implement optional functionality.   The following functions are REQUIRED to be present in any HNR:


### Discovery Proxy
The home network has at least one Discovery Proxy in order to make the services advertised through service discovery using mDNS available on any other links of the home network.  A single Discovery Proxy for the whole network may be sufficient.  Each Discovery Proxy defines a Proxy Zone attached to a specific subdomain. The subdomain can have a link granularity. The use of a single Discovery Proxy considers a single Proxy Zone for the whole home network.


### DHCPv4 Server


For IPv4, the DHCP server may be required for addressing, but this is out of scope for this document.   The function that is required of the DHCPv4 server by this document is that it advertise "home.arpa" using the Domain Name ({{RFC2132}} section 3.17) and Domain Search List ({{RFC3397}}).   The DHCP server must also advertise at least one Domain Name Server ({{RFC2132 section 3.8) option that points to an IPv4 address of the HNR, on which DNS queries for any domain name will be answered on port 53.


### DHCPv6 Server
For IPv6, DHCPv6 is not required.   If a DHCPv6 server is present, it MUST NOT specify any name other than "home.arpa" in the (etc) (or say MUST NOT conflict with RA)


is required to provide a DNS Recursive Name Server option {{RFC3646}} that lists an IPv6 address of an HNR that is able to answer queries for any domain name on port 53.   The DHCP server is also required to provide a Domain Name Search List option {{RFC3646}} that lists exactly one domain name, "home.arpa."


### Router Advertisements


Router Advertisements MUST contain ...


### DNS Proxy
Each HNR has a DNS Proxy which can differentiate between queries that are for domains on the homenet, queries for locally-served zones {{RFC6303}} not provided by the homenet, and queries that are for other names.


This proxy must have a list of link-specific subdomains and be able to forward queries for those subdomains to the Discovery Proxy that is answering for that link, or it must forward all queries for home.arpa and for locally served domains to a Discovery Broker that is present on the homenet.   Both capabilities are required.   Queries for names that are neither locally-served nor subdomains of home.arpa must be forwarded to an appropriate upstream DNS resolver provided by one of the ISPs to which the homenet is connected. What about MPvD?


## Optional Functions


### Discovery Broker


only one activated on the whole homenet. The home network has a single Discovery Broker. The Discovery Broker is believed to coordinates the resolution inside the home network.  First, the Discovery Broker is aware of the different Discovery Proxies and associated Proxy Zones.  In addition, the Broker is also aware of the Advertising Proxy that contains all resources registered by hosts or services.


(Daniel is talking about the DNSSD Registration Server here.)
(Talk about required behavior for Registration Server if present, since if present and done wrong it could break things.)
Advertising Proxy (at least one for the whole network). The home network has a single Advertising Proxy specific homenet-aware devices or services register to.


# HNR Behaviors


## A new host is joining the network


1. It is being provisioned with an IP address, and DNS Proxy via DHCPv6 option DNS Recursive Name Server option or the Neighbor Discovery Extension Recursive DNS Server Option. The DNS Proxy is always on the link so DNS Updates to the destination of the Advertising Proxies can be controlled.
2. If the host or service is not homenet-aware, it MAY perform service discovery using mDNS on the link it is hosted, and sends DNS queries to the DNS Proxy. Only the latest is considered in this document.
3. If the host/service is homenet aware, it performs service discovery using unicast DNS.  These DNS queries as well as those associated to the Global DNS are sent to the DNS Proxy.  The DNS Proxy is responsible to proceed to the appropriate resolution. The distinction between the different resolution is based on the "home.arpa" suffix.


## A new link is created


1. The HNR involved needs to extend the Discovery mechanism to that link.  Either the HNR implements a Discovery Proxy or a Discovery Relay.  The HNR MAY act as a Discovery Proxy, in which case it needs to discover the Discovery Broker and provides the necessary information so the Discovery Broker adds the new link domain - typically "linkxxx.home.arpa".  The name of the link is automatically configured from the IP prefix of the new link.  If there is no Discovery Broker, the HNR MUST be ready to play as a Discovery Broker in addition to the Discovery Proxy.  If the HNR is informed of the existence of a Discovery Discovery, the HNR may prefer to act as a Discovery Relay and attempt to associate with the Discovery Proxy.  In case the association fails, the HNR MUST fall back to instantiate a Discovery Proxy.
2. The HNR also needs to extends the Advertising Proxy to its link and is expected to instantiate a Advertising Proxy.
3. The HNR MUST also be able to provides the necessary information so the hosts and nodes attached to the link can perform the resolution via the DNS Proxy.
When the no DNS Proxy is found inside the homenet work, that is instantiated with a non local scope IP address, the HNR MUST instantiate a DNS Proxy.


## A host performs a DNS resolution or a service discovery lookup


1. A hosts that is not homenet-aware sends mDNS queries on its LAN and DNS queries on the DNS Proxy.
2. A host that is homenet aware sends all DNS unicast queries to the DNS Proxy.
3. The requests that are not intended for the Global DNS are identified by the "home.arpa" suffix.  These requests are sent to the Discovery Broker.  The Discovery Broker performs two resolutions one against the Advertising Proxies and one against the Discovery Proxies.  Results are aggregated back to the DNS Proxy.
4. Global DNS resolutions are forwarded to the resolver by the DNS Proxy.  The DNS Proxy may use different rules to select the appropriated resolver.  When the DNS requested is performed with a non local IP address, the DNS Proxy may use that IP address to identify the appropriated resolver.  When the IP address is of local scope, the DNS Proxy may partition the naming space between the resolvers.


# Management Considerations {#mgt}


This architecture is intended to be self-healing, and should not
require management.  That said, a great deal of debugging and
management can be done simply using the DNS Service Discovery
protocol.




# Privacy Considerations


Privacy is somewhat protected in the sense that names published on the
homenet are only visible to devices connected to the homenet.  This
may be insufficient privacy in some cases.


The privacy of host information on the homenet is left to hosts.
Various mechanisms are available to hosts to ensure that tracking does
not occur if it is not desired.  However, devices that need to have
special permission to manage the homenet will inevitably reveal
something about themselves when doing so.  It may be possible to use
something like HTTP token binding {{I-D.ietf-tokbind-https}} to
mitigate this risk.




# Security Considerations


There are some clear issues with the security model described in this
document, which will be documented in a future version of this
section.  A full analysis of the avenues of attack for the security
model presented here have not yet been done, and must be done before
the document is published.




# IANA considerations {#IANA}


No new actions are required by IANA for this document.


--- back


# Existing solutions


Previous attempts to automate naming and service discovery in the
context of a home network are able to function with varying degrees of
success depending on the topology of the home network.  Unfortunately,
these solutions do not fully address the requirements of homenets.


For example, Multicast DNS {{RFC6762}} can provide naming and service
discovery {{RFC6763}}, but only within a single multicast domain.


The Domain Name System provides a hierarchical namespace {{RFC1034}},
a mechanism for querying name servers to resolve names {{RFC1035}}, a
mechanism for updating namespaces by adding and removing names
{{RFC2136}}, and a mechanism for discovering services {{RFC6763}}.
Unfortunately, DNS provides no mechanism for automatically
provisioning new namespaces, and secure updates to namespaces require
that the host submitting the update have a public or symmetric key
that is known to the network and authorized for updates.  In an
unmanaged network, the publication of and authorization of these keys
is an unsolved problem.


Some managed networks get around this problem by having the DHCP
server do DNS updates.  However, this doesn't really work, because
DHCP doesn't provide a mechanism for updating service discovery
records: it only supports publishing A and AAAA records.


This partially solves the trust problem: DHCP can validate that a
device is at least connected to a network link that is actually part
of the managed network.  This prevents an off-network attacker from
registering a name, but provides no mechanism for actually validating
the identity of the host registering the name.  For example, it would
be easy for an attacker on the network to steal a registered name.


Hybrid Multicast DNS {{I-D.ietf-dnssd-hybrid}} proposes a mechanism
for extending multicast DNS beyond a single multicast domain.
However, in order to use this as a solution, some shortcomings need to
be considered.  Most obviously, it requires that every multicast
domain have a separate name.  This then requires that the homenet
generate names for every multicast domain.  These names would then be
revealed to the end user.  But since they would be generated
automatically and arbitrarily, they would likely cause confusion
rather than clarity, and in degenerate cases requires that the end
user have a mental model of the topology of the network in order to
guess on which link a given service may appear.


At present, the approach we intend to take with respect to
disambiguation is that this will not be solved at a protocol level for
devices that do not implement the registration protocol.

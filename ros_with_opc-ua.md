Author: [Mirko Bordignon](https://github.com/ipa-mirb)

This article is a work-in-progress collecting technical information, TODOs and comments from interested parties on the possibility to use the [OPC-UA](https://opcfoundation.org/about/opc-technologies/opc-ua/) industrial communication standard in ROS 2.0. Given the similarity in purpose and in order to ease the comparison with the current middleware of choice (DDS), the structure of the [ROS on DDS article](http://design.ros2.org/articles/ros_on_dds.html) is replicated as much as possible. The main sources of information used to compile this document are listed at the final section.


## Why consider OPC-UA?

The reasons behind the choice of using an end-to-end middleware rather than continuing to develop one built from scratch are clearly outlined in the [relevant paragraph](http://design.ros2.org/articles/ros_on_dds.html#an-end-to-end-middleware) of the original ROS on DDS article. However, some members of the audience that the ROS-Industrial initiative addresses, and that the two ROS-Industrial Consortia interact with, commented that OPC-UA should be considered as well in addition to DDS. The main reason is that it would be a very appealing feature to have for a robotics framework gaining popularity in more traditional robotics business areas, such as manufacturing and industrial automation, given its increasing significance and adoption as an industry standard.


### What is OPC-UA?

OPC Unified Architecture (OPC-UA) is not to be confused with the previous OLE for Process Control (OPC "Classic") series of standards. The two protocols, albeit coming from [the same foundation](https://opcfoundation.org/), differ significantly in terms of design. OPC-UA is possibly more than a middleware, aiming to provide "a platform independent service-oriented architecture" ([src](https://opcfoundation.org/about/opc-technologies/opc-ua/)) including not only communication capabilities but also information modeling ones. This is meant to provide an easy migration path to existing applications using OPC Classic, which can replace such legacy technology by merely using OPC UA's transport mechanism, or leverage also its modeling system and attach structure and semantics to the data. It is significant in this sense that DDS itself [is being considered](https://www.prosysopc.com/blog/opc-day-europe-2015/) as a transport protocol for use by OPC-UA.

A lot of material is available online outlining the purported advantages of OPC-UA, together with material -typically coming from vendors of competing systems- taking a critical stance towards it as well. For the sake of this discussion and to better motivate the cited recommendations to adopt it coming from industrial entities, we consider the following elements as representative enough of its significance in current and future industrial robotic systems:

- the OPC Foundation is backed by major manufacturers of industrial hardware (robots, PLCs, etc): while this does not necessarily mean much, as typically companies participate in multiple consortia (sometimes even related to competing/alternative technologies), private communications with ROS-Industrial Consortia members confirmed that industrial interest for OPC-UA is significant and that *its adoption could be a decisive factor* leading to the decision to support ROS and ROS-related activites;
- for the aforementioned reason, and given the OPC Foundation's recently announced "Open Shared Source" [strategy](https://opcfoundation.org/news/press-releases/2134/), OPC-UA has good chances to become the de facto standard for Internet of Things and [Industrie 4.0](http://www.automation.com/automation-news/industry/opc-ua-oked-as-communications-standard-for-industrie-40)-related initiatives, further increasing its current appeal;
- manufacturer of automation systems not traditionally considered robots (e.g., process machines), but which are approaching the RIC-Consortia to assess ROS as a means to achieve better software modularity, are looking at OPC-UA as the "lingua franca" for industrial system integration.

### Where did OPC-UA come from?

The OPC foundation released the first version of the standard in 2006. The IEC backed it by releasing it as IEC 62541 as well. The current stable version of the specifications, [available directly from the OPC foundation](https://opcfoundation.org/developer-tools/specifications-unified-architecture) (free registration required), is 1.02, released on July 2012.

### Technical Credibility

DDS has a pretty extensive list of deployments, with mission critical systems being well represented. Even though the older OPC versions were not universally well accepted from a technical standpoint (mostly due to their reliance on DCOM), OPC-UA as a complete overhaul backed by important companies in the industrial automation space (KUKA, Beckhoff, B&R, etc) seems much better. Hard figures are difficult to obtain, but given the number of professional products supporting it and informal reports from users found online, it is realistic to say that it has a significant market presence in the industrial automation business. Moreover, it is multi-platform and multi-language, with the OPC foundation developing and maintaining ANSI C/C++, .NET and Java implementations (with sample ones [available](https://opcfoundation.org/developer-tools/developer-kits-unified-architecture) from the foundation itself and further commercial ones from its members).

### Vendors and Licensing

[This website](http://www.opcconnect.com/) provides a good overview of the [commercially available SDKs](http://www.opcconnect.com/uakit.php) as well as of the 
free (though not necessarily open source) [ones](http://www.opcconnect.com/uafree.php). A list of open source implementations is maintained at [this page](https://github.com/acplt/open62541/wiki/List-of-Open-Source-OPC-UA-Implementations). Among the listed ones, the following seem to be in active development status as of August 2015:

- [open62541](http://open62541.org/): C, MPL
- [Node-OPCUA](http://node-opcua.github.io/): javascript & NodeJS, MIT License
- [FreeOPCUA](http://freeopcua.github.io/): C++, LGPL
- [python-opcua](https://github.com/FreeOpcUa/python-opcua), a sister project to FreeOPCUA implemented in pure python
- [OPC-UA Stack](https://github.com/digitalpetri/opc-ua-stack) and its sister project [OPC-UA SDK](https://github.com/digitalpetri/ua-server-sdk), Java, Apache License

According to the open shared source [strategy](https://opcfoundation.org/news/press-releases/2134/) mentioned early, the source of the OPC foundation [official stacks](https://opcfoundation.org/developer-tools/developer-kits-unified-architecture) will be made available to foundation members under the [RCL license](https://opcfoundation.org/license/rcl/1.00/opc-reciprocal-community-license-1.00.pdf). Although this does not imply the availability to the general public of official open source stacks or SDKs, it might change the scenario of stacks available as freeware from vendors in the near future.

### Ethos and Community

Similarly to what pointed out for DDS, the entities backing OPC-UA are probably quite different in spirit compared to what the ROS community is used to. However, given its status as a (de facto) Industrie 4.0 standard, this should not be a risk in terms of future support. Being locked into a proprietary standard from a company potentially disappearing is a higly unlikely scenario, given the number of players involved and the openly available specifications.

## ROS built on OPC-UA

As is the case for ROS built on DDS, we ideally want to hide all OPC-UA-specific API's and message definitions while leveraging it to provide discovery, publish-subscribe and message serialization. A "vanilla", vendor-independent interface to the OPC-UA stack could then be complemented by vendor-specific additions to access special features of the stacks, as is foreseen for DDS.

### Discovery

The discovery mechanism of OPC-UA is implemented by Discovery Services described in parts 4 and 12 of the specifications. Please note that these parts have been updated from the last official 1.02 release and are now respectively at rel 1.03.09 and 1.03.52 (both dating March 2015). The specifications provide for:

- multiple Local Discovery Servers (LDSs) to be used for servers to register themselves and for clients to interrogate for server lookup. Please note that the terminology server-client in this context refers exclusively to the software components offering and using services and not to physical devices (*hosts*). It is in fact expected that most computational nodes in a network will run a number of instances of both servers and clients. Multiple LDSs can exist in a network, potentially with one running on each host;
- a mechanism provided for servers running on hosts without an LDS to announce themselves: this is expected to happen mostly on embedded devices running exactly one server due to limited resources;
- a Global Discovery Server (GDS) managing discovery services over multiple local subnetworks, i.e., in contexts where the host names can not be discovered. The GDS is also connected to Certificate management services related to the security aspect.

While this mechanism is apparently not as nice as the completely distributed system provided by DDS, something similar can in principle be overlaid on top of the self-advertising / LDS / GDS system, and one of the existing SDKs might already provide it.

**TODO**: inspect the concrete discovery services in the actual APIs of the existing SDKs.


### Publish-Subscribe Transport

A direct counterpart in OPC-UA to DDS-RTPS, on which thus a Pub/Sub protocol could be overlaid, is the MonitoredItems/Notification mechanism: OPC-UA clients can monitor (specific) items in the servers and be notified when a change occurs. This mechanism (much richer, as it includes alarms, etc) is described in Parts 1 and especially 3 of the specifications. However, since this [recent article](https://www.prosysopc.com/blog/opc-day-europe-2015/) mentions a new PubSub protocol draft for the specification, further enquiries were made over private communication channels. A member of the foundation confirmed that the current mechanism is suitable for pub/sub but that it has quite some margin for performance improvement when handling one-to-many subscriptions (i.e., multiple clients subscribing to the same information) and cases where fast even though potentially unreliable communication is to be preferred. These are the use cases that the new upcoming PubSub mechanism addresses.

### Efficient Transport Alternatives

Something like a shared-memory transport is not mandated by the standard as far is it is possible to gather from the specifications. However, this is not explicitly prevented either, so it might be possible that some vendor or that one of the open source implementations provide for it. As concluded for the DDS case, efficient intraprocess communication would need to be custom developed anyway.

**TODO**: check the available transport mechanisms for current implementations.

### Messages

The design goal, as stated in the original article, is to keep ROS-1.x message structure/definition. With DDS .msg files would need to be converted into .idl ones in order to use it. Such article concludes that the ROS 2.0 API would work with .msg files and convert .msg style message objects to .idl ones before publishing (the performance was gauged as totally acceptable, as the serialization cost would be at least 10x higher anyway).

As anticipated earlier in this document, OPC-UA has provisions for transport, data and information modeling. Whereas the transport part is somehow self-describing and mostly related to discovery and communication protocols, it is more difficult to draw a parallel 
as straightforwardly between ROS/DDS messaging (which can to a certain extent be assimilated, barring the .msg vs .idl definition format and other details) and OPC-UA's one. We can to a certain extent associate the "data" part, together with the so called "address space modeling", with a definition of the exposed data somehow similar to message definition. The information modeling specifications add to that ways to enforce semantics, create relantionships between the data and its possible providers, etc. In OPC-UA parlance, this equals to overlaying InformationModels on top of AddressSpaces (~ semantics and further information on top of messaging structures).

**TODO**: investigate the subject more and provide a better summary for this section.

### Services and Actions

OPC-UA prescribes Services as one of the fundamental interaction mechanisms, and they are apparently a direct counterpart to ROS services (i.e., a Request/Response comm mechanism).

**TODO**: check whether a "native" OPC-UA construct exists for Actions.

### Language Support

A list of available implementations has been provided earlier in the document, and even restricting ourselves to open source implementations there would be C, C++, Javascript, Python & Java bindings available. Their quality is to be verified, but even if just a single trusted implementation exposing a C API was (or could be made) available, it would be then feasible to implement other language bindings on top of it (as the article on ROS on DDS concludes as well).

### DDS as a Dependency

**TODO**:

- check what dependencies the OPC-UA implementations which are available as source need in order to be built on win/mac/linux
- check how big the binary to be bundled with the apps is

## The ROS on OPC-UA prototype

To be built if this effort is deemed to be worth pursuing.

## Conclusion

In progress.

### Further TODOs

- check differences between OPC-UA and its realtime ("TTS") variant
- alarm, events and historical data mngmt: how ROS could leverage it?


## References

1. OPC Unified Architecure - W. Mahnke, S-H. Leitner, M. Damm ([Springer](http://www.springer.com/us/book/9783540688983))
2. OPC-UA Specification - OPC Foundation (available [online](https://opcfoundation.org/developer-tools/specifications-unified-architecture), free registration required)

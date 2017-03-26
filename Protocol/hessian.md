Hessian and Burlap are compact binary and XML protocols for applications needing performance without protocol complexity. Hessian is a small binary protocol. Burlap is a matching XML protocol. Providing a web service is as simple as creating a servlet. Using a service is as simple as a JDK Proxy interface.

Hessian

Hessian is a simple binary protocol for connecting web services. The com.caucho.hessian.client and com.caucho.hessian.server packages do not require any other Resin classes, so can be used in smaller clients, like applets.

Because Hessian is a small protocol, J2ME devices like cell-phones can use it to connect to Resin servers. Because it's powerful, it can be used for EJB services.

The Hessian specification itself is a short and interesting description.

Flash

This document describes the ActionScript implementation of Hessian. Usage instructions, technical reference, and MXML instructions are given.

Hessian 1.0 spec

Hessian is a compact binary protocol for connecting web services.

Because Hessian is a small protocol, J2ME devices like cell-phones can use it to connect to Resin servers. Because it's powerful, it can be used for EJB services.

Hessian 2.0 serialization

Hessian 2.0 protocol

Java Binding

Burlap

Burlap is a simple XML-based protocol for connecting web services. The com.caucho.burlap.client and com.caucho.burlap.server packages do not require any other Resin classes, so can be used in smaller clients, like applets.

Because Burlap is a small protocol, J2ME devices like cell-phones can use it to connect to Resin servers. Because it's powerful, it can be used for EJB services.

Burlap 1.0 Spec

Burlap Design Notes

As described in the Burlap 1.0 spec, we created Burlap to implement Enterprise Java Beans (EJB) using an XML-based protocol with reasonable performance. Although many RPC protocols already exist, including several based on XML, none met our application's needs. The name "Burlap" was chosed for a simple reason: it's boring. Unlike the exciting protocols defining "Internet 3.0", SOAP and XML-RPC, Burlap is just boring text-based protocol to make testing and debugging EJB a little bit easier.

Hessian Messaging

The Hessian binary web service protocol can provide a messaging service layered on top of its RPC call. The messaging service itself is based on the standard Hessian RPC call, so Hessian itself has no need to become more complicated.

Debugging

Hessian provides debugging tools to view the wire protocol or serialized Hessian data.

Protocol Taxonomy

Choosing a metaprotocol framework is a major architectural decision, affecting performance, reliability, maintainability, and development effort. For example, Hessian may be a reliable and clean fit for a Streaming or RPC application, while SOAP may be required to match an existing WSDL specification. This article examines the common metaprotocols, categorizes them into a taxonomy, and provides a framework for choosing the appropriate metaprotocol for a given application.
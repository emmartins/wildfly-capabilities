WildFly Capabilities
====================
http://wildfly.org

This project provides the central registry of descriptive information about capabilities accessible via the management layer of a WildFly Core based process. The intent is to provide a central location where capability developers can go to learn about other capabilities available in the WildFly ecosystem and to advertise their capability. Must importantly, registering capabilities helps ensure that different capabilities won't use the same name.

Building
------------------

This project does not produce a build. It is simply a registry of information.

Structure of the Registry
-------------------------

The registry consists of two primary elements:

* a single text file in the root of the repository, 'registry.txt' which is a simple alphabetically ordered list of capability names. This file provides a single location where users can quickly learn the names of available capabilities
* a filesystem tree where the tree structure corresponds to the names of the capabilities. Each leaf in the tree is a 'capability.txt' file which includes standard information describing a particular capability. The 'template.txt' file in the root of the repository shows the standard format of a capability.txt file.

Capabilities
-------------------

A capability is a piece of functionality used in a WildFly Core based process that is exposed via the WildFly Core management layer. Capabilities may depend on other capabilities, and this interaction between capabilities is mediated by the WildFly Core management layer.

Some capabilities are automatically part of a WildFly Core based process, but in most cases the configuration provided by the end user (i.e. in standalone.xml, domain.xml and host.xml) determines what capabilities are present at runtime. It is the responsibility of the handlers for management operations to register capabilities and to register any requirements those capabilities may have for the presence of other capabilities. 

* Capabilities vs modules: A JBoss Modules module is the means of making resources available to the classloading system of a WildFly Core based process. To make a capability available, you must package its resources in one or more modules and make them available to the classloading system. But a module is not a capability in and of itself, and simply copying a module to a WildFly installation does not mean a capability is available. Modules can include resources completely unrelated to management capabilities.
* Capabilities vs Extensions: An extension is the means by which the WildFly Core management layer is made aware of manageable functionality that is not part of the WildFly Core kernel. The extension registers with the kernel new management resource types and handlers for operations on those resources. One of the things a handler can do is register or unregister a capability and its requirements. An extension may register a single capability, multiple capabilities, or possibly none at all. Further, not all capabilities are registered by extensions; the WildFly Core kernel itself may register a number of different capabilities.

Capability Names
-------------------

Capability names are simple strings, with the dot character serving as a separator to allow namespacing.

The 'org.wildfly' namespace is reserved for projects associated with the WildFly organization on github (https://github.com/wildfly)

The structure of this repository is based on the structure of capability names. The name is split based on the dot character and then each section becomes a directory in the repository. Each directory whose path from the repository root represents a complete capability name then includes a capability.txt file describing that capability.

Statically vs Dynamically Named Capabilities
--------------------------------------------

The full name of a capability is either statically known, or it may include a statically known base element and then a dynamic element. The dynamic part of the name is determined at runtime based on the address of the management resource that registers the capability. For example, the management resource at the address '/socket-binding-group=standard-sockets/socket-binding=web' will register a dynamically named capability named 'org.wildlfy.network.socket-binding.web'. The 'org.wildlfy.network.socket-binding' portion is the static part of the name.

All dynamically named capabilities that have the same static portion of their name should provide a consistent feature set and set of requirements. The entry in this registry should be for the static part of the capability name. So, for the socket-binding example above, the registry.txt file would include a line for 'org.wildlfy.network.socket-binding' and the repository would include a capability.txt file at location org/wildfly/network/socket-binding/capability.txt.

Services provided by a capability
---------------------------------

Typically a capability functions by registering services with the WildFly process' MSC ServiceContainer, and then dependent capabilities depend on those services. The WildFly Core management layer orchestrates registration of those services and service dependencies by providing a means to discover service names.

The capability.txt entry for a capability should provide information about services made available by the capability, if there are any.

Custom integration APIs provided by a capability
------------------------------------------------

Instead of or in addition to providing MSC services, a capability may expose some other API to dependent capabilities. This API must be encapsulated in a single class (although that class can use other non-JRE classes as method parameters or return types).

The capability.txt entry for a capability should provide information about the custom integration API made available by the capability, if there is one.

Capability Requirements
------------------------

A capability may rely on other capabilities in order to provide its functionality at runtime. The management operation handlers that register capabilities are also required to register their requirements.

There are three basic types of requirements a capability may have:

* Hard requirements. The required capability must always be present for the dependent capability to function.
* Optional requirements. Some aspect of the configuration of the dependent capability controls whether the depended on capability is actually necessary. So the requirement cannot be known until the running configuration is analyzed.
* Runtime-only requirements. The dependent capability will check for the presence of the depended upon capability at runtime, and if present it will utilize it, but if it is not present it will function properly without the capability. There is nothing in the dependent capability's configuration that controls whether the depended on capability must be present. Only capabilities that declare themselves as being suitable for use as a runtime-only requirement should be depended upon in this manner.

Hard and optional requirements may be for either statically named or dynamically named capabilities. Runtime-only requirements can only be for statically named capabilities, as such a requirement cannot be specified via configuration, and without configuration the dynamic part of the required capability name is unknown.

The capability.txt entry for a capability should provide information about capability requirements, if there are any.

Supporting runtime-only requirements
------------------------------------

Not all capabilities are usable as a runtime-only requirement, and the capability.txt entry for a capability should express this.

Any dynamically named capability is not usable as a runtime-only requirement.

A capability that supports runtime-only usage must ensure that it never removes its runtime services except via a full process reload.


Contents of capability.txt
--------------------------

The following is information describing the contents of a capability.txt file. See the template.txt file in the repository root for context.

NAME: The name of the capability. See above.

DESCRIPTION: A free form description of the capability

REGISTERED BY: Name of the project that is responsible for the integration contract exposed by the capability. There may be multiple projects that provide an implementation the capability, but there should be a single project responsible for the capability's contract. The contract is defined by the capability.txt file and by the Java API of any classes referenced in the "SERVICES PROVIDED" and "CUSTOM INTEGRATION API" sections below. Preferably this section will include a URL for the project.

DYNAMIC: True if the capability is dynamically named; false otherwise.

SUPPORTS RUNTIME ONLY: True if the capability can be depended upon by another capability as a runtime-only requirement. Must be 'false' if DYNAMIC is 'true'

SERVICES PROVIDED: Information about MSC services registered by the capability which can be depended upon or injected by other capabilities, if there are any. For each such service the following should be provided: 

* CLASS: The fully qualified class name of the value type of the MSC Service
* MODULE: The name of the jboss-modules module that typically provides the class

CUSTOM INTEGRATION API: Information about the custom integration API provided by the capability, if there is one.

* CLASS: The fully qualified class name of class that provides a custom API for interacting with the capability
* MODULE: The name of the jboss-modules module that typically provides the class

HARD REQUIREMENTS: Information about other capabilities that must be present if this capability is registered, if there are any. For each such capability the following should be provided: 

* NAME: The static portion of the name of the required capability
* DYNAMIC: True if the required capability is dynamically named

OPTIONAL REQUIREMENTS: Information about other capabilities that must be present if some aspect of this capability's configuration is present, if there are any. For each such capability the following should be provided: 

* NAME: The static portion of the name of the required capability
* DYNAMIC: True if the required capability is dynamically named
* DESCRIPTION: Free form text description of what makes this requirement optional

RUNTIME ONLY REQUIREMENTS: Information about other capabilities that cannot be required by this capability's configuration, but which will be detected and used if present.

* NAME: The static portion of the name of the required capability
* DESCRIPTION: Free form text description of what functionality is enabled if this capability is present
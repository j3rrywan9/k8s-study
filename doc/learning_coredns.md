# Learning CoreDNS

## Preface

### Why a New DNS Server?

To begin with, CoreDNS is written in Go, and Go is a memory-safe programming language.
Why is that important?
 Well, if you've ever run a BIND-based DNS infrastructure and had to upgrade 100 DNS servers ASAP because of a buffer overrun, you know.
A healthy proportion of vulnerabilities in DNS servers of all stripes (at least those written in C and C++) stem from buffer overflows or overruns and dangling pointers.
Written in memory-safe Go, CoreDNS isn't subject to these.

Programs written in Go can also support concurrency, or parallel execution.
This can be useful in wringing more performance out of multiprocessing or multitasking systems.
BIND's performance somewhat notoriously doesn't scale well on multiprocessor systems, whereas CoreDNS's performance scales nicely the more processors it has to work with.

Improving performance can be important because Go tends to run somewhat more slowly than C or C++, partly thanks to the overhead imposed by its many features.
In most cases, however, this isn't an issue: What's important is that CoreDNS performs well enough to handle the workload you offer it, and in the vast majority of cases, it does, Go or no Go.

Probably the most significant capability CoreDNS offers, though, is its ability to communicate with container infrastructure and orchestration systems such as etcd and Kubernetes.

### Who Needs CoreDNS?

The short answer: basically anyone running Kubernetes, and most folks running containerized applications.

The function CoreDNS fulfills in a containerized environment is that of a service directory, which we talk about in detail in this book.
A service directory helps containers determine the IP address or IP addresses where the containers that offer a particular service are running.
For example, a container might look up a domain name that represents the database service for a specified application in order to retrieve some data.
The service directory function is critical because, in the world of containers and microservices, applications are usually decomposed into many small services (hence, "microservices"!), and each service might be offered by several containers, each running at a different IP address.

But CoreDNS's utility isn't limited to containerized environments.
CoreDNS's plug-ins support advanced DNS functionality that even the big boys like BIND don't support.
You can rewrite queries and responses on the fly, for example.
You can automatically load zone data from GitHub or Amazon Route 53.
And because CoreDNS itself is small and usually runs in a container, it's suitable for use in scenarios in which a big DNS server such as BIND would not be.

## Chapter 1. Introduction

### What Is CoreDNS?

CoreDNS is DNS server software that's often used to support the service discovery function in containerized environments, particularly those managed by Kubernetes.

Compared to the syntax of, say, BIND's configuration file, CoreDNS's *Corefile*, as it's called, is refreshingly simple.
The Corefile for a basic CoreDNS-based DNS server is often just a few lines long and - relatively speaking - easy to read.

CoreDNS uses plug-ins to provide DNS functionality.

Plug-ins are also fairly easy to develop.
That's important for two reasons.
First, if you want to extend CoreDNS's functionality, you can write your own plug-in; we cover that in Chapter 9.
Second, because writing new plug-ins isn't rocket science, many have been developed, and more are being written all the time.
You might find one that provides functionality you need.

Probably the most significant advantage CoreDNS offers, though, is its ability to communicate with container infrastructure and orchestration systems such as etcd and Kubernetes.
We discuss this in much more detail later in the book, but let's take a quick look at this functionality here.

#### CoreDNS, Containers, and Microservices

In such an environment, though, tracking where a particular service is running can be challenging.
Say a container supporting the database service needs to communicate with the authorization service to determine whether a given user should be allowed to conduct a particular search.
If the containers that implement the authorization service are being started and stopped dynamically to accommodate load, how do we get a list of all running authorization containers?

The answer is most often DNS, the Domain Name System.
Since the communications between containers is almost always based on IP, the Internet Protocol, and because developers have been using DNS to find the IP addresses of resources for literally decades, using DNS to identify containers that offer a given service is natural.

It's in this capacity that CoreDNS really shines.
Not only is CoreDNS a flexible, secure DNS server, but it integrates directly with many container orchestration systems, including Kubernetes.
This means that it's easy for the administrators of containerized applications to set up a DNS server to mediate and facilitate communications between containers.

#### CoreDNS Limitations

CoreDNS does currently have some significant limitations, though, and it won't be suitable for every conceivable DNS server.
Chief among these is that CoreDNS, at least in the latest version as of this writing, doesn't support full recursion.
In other words, CoreDNS can't process a query by starting at the root of a DNS namespace, querying a root DNS server and following referrals until it gets an answer from one of the authoritative DNS servers.

#### CoreDNS, Kubernetes, and the Cloud Native Computing Foundation

CoreDNS was submitted to the CNCF in 2017 and moved to "graduated" status in January 2019.
As testament to CoreDNS's criticality to Kubernetes environments, CoreDNS became the default DNS server shipped with Kubernetes with Kubernetes version 1.13, which was released in December 2018.
Given that CoreDNS is now installed with almost every new Kubernetes implementation, and Kubernetes is a juggernaut in the world of containers (and containers themselves seem to be taking the world by storm), we expect the installed base of CoreDNS to explode.

Enough of singing CoreDNS's praises. We've talked about what CoreDNS is good for and what it isn't, and how it's had its fate lashed to Kubernetes.
Next, we give you a whirlwind refresher on DNS theory so that we can begin talking about how to configure CoreDNS to do useful work!

## Chapter 2. A DNS Refresher

Our compromise is to try to give you just enough DNS theory to get by, and then to point you in the direction of, for example, DNS and BIND if you're interested in more detail.
(Hopefully that doesn't seem too self-serving.)

### What Is the Domain Name System?

The DNS is a naming system that maps names to other data, such as IP addresses, mail routing information, and more.
And DNS isn't just any naming system: it's the internet's standard naming system as well as one of the largest distributed databases in the world.

DNS is also a client–server system, with DNS clients querying DNS servers to retrieve data stored in that distributed database.
Because the database is distributed, DNS servers will often need to query one or more other DNS servers to find a given piece of data.
DNS clients are often called *resolvers*, whereas DNS servers are sometimes called *name servers*.
Resolvers ask DNS servers for information about particular indexes into the distributed database.

### Domain Names and the Namespace

Those indexes into DNS's distributed database are called *domain names*.
These are the dotted names that should be familiar to you from internet email addresses and URLs.

These domain names actually represent nodes in DNS's *namespace*.
DNS's namespace is an inverted tree, with the root node at the top.
Each node can have an arbitrarily large number of child nodes, and is usually depicted with links between it and its children.
Each node also has a label, which can be up to 63 ASCII characters long.
The root node has a special label: the null label, which has zero length.
Only the root node has the null label.
Beyond that, there aren't many restrictions on labels - mainly that the child nodes of a single node must all have different labels.
That makes sense: It helps avoid ambiguity and confusion, just as giving your children unique first names does.

Clearly a label is useful only in distinguishing one node from its siblings; some other identifier is needed to identify a particular node in the entire namespace.
That identifier is the domain name.

A node's domain name is the list of labels on the path from that node upward to the root of the namespace, with a single dot separating each label from the next.

### Domains, Delegation, and Zones

There are a few other bits of theory we need to introduce before diving into the world of how DNS servers work, so please bear with us.
The first is a *domain*.
A domain is a group of nodes in a particular subtree of the namespace; that is, at or below a particular node.
The domain is identified by the node at its apex (the topmost node in the domain): it has the same domain name.

We'll leave aside for the time being the mechanics of how delegation is done.
For now, suffice it to say that the *berkeley.edu* domain now contains information on where people can find information in the *cs.berkeley.edu* subdomain, rather than containing that information itself.

Thanks to delegation, the IT folks at Berkeley no longer control nodes at or below *cs.berkeley.edu*; those belong to the CS department.
What do we call the set of nodes at or below *berkeley.edu* that the IT folks still control?
That's the *berkeley.edu zone*.
A zone is a domain minus the subdomains that have been delegated elsewhere.
What if there's no delegation within a domain?
In that case, the domain and the zone contain the same nodes.
For example, if there's no further delegation below *cs.berkeley.edu*, the domain *cs.berkeley.edu* and the zone *cs.berkeley.edu* are effectively the same.

There are zones *above berkeley.edu*, too, of course.
The *edu* domain is run by a nonprofit association called EDUCAUSE, which delegates *berkeley.edu* and *umich.edu* and many other subdomains to educational institutions around the world.
What they're left with — what they directly manage — is the *edu* zone.

### Resource Records

If, as we said, DNS is a distributed database, where's all the data?
So far, we have indexes (domain names) and partitions of the database (zones), but no actual data.

Data in DNS is stored in units of *resource records*.
Resource records come in different classes and types.
The classes were intended to allow DNS to function as the naming service for different kinds of networks, but in practice DNS is used only on the internet and TCP/IP networks, so just one class, "IN," for internet, is used.
The types of resource records in the IN class specify both the format and application of the data stored.
Here's a list of some of the most common resource record types in the IN class:

### DNS Servers and Authority

DNS servers have two chief responsibilities: answering queries about domain names, and querying other DNS servers about domain names.
Let's begin with the first responsibility: answering queries.

DNS servers can load zone data from files called, appropriately enough, *zone data files* or, equivalently, *master files*.
Each zone data file contains a complete description of a zone: all of the records attached to all of the domain names in the zone.
A DNS server that loads information about a zone from a zone data file is called a *primary DNS server* for that zone.

DNS servers can also load zone data from other DNS servers via a mechanism called a *zone transfer*.
A DNS server that loads information about a zone from another DNS server using zone transfer is said to be a secondary DNS server for that zone.
The DNS server from which the secondary DNS server transfers the zone is referred to as its *master DNS server*.
After transferring the zone, the secondary DNS server might save a copy of the zone data to disk, sometimes in what's called a *backup zone data file*.
When the secondary periodically transfers a new version of the zone from its master DNS server, it updates the data on disk.
The backup data is useful if the secondary DNS server should restart because it can initially load the backup data, then check to see whether that data is still up to date with the version of the zone on the master DNS server.
If it is, no zone transfer is necessary.
And if the master DNS server is unavailable, the secondary DNS server still has zone data it can answer with.

Both the primary and secondary DNS servers for a zone are said to be *authoritative* for the zone.
This means that they can answer any query for a domain name in the zone definitively.
(Other DNS servers, you'll see, might have cached answers to queries, which might or might not still be current.)

A single DNS server can be authoritative for many zones at the same time and can be primary for some and secondary for others.
Internet service providers and DNS hosting companies often run DNS servers that are authoritative for hundreds of thousands of zones.

That's enough about DNS servers for now.
Let's move on to resolvers, the other main software component of the Domain Name System.

### Resolvers

Resolvers take applications' requests for information about a domain name and translate them into DNS queries.
They then send those queries to DNS servers and await responses.
If the resolver doesn't receive a response to a given query within a reasonable amount of time (typically a second or a few seconds at most), it might retransmit the query to the same DNS server, or it might try querying a different DNS server.
When it receives a response, the resolver unpacks it into a data structure that it passes back to the application.
Some resolvers do even more, including caching recently returned answers.

### Resolution and Recursion

*Resolution* is the process by which resolvers and DNS servers cooperate to find answers (in the form of resource records) stored in DNS's distributed database.
Sometimes resolution is simple: A resolver sends a query to a DNS server on behalf of an application, and the DNS server is authoritative for the zone that contains the domain name in the query, so it responds directly to the resolver with the records that make up the answer.
However, for cases in which the DNS server isn't authoritative for the zone that contains the answer, the resolution process is more complicated.

The process that the first DNS server follows - starting with the root DNS servers and following referrals until it receives an answer - is called *recursion*.

### Caching

In practice, most DNS servers processing recursive queries don’t need to query the root DNS servers very often.
That's because they cache the resource records in responses.

## Chapter 3. Configuring CoreDNS

CoreDNS is configured using a configuration file called the *Corefile*.
The syntax of the Corefile follows that of the *Caddyfile*, given that CoreDNS actually uses the Caddy code to parse the configuration.
First, though, we need to get CoreDNS set up.

### Getting CoreDNS

### CoreDNS Command-Line Options

### Corefile Syntax

Before writing our first *Corefile*, we need to go over its syntax and structure.
*Corefiles* consist of one or more *entries*, which themselves comprise *labels* and *definitions*.

#### Environment Variables

#### Reusable Snippets

#### Import

#### Server Blocks

For CoreDNS, the most common entry is called a *server block*.
A server block defines a server within CoreDNS, a configuration that determines how queries for particular domain names, received on particular ports and over particular protocols, are processed.
In its simplest form, the server block's label is just the domain name of a domain that matches some set of queries, as shown in Example 3-10.
```
foo.example {
    # directives go here
}
```

#### Query Processing

### Plug-ins

What are the configuration directives that control how CoreDNS responds to queries?
In CoreDNS, they're *plug-ins*, software modules that implement DNS functionality.

### Common Configuration Options

### Sample DNS Server Configurations

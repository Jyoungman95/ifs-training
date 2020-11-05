# Networking Model

- TL,DR:

  *Our cluster (nodes and pods) is one big flat IP network.*

--

- In detail:

 - all nodes must be able to reach each other, without NAT

 - all pods must be able to reach each other, without NAT

 - pods and nodes must be able to reach each other, without NAT

 - each pod is aware of its IP address (no NAT)

 - pod IP addresses are assigned by the network implementation

- Kubernetes doesn't mandate any particular implementation

---

## Networking Fundamentals

--

- Layer 2 Networking: Node to Node

--

- Layer 7 Networking: Ingress Controllers

--

- NAT — Network Address Translation: Remapping IPs into another

--

- Network Namespace: Logical copy of the entire network stack; routes, fw rules, and network devices

--

- veth — Virtual Ethernet Device Pairs: Tunnles between Namespaces

--

- bridge — Network Bridge: Connecting two segments as if they were one

--

- CNI — Container Network Interface: A CNCF project consisting of specification and libraries for writing plugins

--

- VIP — Virtual IP Address: Software defined IP that does not correspond to an actual phyrical network interface

---

## Networking Fundamentals

--

- netfilter — The Packet Filtering Framework for Linux

--

- iptables — Packet Mangling Tool

---

## Kubernetes network model: the good

- Everything can reach everything

- No address translation

- No port translation

---

## Kubernetes network model: the less good

- Everything can reach everything

  - if you want security, you need to add network policies

  - the network implementation that you use needs to support them

- There are literally dozens of implementations out there

---

## The Container Network Interface (CNI)

- Most Kubernetes clusters use CNI "plugins" to implement networking

- When a pod is created, Kubernetes delegates the network setup to these plugins

- Typically, CNI plugins will:

  - allocate an IP address

  - add a network interface into the pod's network namespace


---

## Characteristics

- The "pod-to-pod network" or "pod network":

  - provides communication between pods and nodes

  - is generally implemented with CNI plugins

- The "pod-to-service network":

  - provides internal communication and load balancing

  - is generally implemented with kube-proxy

- Network policies:

  - provide firewalling and isolation

---

## Requirements

 - all nodes must be able to reach each other

 - all pods must be able to reach each other

 - pods and nodes must be able to reach each other

 - each pod is aware of its IP address

---

class: pic

![Container to Container](images/networkcontainer.png)

---


class: pic

![Pod to Pod](images/networkpod.png)

---


class: pic

![Pod to Pod Same Node](images/networkpodtopodsame.gif)

---


class: pic

![Pod to Pod Different Node](images/networkpodtopodsdifferent.gif)

---


class: pic

![Pod to Service](images/networkpodtoservice.gif)

---


class: pic

![Service to Pod](images/networkpodtoservice2.gif)

---


class: pic

![Pod to Internet](images/networkpodtozinternet.gif)

---

class: pic

![Internet to Service](images/networkzinternettoservice.gif)

---
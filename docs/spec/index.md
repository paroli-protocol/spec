# Overview

## Core
- [paroli-core-dag](core/paroli-core-dag)
- [paroli-core-room](core/paroli-core-room)
- [paroli-core-session](core/paroli-core-session)
- [paroli-core-sync](core/paroli-core-sync)

## Extensions
- [paroli-ext-acl](ext/paroli-ext-acl)
- [paroli-ext-chat](ext/paroli-ext-chat)
- [paroli-ext-identity](ext/paroli-ext-identity)

# Evil monologues

Sometimes, names aren't just enough to understand the function of each protocol. Thus, below is a humorous set of evil villain monologues to help you better understand the function of each:

- **paroli-core-dag**: I am the evil storage layer! I do not know what a room is. I do not know what an event is. I do not know what YOU are. I only know Merkle DAGs, and I will manage those trees with an iron fist. Do not ask me anything else. Muahahah.
- **paroli-core-room**: I am the local representation of a room! I am blissfully ignorant of the outside world. The network? Never heard of it. The internet? Could be a woman, for all I know. I only know events, trees, and the architecture and local state of a room, which I expose through an API. Muahahah.
- **paroli-core-session**: I am an Ed25519 key pair. That is all. Muahahah.
- **paroli-core-sync**: I speak networks! While I don't define a specific channel (LAN, WAN, TCP, UDP), I will evil-ly venture into the outer network while my villain colleagues sit comfortably in their local databases, meanwhile I gossip with strangers. I do not understand what I am syncing. I just sync and distribute events with the help of `paroli-core-room`. Muahahah.
- **paroli-core-node**: I am the mastermind! While my fellow villains have very specialized evil tasks, I am the one who coordinates them all in order to make a functional node in the network. They work on what they do best, I'm the one who sees the bigger picture and coordinates! Muahahah.
- **paroli-ext-acl**: I am the evil bouncer! I act as another layer in `paroli-core-room`'s event validation chain, defining a standardized role-based access control system. I do not know what a room is, I just allow or disallow actions. Muahahah.
- **paroli-ext-chat**: I am just the definition of a timeline tree and a few sets of events related to chat. Muahahah.
- **paroli-ext-identity**: I evil-ly work along with `paroli-core-session`, and by reusing `paroli-core-room` as a primitive I evil-ly link your sessions together to cryptographically prove that they belong to the same identity, along with evil key management and recovery mechanisms. Muahahah.

# Java WalletConnect Bridge

This is a simple Java implementation of a WalletConnect bridge server. If you don't know what WalletConnect is, see walletconnect.org for more information.

## Motivation

My motiviation for writing this implementation is that the official server (wss://bridge.walletconnect.org) does not seem consistently reliable. Messages are often only dispatched with a considerable delay and sometimes disappear entirely. While part of that can probably be attributed to a high load, part of that is probably also attributable to some flaws in the implementation. So I decided to create our own implementation in the language I'm most fluent in.

## Running the Bridge

To run the bridge, clone this git repository to your server and make sure to have [Maven](https://maven.apache.org/) and [Java](https://openjdk.java.net/install/) installed.

You can then compile the project with 'mvn package' and run it with 'java -cp target/bridge.jar:target/dependency/* com.aktionariat.bridge.BridgeServer'. (Windows users should replace the colon ':' with a semicolon ';' as Windows has a different path separator.)

This spawns a local bridge server on port 8887.

Arguments:

- -port: the port number, defaults to 8887
- -cert: path to the keystore file, enables wss
- -passphrase: the passphrase for the keystore file

(Since certificate handling is not so convenient in Java, it is probably easier to run [Caddyserver](http://caddyserver.com/) or similar in front of the bridge instead of configuring ssl on the bridge server directly.)

## Protocol

The WalletConnect bridge is functionally very simple. All it does is forwarding messages. While not officially documented, the protocol knows three types of messages: sub, pub, and ack. Each message has a 'topic' attached, so the server knows who it is intended for. Anyone can send a 'sub' message to the bridge to subscribe to the given topic. When doing so, all pending messages are sent to the subscriber. When sending a 'pub' message to the server, it is forwarded to the subscribers of the according topic. After having received such a message, most clients seem to answer with an 'ack' message.

## Differences to the Reference Implementation

There are a number of differences to the reference implementation, which I'd like to point out:

* The reference implementation keeps a list of subscribers per topic, our implementation only remembers the latest one. This is a valid simplification under the assumption that the bridge is intended to bridge two end-points with each other and is not used for broadcasting. It also eliminates a subtle problem in the reference implementation: when the reference implementation receives a pub message and finds some connections, it immediately sends the message to these connections and then forgets the message. This poses a problem in case some of these connections are stale and the client (maybe a mobile device with an unreliable connection) has to reconnect to fetch the message the user expects. Unfortunately, the message will never be received in that case as the server does not have it any more.
* The reference implementation ignores the 'ack' messages, we use it to clear the message queue.
* After sending one or more 'put' messages to a subscriber, we add a 'ping' message. The message queue is only cleared once the subscriber answers with the according 'pong' message. This ensures that messages are actually received and not sent to stale connections.
* The reference implementation keeps undelivered messages indefinitely in a redis in-memory database, we drop them after 30 minutes. We do this to keep the memory footprint small. This is ok under the assumption that the use case at hand is about immediate interaction between a website and a wallet, and not for slow, email-like interactions.
* We do not support the push server, ignoring the "silent" flag.
* No support for the 'hello', 'status' and similar methods. But the websocket standard 'ping' is there.

For the future, we are contemplating the addition of sequence numbers in order to eliminate the unlikely possibility of a delayed ack leading to the deletion of an undelivered message.

## Aktionariat Instance

We are running an instance of this server at address 'wss://bridge.aktionariat.com:8887'. Feel free to use it, but do not rely on its availability.


## Copyright and License

Copyright (c) 2019 Pharos Production Inc.

Created by [Dmytro Nasyrov, Founder, CTO at Pharos Production Inc.](https://www.linkedin.com/in/dmytronasyrov/)

[Pharos Production Inc. - Web3, blockchain, fintech, defi software development services](https://pharosproduction.com)

The source code is under the Apache 2 License.

Copyright (c) 2025 Pharos Production Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

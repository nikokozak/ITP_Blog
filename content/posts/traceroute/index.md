+++
date = '2025-09-11T23:05:59-04:00'
draft = false
title = 'Traceroute'
tags = ['Understanding Networks']
+++

Traceroute allows us to see the path a packet takes from our computer to the requested host server. Inspecting a traceroute can give us powerful clues as to the structure of the network that surrounds us, and our location within it.

I chose to use **mkr** throughout these explorations, namely because it's fun to watch (it dynamically updates latency values as it attempts, a default of 10 times, to ping a given server on the path to our target). It also has some nice affordances around saving data to json files, as well as reading from a `hosts` text file (making it easy to list out a number of addresses we might want to try without doing it manually).

## Finding ^high^ hops!

Probably the first thought to entertain is "I wonder what the largest number of hops I can get is...". 

Turns out, (at least for me), the winners are the **Rwanda and Vanuatu** government websites, with 24 hops each. The logic here is that these are locally served sites (no large CDNs like Cloudflare) in fairly "remote" places, requiring complex navigation to get to. That said, it's still *insane* that we can get pretty much anywhere on earth in just 20 hops.

Here's a pair of maps, the first for the Rwandan site and the second for the Vanuatu site:

![image](rwanda_tracroute.png)
![image](vanuatu_traceroute.png)

Other sites of note are:

`egov.sc` - 16 hops - Seychelles
`presidence.gov.bi` - 16 hops - Burundi
`github.com` - 13 hops w/ tcp 443 - Github!
`reuters.com` - 13 hops w/ tcp 443 - Reuters!
`emol.com` - 12 hops - Chilean news site I read in the mornings
`kiribati.gov.ki` - 10 hops - Kiribati

You'll notice that some sites, like `Github`, display oddly large numbers of hops using a `tcp 443` protocol. The logic for this, apparently, is that servers will often ignore or de-prioritize ICMP requests, essentially making hops "invisible", thus resulting in artificially low numbers. 443 is the standard HTTPS port, and thus it is more likely that servers will respond in kind - often these packets will be routed differently because of this.

Also of note is that we can usually identify hops in-and-out of AS's by sharp increases in latency from one hop measure to another.

In making a traceroute to `emol.com`, located in Chile, the hop from `Level 3 Parent, LLC` (our "gateway" AS) to `Cirion` (AS in Latam) takes about 105ms. What's interesting is that in the `emol.com` case, there's a jump from `Level 3` to `Cirion` and then *back* to `Level 3` before jumping to the final AS (`Telmex Chile`). I can only presume there's a bottleneck somewhere in Cirion, or Level 3 actually manages bridge-infrastructure between Cirion and Telmex, which is something I'll investigate further.

## Interesting things close to home

It seems like every request I make out of the NYU network has to pass through `Level 3 Parent, LLC` (AS3356). Presumably the NYU network is directly tied to `Level 3`, and as such acts as the entrypoint into the wider web.

Another interesting AS I found is the one belonging to `NYSERNET` (AS3629). This one popped up while making a `traceroute` for `google.com`. It turns out that NYSernet is a fairly old non-profit, which also was the first commercial ISP in the United States! They were also the first to set up a proper network outside of the DoD (or at least they claim). Still, very cool to find these rich histories hidden behind a terminal.

I found it curious that on the edge of the NYU network we always find domains like: `extgwa-dmzgwc.net.nyu.edu`. I looked around a bit, and it seems like the subdomain naming convention is fairly common, and refers to specific interfaces and how they're connected. In this case, `extgwa-dmzgwc` might refer to:

extgw[a|b|c] = external gateway [A/B/C]
dmzgw[a|b|c] = DMZ gateway [A/B/C]
dash between names = point‑to‑point link naming convention (routerA–routerB), sometimes with interface suffixes (e.g., eth11-1 ↔ eth7-1)

I asked an LLM for a bit of help defining what `external gateway` and `DMZ` might mean in this context; here's Claude's answer:

"External gateway (extgwX): campus edge router/firewall pair that peers with ISPs/CDNs (eBGP), does border ACL/NAT/policy, and originates your default route. It’s the handoff from NYU to external ASes.

DMZ gateway (dmzgwX): firewall/router facing the DMZ VLANs. It’s the L3 gateway for public-facing services (reverse proxies, mail relays, VPN portals, etc.), enforcing policies between campus/core and the DMZ. It is not “internal open”; it’s semi‑trusted and more tightly filtered from the Inside than the Outside is.

DMZ (demilitarized zone): segregated subnet(s) specifically for systems exposed to the Internet. Inbound from Internet to DMZ is allowed per policy; DMZ to Inside is very limited; Inside to DMZ is controlled."

## Implementing RAND Networks in Elixir/Erlang

I can't understate how *fascinating* I find the RAND Networks paper. I'm very surprised at how simple the fundamental pathfinding algorithm is - the combination of "hot potato" with the routing table tracking hops across nodes. It's incredibly simple yet manages to effortlessly encompass the surreal complexity of tracking and operatinga self-healing system.

I have a long working history with Elixir (a modern offshoot of Erlang). One of the beauties of Erlang is that it allows us to create thousands of lightweight, managed, concurrent processes that excell at message-passing (it was developed as a telephone-switching platform after all). It ocurred to me that this would make modelling the famous RAND "mesh" network described in the paper a fairly easy project, and I could get the pleasure of running my own traceroute against my little network. 

### Basic Structure

Every process we spawn in Elixir gives us a `pid` that we can use to call it. Because processes automatically implement mailboxes (queues) for messages, we don't need to worry about handling FIFO queues for each entity we create (truly, Elixir is probably the most "sane" language I've ever worked with). 

The RAND paper is quite clear about the structure of a Node:

```code
NODE -> PROCESSOR + INTERFACE
INTERFACE -> NETWORK_CARD | INTERFACE + NETWORK CARD
```

Where a `NETWORK_CARD` (my language) is nothing more than the actual hardware connector to another node. A `PROCESSOR` in this case is understood as the Node process that receives a `PACKET` and "hot-potatoes" it to another `NODE` based on the network table (or randomly if the other `NODE` is unknown).

Therefore, in order to simulate this structure in Elixir, we create the following mapping:

```code
NODE -> GenServer + INTERFACE
INTERFACE -> Agent | INTERFACE + Agent
```

We create a series of `GenServers` which will act as our `NODE`s, in that they will: a) own a series of "interfaces" (smaller, Agent processes) & b) hold state for routing. On creation, 4 network "cards" (`INTERFACES`) are created per `NODE`, and interconnected randomly (while disallowing repeat connections) - these are then assigned to each of the X number of `NODE`s we wish to create; in this way, we simulate the creation of a mesh network as specified in the RAND paper (a series of `NODES`, all interconnected, with no "central" nodes).

We can also create a rudimentary `PACKET` to simulate the `1024`-bit packet the paper suggests.

```elixir
    %Packet{
      bits: <<from_node_pid::256, to_node_pid::256, 0::16, message_size::16, message::binary>>
    }
```

At this point our basic structure is done.

### Node logic

Our `NODE`s hold the following logic:

```elixir
    state = %{
      name: self(),
      node_map: %{},
      # List of pids of HardwareLink processes
      interface_pids: network_interface_pids
    }
```

As we receive `PACKET`s from the `HARDWARE_LINK` Agents, we add these to our `node_map`, which is just a `Map` representation of the hop-table suggested in the RAND paper. Because Elixir allows *anything* to be a `Map` key, we can easily implement a table like so:

```elixir
node_map = %{{node_pid, local_hardware_pid} => hop_count, ...}
```

Then, when looking to route a `PACKET`, we can look up the equivalent pairing in our `node_map`. If the pairing doesn't exist, we "hot potato" the `PACKET` out of another `HARDWARE_LINK`.

The basic logic is encompassed in the following `handle_call` callback in our `NODE` GenServer:

```elixir
  @impl true
  def handle_call({:incoming_packet, packet}, from_local_interface_pid, state) do
    %{from_node: from_node_pid, to_node: to_node_pid, hops: hops, message: _message} =
      parsed_message = parse_packet(packet)

    state = update_node_map(state, from_node_pid, from_local_interface_pid, hops + 1)

    if is_this_node?(to_node_pid) do
      IO.puts("Message reached destination: #{inspect(parsed_message)}")
      # Mark as processed
      {:reply, :ok, :processed}
    else
      # Forward the packet to the fastest known route to the destination node
      # Otherwise send it out of a random link
      case fastest_route(state, to_node_pid) do
        {{_node, link_id}, _hops} ->
          if link_id != from_local_interface_pid do
            forward_packet(link_id, Packet.update_hops(packet))
          end

        nil ->
          # No known route to the destination node
          # Send it out of a random link
          Enum.random(
            state.interface_pids
            |> Enum.filter(fn ipid -> ipid != from_local_interface_pid end)
          )
          |> forward_packet(Packet.update_hops(packet))
      end

      {:reply, :ok, state}
    end
  end
```

To add a bit of realism, our `PACKET` is implemented as a module that encodes everything into a 1024 bit package:

```elixir
  def make_packet(from_node_pid, to_node_pid, message) do
    # For simplicity, we will just create a binary with fixed sizes for each field, with hops starting at 0
    message_size = byte_size(message)

    %Packet{
      bits: <<from_node_pid::256, to_node_pid::256, 0::16, message_size::16, message::binary>>
    }
  end
```

Of note is that nowadays `hops` is replaced by the concept of TTL, which counts down as it passes through routers - when it gets to 0, the packet is discarded to avoid forever loops.

### Basic Behavior
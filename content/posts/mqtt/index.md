+++
date = '2025-11-24T23:51:36-05:00'
draft = true
title = 'Mqtt'
tags = ['Understanding Networks']
+++

So, like a dumb-dumb, I forgot my ESP32 at home. And so, stuck here without any real way of implementing an MQTT flow, I've instead decided to look a bit more closely at the protocol itself, in the hopes of understanding why one would choose to use it over anything else (this is still not clear to me).

MQTT stands for Message Queue Telemetry Transport, and is a pub-sub, lightweight messaging protocol originally designed in the 90’s for linking oil pipeline sensors with satellites.

The conceptual model for MQTT is publication/subscription (known more commonly as pub-sub), which had a brief viral resurgence when microservices were all the rage, and large, monolith-type codebases were being broken down into things like containerized modules. It turns out that pub-sub, where clients post messages to a central service, which then redistributes said messages to subscribers of that particular message stream, is a great way of orchestrating communication amongst many nodes.

I find it helps to build a metaphor in order to really understand the value of a protocol for MQTT. Pub/sub, clients and brokers are all terms that can get a bit abstract. So - let’s imagine the following:

## The Town of Sedona

In the town of medieval Sedona live Albert and Mary, brother and sister. They run a small family business: Mary is the town’s doctor, and Albert is the town’s veterinarian. They both spend their days running around Sedona, heeding urgent calls for help. One day, a terrible plague strikes Sedona, one that affects both animals and humans. Albert and Mary suddenly have to become researchers, and have to share field notes as they go about their respective cases, in the hopes of finding patterns to the disease and eventually a cure.

The problem is that they need to share information frequently, but they’re both on the move. They first devise a plan to share their findings. Every day at noon, they’ll meet for an hour at the town square and compare notes. They quickly find they cannot spare the time - people and animals are calling out to them, and they can’t afford to meet and block out the world in order to discuss the plague and its symptoms.

So, they come up with another plan. Mary will leave her notes in the town square whenever she gets the chance, and Albert will pick them up whenever *he* gets a chance, leaving his notes in turn. This works for about a week, until someone starts stealing the notes! Mary and Albert often show up, only to find no documents waiting for them. Upon closer investigation, they realize that Tina has been taking the notes in the hopes of figuring out how many beds to prepare at the local hospital. They realize they need a new system.

And then it finally occurs to them: there’s a post-office at the town square, and they know Ralph, who runs it, is a trustworthy person. Maybe Ralph can be a sort of broker for them? During the day, when rushing from one job to the next, Albert will leave a message for Mary. Mary, also rushing by, pops into the post office and asks Ralph - “are there any messages from Albert? If so, can I have them?”. Ralph, out of an abundance of caution, has made a copy of Albert’s message for safekeeping, and gives Mary this copy. Delighted, Mary leaves her own message for Albert. 

Soon after, Tina shows up at the post office, and asks Ralph - “do you have any messages from Albert? If so, can I have them?”. Ralph feels vindicated - he was right to copy those messages: he can now hand one to Tina, and keep another copy around in case anyone is interested.

Sometimes, Albert will stop by multiple times during the day, and leave a bunch of messages. During a particularly busy afternoon, Mary might only be able to stop by the post office once, on her way back home. She asks Ralph (who never sleeps) - “are there any messages from Albert? If so, can I have them?” And Ralph will gladly deliver the 15 messages Ralph left throughout the day. Mary, in turn, can leave her own.

Soon a problem comes up. It turns out that the brothers have a sense of humor, and in order to alleviate the trauma of the plague, have resorted to sending each other jokes. Tina, who is picking up all of Albert’s messages (remember, he’s the human doctor, and she wants to know how many beds to prep at the hospital), is starting to get tired of the recycled “knock knock” jokes. One day, speaking to Albert, she asks “why don’t you label your messages according to topic? I’m tired of the jokes, but your notes on the sick in this town are very helpful. While you’re at it, why don’t you tell Mary to do the same? I’m especially interested in how many animals she’s finding with fleas - I might have to buy special soap for patients that live with their animals.” 

And so, the previously uncategorized messages are now labeled according to their topics! Ralph, being the diligent postmaster he is, creates little cubbies for each topic. Now, when Tina walks in, she can ask “hey Ralph, are there any messages from Albert regarding people, and messages from Mary regarding fleas?” Albert turns around and looks at the section of cubbies for Albert, and finds the one about people (Albert/people), and then looks for the section on Mary, and finds the cubby for fleas (Mary/fleas). He promptly gives these to Tina, saying - “I see you’re interested in subscribing to the Albert/people and Mary/fleas messages - I’ll make sure to pass these on to you whenever you stop by”. When Mary stops by, she simply asks for all of Albert’s messages (Albert/*), regardless of topic. 

In this way, Albert and Mary have solved their communication issues. They’ve pulled on a trusty broker who can handle their exchanges, and by doing so, created a way of sharing information that is robust, lightweight, and suited to the nature of their work. Eventually they added even more features - sometimes Albert had a very important message for Mary, and requested that she leave a signed confirmation that she received it and read it. These were Quality of Service (QoS) offerings by Ralph, who understood that sometimes you needed to make absolutely sure the other person received what was intended for them.

In the end, thanks to MQTT, Albert and Mary saved the city of Sedona from the horrible plague, and retired to quieter lives. 
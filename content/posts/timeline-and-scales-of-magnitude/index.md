+++
date = '2025-09-27T19:08:33-04:00'
draft = false
title = 'Timeline and Scales of Magnitude'
tags = ['Project Development']
+++

## Timeline

| :----- | :---------------------------------------------------- |
| 30-Sep | Basic BLE network w/ sensor input, reporting to RBPi  |
| 7-Oct  | Substrate Material Palette and first conductive print |
| 14-Oct | Sensor refinement                                     |
| 21-Oct | Sensor refinement & RBPi Interface V1                 |
| 28-Oct | Sensor refinement & Sensor attachment system          |
| 4-Nov  | Sensor attachment system & sensor enclosure V1        |
| 11-Nov | Enclosure & sensor testing & custom board dispatch    |
| 18-Nov | Sensor & Enclosure refinement                         |
| 25-Nov | Full board and sensor assembly v1                     |
| 2-Dec  | General work & RBPi interface V2                      |
| 9-Dec  | Backend infra deployment & celular contract           |
| 16-Dec | Final Presentation                                    |

## Scales of Magnitude
#### $10,000 vs. $10

**Things that change**: Material selection, sensing method, test equipment, tooling, research extent (purchasing, etc), external hiring (consultants, assistants), marketing & branding.

**Things that don't change**: Core concept, form-factor, software, value proposition.

### $10,000

Likely the most important thing that changes here is sensor development and testing. With $10,000 it's feasible to not only hire a materials expert for consulting purposes, but also to engage in much more in-depth testing of a much greater variety of materials and ways-of-sensing than would be possible if working with a lower budget.

**Conductive Inks** - part of my idea is contingent on the use of printed conductive inks on flexible, adhesive substrates. What ink exactly to print is a bit of a crap-shoot still, and there's a few options. A higher budget would enable the purchase of a variety of test-inks, as well as their printing and testing with dedicated test equipment (saltwater-dispensing chambers, reliability testing, fatigue testing, etc.).

**Housing and Material** - achieving true IP67 or IP68 requires a good amount of materials testing, and, in an ideal world, the creation of custom injection-molded parts and gaskets (which even with a $10,000 dollar budget is a stretch). Again, above everything the budget allows for testing, and fairly extensive testing at that.

**Sensing Method** - There might be a world in which I adjust the trace-based sensing method to take advantage of more complex manufacturing techniques like, for example, the use of Kapton carbonization in order to create conductive traces (Kapton being fairly expensive unless purchased in bulk). That, or, more testing with custom flexible PCB boards and significantly more expensive trace-widths might be entertained.

All in all, I again think that a scale of $10,000 dollars is reflected largely in the amount of testing that can be done, in terms of materials, reliability, and sensor design. It might be nice to throw some of that money towards some good branding or advertising.

### $10

Things become pretty different at $10. Whereas before we're able to consider the entire scope of a project, here we're somewhat limited and have to become comfortable with a simple (hopefully working) prototype.

**Methods of Sensing** - The lower budget means a return to much more traditional methods of sensing. B recently mentioned more standard [leak detectors](https://www.ebay.com/itm/226779855314?mkcid=16&mkevt=1&mkrid=711-127632-2357-0&ssspo=iqIhimy1Rsu&sssrc=2047675&ssuid=&stype=1&widget_ver=artemis&media=COPY) that simply sense a bridge (much larger than ours) across two posts. A similar thing can be developed for our use-case, aided with nothing but some wire and/or copper tape. 

**Material** - The idea here would be to repurpose and enclosure, more than anything to hide the CR2032 battery that would power the little sensor. A small, plastic pillbox would do.

**Alarm** - Unlike our previous version where we could notify the user via cellular technology, the $10 budget means that **at most** we could have a small LED light up, and maybe have a small buzzer ring. Better than nothing, this would simply mean that the user would have to be on-board in order to detect the leak. Alternatively, the sensory could bridge part of the ship's 12V network, lighting up the anchor light on the mast, which would serve as an indicator to those in, say, a marina, that something is amiss.

What interesting is that looking at the $10 version, the main thing that changes is not the idea itself, but rather how "far" we can communicate a situation. This is an interesting revelation to me: technology in our case facilitates spatial compression more than anything.
# Dream Mobility — CTO Working Session

## Background & brief

**Format**: 2-hour working session on the whiteboard with the C-level team.

**Goal:** hear how you'd shape Dream's target tech architecture and how you'd build it toward our 2029 launch, and then a joint discussion on train IT & revenue management specifically.

Treat this as thinking out loud with us. The context below is how we currently see our tech, so we start from the same place – and pushing back on it is welcome.

---

## How we see our technology

dream is being built at an unusual moment. The tools to build software – and to build a *company* – are far more powerful than they were even a few years ago, and most of what they now make possible is still unclaimed. Travel tech especially is rigid and decades old: incumbents are stuck bolting new software onto systems built for paper tickets and national borders. Starting from scratch, we get to design the company and the product around what's now possible. Three convictions:

**Our product is an experience, and the train is a moving place we build around the people inside it.** A dream train is somewhere a guest lives for up to sixteen hours, and we get to shape every part of how that feels. The best technology on board is often the part the guest never has to operate, e.g., when the cabin senses and adapts around them – light, sleep, sound, climate. Because we're specifying the rolling stock now, that responsiveness has to be designed into the hardware itself, and what we decide today is effectively permanent when we launch.

**We can be a radically leaner company than any operator before us.** Because so much of running a business can now be automated, instrumented, and handled by very small teams, we believe a pan-European operation can run from an HQ of never more than ~100 exceptionally strong people. And in our trains just one host covers five coaches, focusing on highest value-adding activities only. We treat that as a deliberate design constraint that shapes how we build everything else, and it's the part that would have been impossible to claim ten years ago.

<!-- can you elaborate on this last sentence? -->

**We can run each train for far less, and earn far more from it, than incumbents can – and tech is a key enabler for that.** On the cost side, rolling stock that monitors itself, predictive maintenance, and smart energy and asset use keep operating costs well below the industry norm. On the revenue side, a continuous, network-aware pricing engine captures value that rigid legacy fare systems leave on the table.

<!-- pricing engine mentioned -->

**The key tech elements we currently see:**

- **Commercial & revenue**: the booking and reservation system that runs every sale, plus the pricing and revenue-management brain on top of it.
- **Rail operations**: running the network. Planning, fleet and predictive maintenance, crew, control, disruption recovery.
- **The dream app**: the guest's touchpoint across the whole journey, from booking to on board.
- **The on-board layer**: the train as a responsive place: sensing and controlling light, sound, climate, the seat, the whole ambience around the guest.
- **Our company OS**: the internal tooling and automation that let a <100-person HQ run a pan-European operation.

A shared data and event backbone runs underneath all of these and connects them.

---

## What we'd like you to walk us through

Rough initial thinking is completely fine — hand-drawn diagrams welcome 🙂.

- **Target architecture & ownership**
  Your view of the end-state shape for a company like ours, and any first thoughts on where you'd own vs. integrate.
- **How you'd build it**
  With a December 2029 launch, how would you use the long runway to arrive battle-tested rather than scrappy: what you'd start now, and how you'd harden the experience and systems before any paying guest?

---

## Two specific aspects we'd like to discuss together

The following are two real, open topics for us, and we'd like to work them through with you live. We're not looking for finished answers but instead are more interested in how you'd run problems like this with us.

**Topic 1 – a digitally born train.**

This year, we're finalising the purchase of our trains, i.e., the hardware that a big part of the on-board experience will eventually run on. This year's procurement contract, and the design-phase work with the OEM next year, decide what the manufacturer physically builds into the train and what interfaces it exposes. We'll be working with a train tech stack we don't fully own – often still built on legacy, manufacturer-controlled systems – and it freezes years before our software exists.

We'll take as given what you and Lorenz already discussed: a homologated, safety-relevant train system, and a second, independent edge-compute layer with defined interfaces to it, that we can iterate and upgrade on top of without touching the train. What's still open is the part we can't take back – the interface between the two, and the physical provisioning in the steel, are decided now. If the OEM freezes something we can't work with, or that's merely not ideal, we live with it until the next fleet.

We don't expect rail-hardware expertise (yet) – we're more interested in how you'd approach a topic like this, under deep uncertainty about what the product will need years later. Starting points for the discussion:

- How would you concretely approach this topic over the next months to make sure we're getting this right?
- What must we make sure the OEM's train exposes, accepts, and physically provides for our layer? How should we decide on where to over-provision for future optionality, and where to stay lean?
- Predictive maintenance and the in-cabin ambience (lighting, HVAC, seat door control) straddle the boundary – the data and controls sit partly in the safety domain. How should we deal with that?
- How would you de-risk the interface itself before a train exists, given the edge layer is the easy part to prototype?

**Topic 2 – how we build revenue management.** We've touched on it with you before, and now we'd like to go deeper and think with you about how to build it best for us. Part of what excites us is that we're building this from scratch in 2026: no legacy fare logic to inherit, and a chance to build it around what's possible now — when AI changes not just how fast you build, but how the system itself can think. Starting points for the discussion:

- What could be genuinely specific about our future revenue management – versus other rail operators, and versus off-the-shelf revenue systems – and what opportunities does that open for us? (The same train serving both overnight and daytime guests is the obvious example; we're curious what else you'd flag.)
- What subsystems would you break it into – pricing, inventory / capacity management, demand forecasting, and so on – and how would you design each so the whole is genuinely strong for us?
- Building this fresh today, with everything now possible that wasn't a few years ago – where does that change how you'd design or build it, versus how a rail or airline RMS was built ten years ago?
- Which of those subsystems make sense to build ourselves, and which to buy or integrate – and where would you draw that line?

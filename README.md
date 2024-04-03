# EDM
This text outlines a novel EDM Current Source Idea. It is, as far as I can tell, original, and has not been proposed so far, in this exact configuration. 
If you wish to research this topology and publish it, or reproduce it non-commercially, do so as you wish, but please mention this GitHub. Yes, I'm calling dibs on this idea. You can do what you wish with it, I seek only recognition. I would love to look deeper into this, maybe publish on this topic but it would require *a lot* of time and effort. Yes, I'm a little lazy right now, I have a lot on my plate, but I think the idea itself is interesting. Also, this might not work in the first place, so feedback is much welcomed.

# Introduction
I've been thinking about building an EDM (Electric Discharge Machining) setup for some years now. 
My thinking went a bit like this:

I have a limited idea of what the actual work parameters would need to be for this setup, due to the lack of information about exact EDM parameters and the large variation between different setups, they should be widely adjustable. As widely as possible. I have looked at a variety of different topologies, of course, I read the classic by Makarand M. Kane, Ajinkya Ajit Phanse, Himanshu J. Bahirat, and  S.V. Kulkarni, "Classification and comparative study of EDM pulse generators". And many others. I have a general idea of how the EDM process functions and the working parameters, roughly speaking.

**How an EDM supply should look like ideally (as of my understanding):**
Ideally, an EDM supply should be a voltage-limited current source (CS). The voltage-limited current source can be toggled itself or can be switched on and off using an external switch. This is dictated by the commonly described steps of the EDM process, which, after turning on the pulse is:

> 1, a low-current high-voltage dielectric breakdown, then 

> 2, A constant, high-current low-voltage conduction work period,

followed by the extinguishing of the pulse, which may be a set time after the beginning of the pulse, or after a specified time spent in the work period, alternatively. Following this, the different high-ion and particle-polluted dielectric medium is flushed in the off-period.

The characteristic of *increasing the voltage* for the breakdown of the dielectric is *naturally achieved by current sources*. (Most) Current sources are also intrinsically *safe to short*, as in a short condition, the *voltage drops* rapidly, dropping the power as well. I would expect a lot of hard short conditions in an EDM power supply.

# Design Considerations

**1, For the CS control and feedback simplicity's sake, the design should use an external toggle for pulse generation (Or not, I would have to do the math on closed-loop  response to know precisely).** 

**2, The CS should be simple to control in itself, and be resistant to rapid load changes, and short conditions**

**3, The EDM supply topology should be capable of a wide range of adjustments in all relevant parameters, especially current output**

**4, The EDM supply topology should use COTS parts as much as possible, and be relatively cheap**

**5, The topology should be high efficiency, like 80-90+ % efficient. Resistively dropping voltages, using a lot of diodes as switching elements, hard switching, etc are to be avoided**

**6, The topology should be scalable.**

For considerations 2, 5, and 6, my attention was turned towards resonant topologies.
I found some prior work using LLC and LCC resonant converters as sources for EDM supplies, eg.:

**LLC:** Carl Odulio, Jon Clare,  Alessandro Costabeber, Allan Nerves, Raymond Ridley, and Alan Watson: Design of an LLC Resonant Converter for EDM
Applications

**LCC:** Rosario Casanueva, Francisco J. Azcondo and Salvador Bracho: Series–parallel resonant converter for an EDM power supply

The LCC or series-parallel resonant topology, or T13 in "Classification and comparative study of EDM pulse generators", is especially interesting for its capacity to supply constant current with very low feedback if it's operated near its resonant frequency. The math  of it lets it produce close to one specific output current at the resonant frequency. This is great. Due to its resonant nature, switching losses are almost non-existent. Efficiency is extremely high. One drawback: No real control over the current output. This is the reason for the mentioned rough cutting surface, and the inability to adapt this topology widely.

I wanted to build something with this LCC topology in mind, after crunching the numbers on various buck-type schemes and being horrified over and over again by the inefficient nature and overall complicated power flow if one wants to go 110/230 V > EDM Supply. It takes like 3-4 stages of power conversion to reasonably do it, with safety isolation. (I'm only a third of an EE so maybe I'm off on that point but I ran a few designs by really rough numbers and estimates on paper and in SPICE and that was the best I could manage).

# The core of the idea

As I was thinking, it dawned on me, that by the time the voltage gets through the resonant tank, it roughly estimates a sinusoidal wave. Of course not perfectly, but meh, good enough for an idea. Then this sinusoidal wave enters the transformer for conversion. Maybe if we had two sine waves in that transformer, at the same frequency, they would be added... Interestingly, the only thing that changes is the amplitude. **And that amplitude is dependent on the instantaneous phase!**

So that was my idea. Let's use a transformer with two primary windings, connect them to two separate resonant tanks, and drive the tanks with a phase difference!
The frequency is mostly constant (*not really, more at 11*) and should be somewhat above the resonant frequency of the tank, so we're constantly in resonant switching, the switching elements see no power. The output current is more or less only dependent on the phase between the two resonant tanks. In practice it depends on the load too, but quite linearly and predictably, on paper. When you change the phase, however, the instantaneous frequency changes as well because **frequency is the derivative of phase**. 

But in my mind, this should be okay too, because as you change the instantaneous frequency to change the phase, it changes the current in the direction you want it to change in the first place. 
As the phase stabilizes so does the frequency. If we limit the maximum change of phase, the effect of changing the frequency can be minimized. (I think intuitively, no rigorous math here).

Also for changes decreasing the frequency, they should be limited to avoid getting into the Zero Current Switching danger zone (Zero Voltage Switching = GOOD, Zero Current Switching  = BAD, to put it shortly, also, to my knowledge). The frequency should also be high enough to help avoid this, but low enough to keep the constant current nature of the supply as pure as possible. The more we deviate from the resonant frequency, the worse the supply does as a constant CS.

As I was thinking through all of this, it hit me, that it all works too perfectly (on paper), and good original ideas are hard to come by, so someone else must have thought of this by now.

After some digging, I found a Chinese paper where the authors actually did this, (Qi Cao, Zhiqing Li, Bo Xue, and Haoyu Wang, "Fixed Frequency Phase Shift Modulated LLC
Resonant Converter Adapted to Ultra Wide Output Voltage Range") but using LLC resonant tanks, and they ended up with a fairly good, wide-range constant voltage source (LLC resonant tanks exhibit a constant voltage nature at their resonant frequency). And they worked out all the math for the LLC case! Wonderful! Also kind of sad as my idea isn't as original as I first thought it to be, but still a fairly exotic one, and in this specific application and configuration, definitely a novel one, to my knowledge at least.

# Some further considerations

**Pulse Generation**

It is possible to use this described phase-shifted dual LCC topology to supply a constant current, especially with a current-smoothing inductor on the output (maybe, sounds good to my intuition). It can be used alongside a high-power FET or IGBT switch to make an EDM pulse generator. 

Or, by Shanon and some added margin, we can use feedback and the set point to turn the supply on and off, by having a maximum EDM pulse frequency, multiplying it by 5, taking our samples for feedback at this frequency, and again multiplying by around five times to get our switching frequency. Five is more or less practical, *theoretically*, we could do with 2. In practice, it's often not enough. But this way we waste less power at the cost of reduced EDM pulse frequency and increased switching frequency. Also, it's possible this idea would lead to problems with pulse ignition, this would have to be verified by testing.  

We can supplement the feedback by feedforwarding the coming pulses too, increasing response and stability.

**EDM Feedrate**

EDM feed rate would have to be controlled dynamically by measuring each pulse's voltage levels and classifying it accordingly. A fuzzy or similar controller could then set the feed rate based on the frequency of a given pulse type. 


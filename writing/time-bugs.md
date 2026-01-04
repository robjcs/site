# Time Bugs
I recently read a blog post [Clock Synchronization Is a Nightmare](https://arpitbhayani.me/blogs/clock-sync-nightmare/) by Arpit Bhayani, which describes the non-trivial problem of syncing clocks. I love this topic because it's typical of the class of problems, and solutions, that we take for granted in everyday engineering (and life!).

So we can discuss how we sync clocks between computers to some tolerance, but the bigger question is: how do we set the clock in the first place? How do we actually track the world's time?

I expect that most people would assume a couple things about how we solve this problem in the modern day:
- There is a single global truth
- It is infallible

The purpose of this post is to explain that in fact, neither of these assumptions are correct.

<figure>
  <img
  src=/writing/images/atomic-clock.png
  alt="atomic clock diagram and photo, from NIST">
  <figcaption>NIST-F4, Diagram of a fountain atomic clock, Credit: A. Novick/NIST</figcaption>
</figure>

### The Source

Time standards are defined by atomic clocks. The very basic operating principles of these are as follows:
- A microwave oscillator shoots radiation into some atoms (typically cesium-133)
- Some atoms are excited by the radiation. (I would be too!)
- Lasers are shot at the atoms. The more excited by the radiation they are, the more they light up
- The radiation frequency is slightly tuned and this process is repeated until it is fully optimized (all the atoms are excited)

At the optimization point, the radiation is tuned to that atom's resonant frequency. The result of this, when working with atoms with known resonant frequencies such as cesium-133, is an _absolutely_ calibrated microwave frequency.

In SI units, we define 1 second as 9,192,631,770 oscillations of an oscillator tuned to the resonanace of cesium-133. This is the foundation of our measured time system.

### UTC, UT1, and Leap Seconds

Atomic clocks around the world work continuously. To maintain an _absolute_ calibration, clocks around the world are run against their cesium-133 samples and their results are averaged together to create a globally accepted interval of a _second_.

This average, made of ~450 clocks worldwide, are the foundation of International Atomic Time (TAI). However, TAI is only one of many time standards. TAI differs slightly from the actual speed of earth's rotation, so it is adjusted by leap seconds to form Universal Coordinated Time (UTC).

Time according to earth's rotation, called Universal Time (UT) or UT1, is defined to satisify the following relationship:
$$\theta(t_u) = 2\pi(0.7790572732640 + 1.00273781191135448 \cdot t_u) \text{ radians}$$

where

$$t_u = \text{Julian UT1 date} - 2451545.0$$

Where a Julian date is days since 12:00 January 1, 4713 BC, expressed with a decimal calculated to he current instant.

Since 1972, 27 leap seconds have been introduced to (added to) UTC to align it closely with UT1. At the time of writing, TAI is ahead of UTC by 37 seconds.

Now consider another time standard, GPS time, which broadcasts its time signals globally and sets the time for a lot of phones, computers, and watches. GPS is actually maintained _independently_ from UTC, but is frequently synced to it. This means there _could_ be slight deviation between UTC and GPS time, meaning __UT1, UTC, GPS, and TAI all have different values at a given instant.__

### When It Fails

Earlier this month, a windstorm in Colorado caused extended power outages at the U.S.'s National Institute of Standards and Technology lab in Boulder. When a backup generator failed, UTC (U.S.-based average) time deviated by 4.8 microseconds. 

At the NIST laboratory, between 10-15 atomic clocks are running at any given time. Assuming 15 are affected, and assuming they are all contrubuting to the ~450 used globally to define UTC, the net effect of a 4.8 microsecond deviation on an even average of 450 atomic clocks for UTC calculation could result in a

$$\frac{(4.8e^{-6} + 1) * 15 + 1 * 435} {450}$$

$$\frac{450.0000720}{450} = 1.00000016$$

$$= .000016 \\%$$

deviation of UTC.

Of course, this is a small deviation, but still very significant considering the scale of applications, people, science, and industry that time deviation affects.

During the power outages, the NIST's affected clocks were excluded from the UTC average. Even in the case that there was some captured error, it could be accounted for later by the addition or omission of a leap second as necessary. But, this still highlights the sensitivity of this standard, and how our engineered solution to this foundational problem are not infallible. 

Imagine the effect of a drift on a larger scale. A power outage, flood in a laboratory, or a cyber attack poisoning a data reporter. How much could our lives change in a second?
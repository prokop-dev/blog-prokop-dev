---
title: "PEN fault device"
date: 2023-05-06T06:06:06Z
draft: false
author: "Bart Prokop"
description: "How to make your own PEN fault detection device for Tesla Wall Connector"
tags: ["DIY", "spark", "electricity"]
ShowToc: true
---

# Disclaimer

Electricity can kill or severely injure people and cause damage to property.

I'm not a qualified electrician. I accept no responsibility for anything you decide to do. I'm just posting this up for information only. This page reflects my own research and experiments.

In most jurisdictions you must be a *competent person* to carry electrical work. If you are not confident in what you are doing, you should employ the services of a qualified electrician.

The information contained herein is not mean to be comprehensive and is for informational purposes only. You should not undertake to perform anything described herein without adequate training and/or supervision. The Author disclaims any responsibility for any injury, damage, or loss as a result of reliance upon the information found on this site/blog.

Amazon links are affiliate links.

# Tesla Wall Charger Gen 3

For couple of months I was using granny cable to charge my PHEV. Not the best experience, especially that I park my car in driveway and charging brick is connected to a 13A socket in garage. Then, broomstick is used to secure a gap in garage door, so charging cable can fit it. It was the highest time to get proper charger.

I was really baffled with the charger offering in UK market. Most of chargers looks like DIY stuff put inside [Weatherproof electrical box](https://www.amazon.co.uk/Waterproof-Electrical-Connector-Weatherproof-150x110x70mm/dp/B09SPB39B9?crid=3HJID6Q3ZRHGA&keywords=Weatherproof+electrical+box&qid=1683411704&sprefix=%2Caps%2C175&sr=8-26&linkCode=ll1&tag=bartprokop-21&linkId=68baea70c390d42563ae0c01c3dd6eca&language=en_GB&ref_=as_li_ss_tl).
And really, researching the Internet shows that most of them have "stocks" internals. You can even build your own, there is one gentleman living in some secluded island who runs online shop with all components and provides excellent tutorials. I highly recommend you to check it out: [evbitz.uk](https://evbitz.uk).

After long research, I come to conclusion that actually Tesla makes really nice charger. And in UK (and European) market, Tesla chargers can work with all car brands - what makes the choice really futureproof. So I snipped [TESLA Wall Connector](https://www.amazon.co.uk/gp/product/B0BW16BBKQ?&linkCode=ll1&tag=bartprokop-21&linkId=f80508bd9d0e038eeabf68307cebd38b&language=en_GB&ref_=as_li_ss_tl) on Amazon.

While considering the Tesla charger, I noticed someone complaining in review about lack of "PEN fault detection" by the charger and the need for the "extra and costly" device to be fitted by electrician, to achieve regulation's compliant Tesla Wall Connector instalation. I've almost decided against buying it, but after doing extra research realised that actually charger must not implement "PEN fault devide". The reason for this is EIC-61851-1 that forbids an EV charger to disconnect the protective earth conductor.

Most UK chargers now implement "PEN fault protection device" and, I suppose are better avoided because of it. I'm exploring that topic below, but shortly: I think (as reasonable, but not necessary by letter of regulation) that PEN fault device should be separate from charger and installed inside and not as part of outdoor appliance.

# TN-C-S and PEN fault research

[TN-C-S is used for the electricity supply](https://en.wikipedia.org/wiki/Earthing_system) to the majority of UK electrical installations. Adding an EV charger creates a potential safety issue if an [open PEN conductor fault](https://electrical.theiet.org/wiring-matters/years/2021/84-march-2021/broken-pen) occurs within the electricity distribution network. Because of this, in the UK, EV chargers that are used to operate outdoors have a legal requirement to be protected from ‘neutral faults’ when connected to a PME (protective multiple earthing) network. This ensures that the user cannot get an electric shock if the supply neutral becomes disconnected. This is done by disconnecting the live, the neutral, and the earth connections to the vehicle if a fault is detected, in accordance with IET wiring regulation 722.411.4.1.

## What to protect against?

Please watch this video: [TN-C-S Danger - Broken PEN Conductor (Combined Earth & Neutral)](https://youtu.be/JRHyqouJPzE).

A broken PEN conductor (before installation) can be potentially fatal as your car body can get live potential. If you touch your car and at the same time something that has good contact with earth you will be electrocuted. Note that your existing RCD protection will not work. This problem is not specific to electric cars or EV chargers. It is specific to having conductive (e.g. metal) parts of outdoor device connected to PME network. Note that car tyres are isolating car body from earth, so electric potential cannot be neutralised.

To protect against this scenario, it is necessary to either provide a dedicated earth to the EV charger or fit a PEN fault protection device that will automatically disconnect the PEN. If there is a true dedicated earth available and the earthing system is in good order, PEN fault protection may not be required. As this is usually not feasible (and in most cases neither practical nor economical), then PEN fault protection must be fitted.

As PEN fault danger is primarily related to outdoor appliances, I would argue that any protection device should be housed inside your property and not be a part of the charger. That is why I actually decided not to have charger with PEN protection built in. Quote from clause 8.4 of BS EN 61851-1:

> For Modes 3 and 4, permanently connected EV supply equipment, protective earthing conductors shall not be switched.

There is [BEAMA Technical Bulletin](https://www.beama.org.uk/static/794d3387-de78-49cb-9d89639c4aea5310/PROTECTIVE-EARTH-DISCONNECTION-WHEN-CARRYING-OUT-ELECTRIC-VEHICLE-CHARGING.pdf) that covers the possibility of building-in PEN fault protection device into EV charges. In my opinion, PEN fault detection is not something that charger should be doing and I steered away from chargers having "Declaration of Confrmity" stating "with the exception of clause..." and "...this clause conflicts with UK’s IET Wiring Regulations". Simply speaking, manufacturers know customers will be unhappy with yet another clunky device on the wall, so "let's put it in clunky EV charger". My point is that having "unprotected PEN conductor" outdoors (i.e. inside charger) is risk on its own. Similarly, I think the separation of duties principle supports separation of earthing function from charging into two devices. Not to mention that building own PEN fault detector is cool learning excercise for someone with my background.

## What other protection is required?

PEN protection fault will inevitably become a sort of "consumer unit" shape box hanging in my garage. As likely it will become powered directly from meter tails (e.g. just to prevent any cascading RCD issues), let's just check for other protection requirements.

The requirements for charging of electric vehicles were the subject of amendment 1 to BS 7671 which came into effect in July 2020.

### RDCB, MCB or RCBO

Regulation 722.531.3 requires that an RCD (max 30mA) supplies a car charger. This RCD shall disconnect all live conductors including the neutral.

### MBC

# Designing my own PEN fault device

The modus operandi for "PEN fault device" is simple. When fault is detected, just disconnect all the cables.

# Parts

Bill of materials and what it costed me:

- £14 for [Small 6 ways Distribution Board](https://www.amazon.co.uk/dp/B018SP2GP0?psc=1&linkCode=ll1&tag=bartprokop-21&linkId=307bbb0a2153d48d75da4b26cecce70a&language=en_GB&ref_=as_li_ss_tl)
- £7 for [Voltage Monitoring Relay](https://www.amazon.co.uk/Protector-Adjustable-Protection-Stabilizer-Appliance/dp/B097RPCTJS?pd_rd_w=XYY7x&content-id=amzn1.sym.16225f1b-bac2-4141-a623-80a3a4c8dce7&pf_rd_p=16225f1b-bac2-4141-a623-80a3a4c8dce7&pf_rd_r=6FHG40YXYPT21QGJ4EG2&pd_rd_wg=FiXBK&pd_rd_r=3778f606-483c-4f3f-bb76-26b2da907608&pd_rd_i=B097RPCTJS&th=1&linkCode=ll1&tag=bartprokop-21&linkId=e882691f1b1d69e19c41e0f5f5d543dc&language=en_GB&ref_=as_li_ss_tl)
- RCBO 32A Type AC and 

A wee note on prices. I did a lot of opportunistic purchases. Instead of "consumer unit", I was able to grab professional distribution board, which was insanely cheap on Amazon (likely because manufacturer has upgraded its offering). Same thing on other suppliers is listed for over £100. I was able to grab some compatible KQ (or iKQ) MCBs and RCBOs on eBay. I believe, I have built much better thing for 30% of "market price".

# Closing notes

I.

# Emergency Communication Platform 
This is a work in progress emergency communication system design blueprint.
`EMCO` for short. It is the abbreviation of `Em`ergency `Co`mmunication

- [TLDR](#emco-protocol)
- [Project Organization Page](https://github.com/Emco-Project)
- [Disaster.radio] (https://disaster.radio/) I recently found out about them.

## WHY / HOW / STATEMENT
We're living in a connected yet isolated world. We've all the technology yet it seems not utilized enough.

Troublesome things are happenning all around the world, and we see that communication infrastructure (& providers) takes the first hit after a disaster. I wanted to make a messaging tool, something like pager. I've been thinking about [LoRaWAN](https://hackaday.com/blog/?s=lorawan) text messengers for a long time. But I'm short on electronics knowledge to make (a good) one. 

I've seen [1999 earhquake](https://en.wikipedia.org/wiki/1999_%C4%B0zmit_earthquake), we know [protests](https://en.wikipedia.org/wiki/Mahsa_Amini_protests), fires, [invasions](https://en.wikipedia.org/wiki/2022_Russian_invasion_of_Ukraine), hurricanes all around the world. Now another [earthquake](https://en.wikipedia.org/wiki/2023_Turkey%E2%80%93Syria_earthquake) and same things all over again. 

I had enough.

Either by the governments or technical reasons, first thing we loose is communication.
I wanted to do something about it, instead of expecting governing people to do something about it.

### It has to be `simple`, `accessible`, `extendible` (sae?)
I've an idea. Inspired by the [nostr](https://github.com/nostr-protocol/nostr) I wanted create a protocol supported with;
1. Software Servers
2. Hardware Server
3. Clients (Software & Hardware)

> Every client should also be the member of the mesh network an act as a range extender.

`This is just a draft idea` and I want your opinion on this, you can read more about what I am trying to do below. I know the power of the people (Internet). 

Thank you.


> NOTE: Everything is subject to change.

## EMCO Protocol
`EMCO-P` is an emergency communication protocol designed to be accessible for any type of situations.

Can be used with WebSocket, HTTP Post or radio frequency.
All request (package) parameter names are kept as short as possible in order to lower the amount of data transferred. This should be helpful for limited connections.

To make it readable, all parameter names are usually synonyms.


## EMCO Relay 
It's a service (hardware or online server) for relaying and saving messages. Can be mentioned as `EMCO-R`

Initially it is a web server but planned to be extended to work on hardware level too.

## EMCO Client
A Client can connect via `EMCO-P`
There are 2 types of clients.
1. Hardware Clients.
2. Software clients.


## EMCO Network (Mesh System)
It's (ideally) a hardware to extend network communication, but clients can be used for this purpose as well.
Bridge devices can translate or transmit between protocols LoRa <-> TCP 
- LoRa => Wifi Bundle
- LoRa => Ethernet Bundle 

# EMCO Hardwares
- Hand Terminals (Text Messenger)
  - LoRa 915, 868, 433
  - nRF24L01 (?)
- Access/Repeater Points (LORA To TCP / TCP to LORA) + w/ Capture Portal
  - Bluetooth Mesh (?)

![Mesh Schema](https://github.com/siniradam/emco/blob/main/assets/emco-mesh-schema.png?raw=true)
**Portable HQ:** It's a simple mobile unit (laptop?) that can access and list requests in a disaster zone, can be used by rescue teams.

## Basic Data Structure
Pretty much every parameter is optional.
```jsonc
{
    is:"PUBKEY",//Issuer
    sig:"SIGNATURE",
    tm:{ //Telemetry
        //Location
        lt:"", //Latitude
        ln:"", //Longitude
        csp:"", //Coordinate source / provider & precision 
    
        //time
        ldtz:"", //ISO String
        ut:"", //Unix Time

        //Addres Parameters 
        ac:"",//Country
        as:"",//State
        at:"",//Town (City) 
        am:"",//Muncipality
    },
    r:{//Request
        t:"",//Type
        c:"io.ohshift.emco.v1",//Client definition.

        m:"",//message string
        sc:"",//Request content flag (aka short code), can be used without message.
        th:"",//Thread ID 

        q:[//Query
          {t:"",v:""}
        ]
    },
    i:{//Identification
    //This is an object for a client/user to introduce itself.
    //Can contain informations like; 
    n:"",//User name
    t:"",//User title
    b:"",//User bio
    p:"",//User photo
    // - name, 
    // - photo, 
    // - location, 
    // - bio, 
    // - twitter, 
    // - title

    },
    e:[//Echo
      {//Needs to be explored.
        is:"ISSUER",
        sig:"SIGNATURE"
        //Echo object should contain its own signature, 
        //Echo object should be ignored when calculating request signature
        //Echo message mustn't modify original request.
      }
    ]
}
```

| Key   | Type                        | Required | Description                                                                                | Sample                    |
|-------|-----------------------------|----------|--------------------------------------------------------------------------------------------|---------------------------|
| is    | Issuer                      | Yes      | 32-bytes lowercase hex-encoded public key of the event creator.                            |                           |
| sig   | Signature                   | Yes      | 64-bytes hex of the signature of the sha256 hash of the serialized event data.             |                           |
| tm    | Telemetry                   | ~        | Telemetry Data Object.                                                                     | {}                        |
| tm.lt | Latitude                    | No       | Latitude coordinate.                                                                       | 37.4459                   |
| tm.ln | Longitude                   | No       | Longitude coordinate.                                                                      | 43.7450                   |
| csp   | Coordinate Source Precision | No       | 2 Character identifies source of the coordinate and how precise.                           | MA (Mobile & Approximate) |
| ldtz  | Locale date time w/ zone    | Yes      | ISO Local date time with zone information. Not needed when ut is provided.                 | 2023-02-11T04:09:15.655Z  |
| ut    | Unix Time                   | Yes      | Can be used over ldtz.                                                                     | 1676087538810             |
| tm.ac | Country ISO 2 Chars         | No       | ISO 3166 ALPHA-2 Country Code                                                              | TR for Turkey             |
| tm.as | State Name                  | No       | State name abbreviation if available.                                                      | CA                        |
| tm.at | Town / City                 | No       | Name of the city                                                                           | Hakkari                   |
| tm.am | Muncipality                 | No       | Next type of location under a city                                                         | ??ukurca                   |
| r     | Request Object              | ~        |                                                                                            |                           |
| r.t   | Request Type                | Yes      | Single char request type. See request type table.                                          | P                         |
| r.c   | Request Client              | Yes      | Web service for resolving request contents with version info. See [Request Client](#request-client) section.  | io.ohshift.emco.v1        |
| r.m   | Request Message             | No       | Optional text content for a request. Up to 320 characters.                                 | Fire, help!               |
| r.sc  | Short Code                  | No       | Optional short code for various situations. See Short Status Code Table                    |                           |
| r.th  | Thread Id                   | No       | If this message is an addition to another message, specify sig id of the previous message. |                           |
| i     | Identification Object       | ~        | Client identification broadcast data.                                                      |                           |
| i.n   | Name                        | No       | User name                                                                                  |                           |
| i.t   | Title                       | No       | Any title a user can use                                                                   |                           |
| i.b   | Bio                         | No       | 320 Characters text                                                                        |                           |
| i.p   | Photo URL                   | No       | A web URL or encoded file                                                                  |                           |
| i.smt | Twitter handle              | No       | Twitter handle                                                                             |                           |


## Coordinate Definitions
- Coordinate provider types
  - M: Mobile
  - I: ISP
  - G: GPS
- Precision types.
  - A: Approximate
  - P: Precise
  - U: Undetermined / Unkown


## Request Types
These are the types of messages.
- P: Ping
- M: Message
- I: Identifier
- Q: Query
- S: Subscription
Idea: by adding '@' symbol to the type it can be indicated as test. Might be helpful while developing.


## Requst Query / Subscription parameters (`r.q`)
It's an array object, contains parameter and value pairs.
- before => Requests before this date.
- after => Requests after this date.
- is => Requests by issuer id
- tid => Thread ID (Root ID of an event chain. It is original request's signature ID)
- sc => Requests flagged with this short code.
- ac => Filter by Country
- as => Filter by State
- at => Filter by Town (City)
- am => Filter by Muncipality

## Echo Data
Relayed messages are called echoes. I need to think more about this topic.

My intial thoughts are; 
1. If a request does not contain a location relay can add it's own location
2. Echo should contain a identifier like: It's hardware or a server etc.


## Short Codes

<table>
<tr><th>
Codes</th><th>Status Values</th></tr>
<tr><td>

| Code | Meaning |
|---:|---|
|AA|  ALL OK |
|A |  Just messaging |
|A0|   |
|B |  `Bio Hazard` |
|B1|  Bio hazard exposure |
|B2|  Chemical exposure |
|B3|  Radioactivity exposure |
|C |  `Earthquake` |
|C1|  Under wreck |
|D |  `Flooding` |
|D1|  Flood building |
|D2|  Flood area |
|E |  `Explosion` |
|E1|  Near explosion|
|E2|  Witnessed explosion|
|F |  `Fire` |
|F1|  Wildfire |
|F2|  Homefire |
|G |
|H |  `Hurricane` |
|H1|  Stranded in a hurricane zone |
|I | |
|J | |
|K | |
|L|  `Life threatening` situation |
|L1|  Heart Attack |
|L2|  Attacked / Stabbed / Wounded |
|L3|  Being followed |
|L4|  Need medicine |
|L5|  Need water |
|L6|  Need food |
|L7|  Need shelter |
|L8|  Trapped |
|L9|  Trapped |
|M |  `Medical` |
|M1|  Insulin |
|N |  `Nature` |
|N1|  Landslide |
|N2|  Storm / Tornadoe / Hurricane / Typhoon |
|N3|  Avalance |
|N4|  Under Avalanche |
|N5|  Flood |
|O||
|P||
|Q||
|R||
|S| `Sickness` |
|T| |
|U| |
|V| |
|W | `Water` (Sea, Ocean related messages) |
|W1| Stranded |
|W2| Boat, ship crash |
|X | `Emergency responses` |
|X0| Ping |
|X1| Yes |
|X2| No |
|X3| Maybe |
|X4| Possible |
|X5| Impossible |
|X6| True |
|X7| False |
|X8| False |
|X9| Rescued |
|Y | |
|Z | `Emergency updates` |
|Z0| Request received|
|Z1| En route |
|Z2| Getting close |
|Z3| Arrived |
|Z4| Started |
|Z5| Completed |
|Z6| Leaving |
|Z7| Can't Locate |
|Z8| Wrong Address |

</td><td valign="top">

| Code | Description |
|---|---|
| 0 | All Good |
| 1 | Mildly Affected |
| 2 | Moderately Affected |
| 3 | Severely Affected |
| 4 | Very Severe |
| 5 | Getting Worse |
| 6 | Getting Better |
| 7 | Stranded |
| 8 | Rescue Here |
| 8 | Rescued |
| 9 | Witnessing |

</td></tr> </table>

## Ideas About Short codes
- First character => Category.
- Second charater => Category detail.
- Third character => Status of the requester.
> Examples: 
> F23 => Wildfire, severely affected
> 
> 

## Request Client
Request client data is basically a definition file for the client itself.
- io.ohshift.emco.v1 => `translates into` => https://emco.ohshift.io/v1.json
  
This file should contain information about how client operates, the api version it uses and other details about the client capabilities.


### Mind Map - Topics to explore
> A Client can store, relay, if required add it's own location while relaying.
>
> Twitter proxy? (@Mention a relay's twitter account to send message.)
>
> Tweet capture => If a twitter acount of a relay is mentioned, location of the tweet & user & situation is captured.
>
> Heart Beat => Telemetry about a person while an act
>
> Automated communication (a smart device sending a request detected automatically like a smart watch maybe)
>
> Regular messaging might/should be allowed but mesh access points also should prioritize emergency messages.
> 
> Initial message can be broadcasted over and over again until it gets a ping response.
> Maybe just be sure, after a ping, can be repeated with a longer intervals.
> ---

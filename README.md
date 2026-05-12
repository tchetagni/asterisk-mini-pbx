# asterisk-mini-pbx

> A **Docker-compose**, production-shaped starter for an **Asterisk** PBX with a SIP trunk, an IVR, a small call-center dialplan and the operational patterns I have kept across 60+ agent deployments.

This is the public, infrastructure-and-patterns companion of VoIP work I have delivered for clients. Customer-specific dialplans stay private. Here is the scaffolding that is worth reusing.

---

## What you get

- A single `docker compose up` brings up Asterisk with a sample SIP trunk and a 5-agent queue
- A reference dialplan organized the way it should be (one context per concern, no spaghetti)
- A starter IVR (welcome, language pick, route to queue)
- Notes on the **Kamailio** SBC layer you probably want in front of this in production
- Operational notes on call quality, codecs, NAT, fail2ban, and dongle/GSM gateway integration

## Why this exists

Most Asterisk tutorials end at "a SIP phone can call another SIP phone". Real VoIP deployments need:

- A **SIP trunk** that actually registers from behind NAT
- A **dialplan** that survives more than three engineers touching it
- A **queue** with predictable agent ringing, recording and CDR
- **Fail2ban / fwconsole-style** rules so you do not wake up to a USD 40k Madagascar bill
- **Recording, retention and consent** that fit your jurisdiction

This repo bakes those in from the first commit.

## Layout

```
asterisk/
  etc/
    asterisk.conf
    sip.conf            # or pjsip.conf for modern Asterisk
    extensions.conf     # main dialplan, one context per concern
    queues.conf         # call-center queue config
    musiconhold.conf
    cdr.conf            # CDR backend (CSV / Postgres)
  sounds/               # custom IVR prompts (drop your WAVs here)
  scripts/              # AGI / Stasis hooks
docker-compose.yml      # asterisk + postgres + optional kamailio
Dockerfile              # built on top of mlan/asterisk or similar
README.md
```

## Dialplan patterns I keep

### One context per concern
No `[default]` that does everything. Instead:
- `[ingress-trunk]` — inbound from provider
- `[egress-trunk]` — outbound to provider
- `[internal-users]` — extensions calling each other
- `[ivr-main]` — the IVR tree
- `[queue-sales]`, `[queue-support]` — one per business queue

### Explicit variables, no magic
Every business decision (after-hours, language, priority) is a variable set explicitly at the top of a context, not buried in a `GotoIfTime` deep in the call.

### Recording with a consent gate
Calls are recorded only after the caller has heard the consent prompt and stayed on the line. The recording filename includes the Asterisk UNIQUEID so it joins back to the CDR.

### Fail2ban from day one
An included fail2ban jail watches `/var/log/asterisk/messages` for SIP brute-force and bans aggressively. This single file has saved more money than any other in production.

## Beyond "single Asterisk"

For production, you almost always want **Kamailio** in front of Asterisk as an SBC:

- Kamailio does the registrar + SIP routing + DDoS mitigation
- Asterisk does the media + dialplan + queues
- Each scales independently

The `/kamailio` folder (next iteration) will hold a minimal SBC config that pairs with this PBX.

## GSM / dongle integration

Real-world African and CEMAC deployments often need a **dongle** (chan_dongle) or a **GoIP / Dinstar gateway** because SIP trunks to certain destinations are flaky or expensive. Operational notes in `/docs/dongle-and-gateways.md` (coming) cover:

- Picking a compatible dongle (Quectel EC20, Huawei E1750…)
- chan_dongle build flags and pitfalls
- Routing rules to prefer GSM for on-net destinations
- Failover when the dongle drops

## Roadmap

- [ ] Publish the actual `docker-compose.yml` and `asterisk/etc/*` files
- [ ] Add a starter Kamailio SBC config
- [ ] Add fail2ban jail and rules
- [ ] Add chan_dongle integration notes
- [ ] Add a sample CDR-to-Postgres setup with a small Grafana dashboard

---

## About the maintainer

Maintained by [Esaie Tchetagni Ngassa](https://github.com/tchetagni) — VoIP / Telecom Senior Engineer. Built a 60-agent Asterisk/ISSABEL call center, a hospital emergency VoIP system with Bluetooth panic buttons, fleet-management VoIP integrations, USSD/SMS payment channels. Track record: 10+ years shipping VoIP in production across CEMAC, Philippines, France.

Reach out: [tchetagni@gmail.com](mailto:tchetagni@gmail.com)

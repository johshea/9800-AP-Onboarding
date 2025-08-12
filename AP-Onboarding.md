# Catalyst 9800 AP Onboarding Process

## Overview
This document provides a step-by-step process for onboarding an AP to a Cisco Catalyst 9800 Wireless LAN Controller (WLC), detailing required policies, tags, and configurations for both **central-switched** and **flex (local-switched)** SSIDs.

---

## Mental Model (Mapping Components)
- **WLAN Profile**: SSID definition (name, security)
- **Policy Profile**: Client behavior (VLAN, ACLs, QoS, session timers, central vs local switching)
- **Policy Tag**: Binds WLANs to their policy profiles
- **RF Profiles**: Radio settings per band
- **RF Tag**: Assigns RF profiles to APs
- **AP Join Profile**: AP join settings (optional tweaks)
- **Flex Profile**: FlexConnect settings (local switching/local auth)
- **Site Tag**: Assigns AP Join profile and optionally Flex profile

**Every AP has three tags:**
- Policy Tag
- RF Tag
- Site Tag

---

## Prerequisites
1. Configure NTP, time, country code(s), management SVI/routing, DNS, DHCP/relay, device cert trust.
2. AP Discovery: DHCP Option 43, DNS `CISCO-CAPWAP-CONTROLLER`, or static.
3. AAA: RADIUS server(s), method lists.
4. VLANs/SVIs:
   - **Central switching**: VLAN/SVI on WLC or upstream L3.
   - **Flex**: AP’s switchport must trunk client VLANs locally, DHCP must exist locally.

---

## Build Order
1. WLAN profile(s)
2. Policy profile(s)
3. Policy tag
4. RF profiles and RF tag
5. (Optional) AP Join profile
6. Site tag (with Flex profile for local switching)
7. Assign tags to APs

---

## Central-Switched SSID (Step-by-Step)
**Purpose**: Traffic tunnels to WLC.

1. **WLAN Profile**
   - SSID, WLAN ID, Security.
2. **Policy Profile**
   - Central Switching: ON
   - VLAN: Client VLAN
3. **Policy Tag**
   - Map WLAN → Policy Profile
4. **RF**
   - Create/edit RF profiles; create RF tag.
5. **Site Tag**
   - Use default or site-specific without Flex.
6. **Tag APs**
   - Assign Policy Tag, RF Tag, Site Tag.

**Validation:**
```
show wlan summary
show wireless tag policy detail <tag>
show ap tag summary
show wireless client summary
```

---

## Flex (Local-Switched) SSID (Step-by-Step)
**Purpose**: Traffic breaks out locally at branch.

1. **WLAN Profile**
2. **Policy Profile**
   - Central Switching: OFF
   - VLAN: Branch VLAN
   - Auth: Central or Local
3. **Flex Profile**
   - Add local RADIUS (if local auth), VLAN/ACL mappings.
4. **Policy Tag**
5. **RF**
6. **Site Tag**
   - Must reference Flex profile.
7. **Switchport Config**
   - Trunk client VLANs to AP.
   - Local DHCP for VLANs.
8. **Tag APs**

**Validation:**
```
show ap tag summary
show wireless client summary
```

---

## CLI Skeletons

**Create WLAN:**
```
conf t
wlan <WLAN_NAME> <WLAN_ID> <SSID>
  no shutdown
  security wpa psk set-key ascii 0 <your_psk>
exit
```

**Policy Profile (Central Switching):**
```
wireless profile policy <POL_CENTRAL>
  vlan <VLAN_ID>
  dhcp required
exit
```

**Policy Profile (Flex Local Switching):**
```
wireless profile policy <POL_FLEX>
  no central switching
  vlan <BRANCH_VLAN_ID>
  central authentication
exit
```

**Flex + Site Tag:**
```
wireless profile flex <FLEX_SITE_A>
exit

wireless tag site <SITE_A>
  flex-profile <FLEX_SITE_A>
exit
```

**Policy Tag:**
```
wireless tag policy <PT_BRANCH>
  wlan <WLAN_NAME> policy <POL_FLEX>
exit
```

**Assign Tags to AP:**
```
ap name <AP_NAME> policy-tag <PT_BRANCH>
ap name <AP_NAME> site-tag <SITE_A>
ap name <AP_NAME> rf-tag <RT_BRANCH>
```

---

## Checks
```
show wlan summary
show wireless tag policy detail <PT_BRANCH>
show wireless tag site detail <SITE_A>
show ap tag summary
show wireless client summary
```

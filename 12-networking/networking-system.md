# Phase 9: Networking/Multiplayer System — IGI 2: Covert Strike

**Status:** ✅ REVERSED  
**Functions Documented:** 50+  
**Confidence:** HIGH

---

## Architecture Overview

**Protocol:** UDP-based client/server  
**Transport:** Raw Winsock 2 (non-blocking sockets, FIONBIO)  
**Game Traffic:** UDP packets with guaranteed/non-guaranteed delivery  
**Remote Console:** TCP telnet (rcon)  
**Server Discovery:** LAN broadcast + manual connection  
**No DirectPlay:** Despite dpnhpast.dll import, game uses custom netcode

---

## Network Initialization

### Socket Layer
| Function | Address | Purpose |
|----------|---------|---------|
| Network_StartSockLib | 0x0069bbac | Winsock startup (WSAStartup) |
| Network_CloseSockLib | 0x0069bb74 | Winsock cleanup |
| Network_StartServer | 0x0069be30 | Create server UDP socket + bind |
| Network_StartClient | 0x0069bc18 | Non-blocking client socket |
| Network_ClientBindGlobalUDPSocket | 0x0069c800 | Client UDP bind |
| Network_StoreLocalAddress | 0x0069c868 | Local address via getsockname |

### Error Messages (WSAGetLastError mapping at 0x0069ca00)
- WSASYSNOTREADY, WSAECONNREFUSED, WSAETIMEDOUT, WSAECONNRESET, etc.

### Connection Flow
- `Sending connect request to server, password '%s'...` at 0x0069ba18
- `ClientConnect: Got no response, pausing and trying again...` at 0x006a43a0
- Connection succeeded: `got networkID=%d` at 0x0069c23c

### Connection Failures
| Reason | Address |
|--------|---------|
| Server out of IDs | 0x0069c258 |
| WRONG_MAP | 0x0069c2bc |
| SERVERFULL | 0x0069c2e4 |
| BANNED | 0x0069c30c |
| SOCKET BUSY | 0x0069c330 |
| PASSWORD | 0x0069c35c |

### Connection Responses
| Response | Address |
|----------|---------|
| NETWORKCONNECTIONRESPONSE_FAILED | 0x0069bf70 |
| NETWORKCONNECTIONRESPONSE_ALREADY_UP | 0x0069bfa4 |
| NETWORKCONNECTIONRESPONSE_SUCCESS | 0x0069c07c |

---

## Core Network Functions

### NetManager (0x006a25f0-0x006a3024)

| Function | Purpose |
|----------|---------|
| NetManager_RunNetworkInterrupt | Main network update/tick |
| NetManager_GetActiveSession | Get active game session |
| NetManager_SetActiveSession | Set active session |
| NetManager_SetMode | Set game mode |
| NetManager_SetLocalBandwidth | Bandwidth cap negotiation |
| NetManager_RequestTeamChange | Team switching |
| NetManager_SetFace/SetModel | Character customization |
| NetManager_GotoMap | Map travel |
| NetManager_ListMaps | Map listing |
| NetManager_VoteMap | Map voting |
| BanPlayer | Server admin: ban player |
| KickPlayer | Server admin: kick player |
| VoteKick | Community vote kick |

---

## Network Modes (NETMANAGER_MODE_*)

All at 0x006a2768-0x006a281c:

| Mode | Address | Description |
|------|---------|-------------|
| MODE_DEAD | 0x006a2768 | Dead/spectating |
| MODE_PLAY | 0x006a2780 | Active gameplay |
| MODE_SPAWN | 0x006a2798 | Spawn selection phase |
| MODE_SHOP | 0x006a27b0 | Buy/equipment phase |
| MODE_SPECTATE | 0x006a27c8 | Spectator |
| MODE_SELECTSKIN | 0x006a27e4 | Character customization |
| MODE_SELECTTEAM | 0x006a2800 | Team selection |

---

## Network Messages (GM Types)

All at 0x006a2dac-0x006a3008:

| GM ID | String Address | Purpose |
|-------|---------------|---------|
| GMAuthenticate | 0x006a2dac | Player authentication |
| GMPing | 0x006a2e20 | Latency test |
| GMPong | 0x006a2e0c | Ping response |
| GMChat | 0x006a2e60 | Chat messages |
| GMStats | 0x006a2e34 | Player statistics |
| GMSettings | 0x006a2e48 | Game settings sync |
| GMTeamScore | 0x006a2e74 | Team score updates |
| GMCreatePlayer | 0x006a3008 | Player join/spawn |
| GMDestroyPlayer | 0x006a2fec | Player disconnect/kill |
| GMSpawn | 0x006a2f64 | Spawn confirmation |
| GMRequestSpawn | 0x006a2f48 | Spawn request |
| GMSpectate | 0x006a2f30 | Spectator mode |
| GMDrop | 0x006a2eec | Weapon drop |
| GMRequestDrop | 0x006a2ed0 | Drop request |
| GMPickup | 0x006a2f1c | Item pickup |
| GMRequestPickup | 0x006a2f00 | Pickup request |
| GMEquip | 0x006a2e8c | Equipment give |
| GMDamage | 0x006a2f8c | Damage dealt |
| GMKill | 0x006a2f78 | Kill notification |
| GMObjective | 0x006a2ea0 | Objective updates |
| GMActivate | 0x006a2eb8 | Activate object |
| GMInitGameSettings | 0x006a2fcc | Settings initialization |
| GMUpdate | 0x006a2fb8 | General state update |
| GMZUpdate | 0x006a2fa0 | Z-position update |
| GMBomb | 0x006a2de4 | C4/bomb state |
| GMGrenadeUpdate | 0x006a2dc8 | Grenade sync |
| GMList | 0x006a2df8 | Entity/list sync |

---

## Packet Delivery System

### Guaranteed vs Non-Guaranteed
| Message | Description |
|---------|-------------|
| Network_AddPacketToQueue | Add to send queue |
| Network_HandleSendQueues | Process pending sends |
| Network_FreeGuaranteedQueue | Remove guaranteed packet |
| Network_FreeQueues | Free queue entry |

### Packet Loss Simulation
- `NO ADDED PACKET LOSS ON RECEIVE` at 0x0069c5d8
- `ADDED PACKET LOSS ON SEND` at 0x0069c620
- `Network_Send: DROPPING DATA. select():` at 0x0069c540

### Send/Receive Operations
| Operation | Description |
|-----------|-------------|
| Network_Send():sendto | UDP send |
| Network_receive:recvfrom | UDP receive |
| Network_Receive: select() | Non-blocking recv |
| Network_receive: Slow recvfrom() | Latency measurement |

---

## Server Configuration System (0x0069f7dc-0x006a07a8)

### Quality of Service
| Setting | Get | Set | Description |
|---------|-----|-----|-------------|
| pingMax | 0x09f874 | 0x09f88c | Max allowed ping (ms) |
| plossMax | 0x09f83c | 0x09f858 | Packet loss threshold |
| packetLoss | 0x09f8dc | 0x09f8f8 | Current packet loss |
| timeOut | 0x09f914 | 0x09f92c | Connection timeout |
| autoKick | 0x09f8a4 | 0x09f8c0 | Auto-kick enabled |
| choke | 0x09f9e8 | 0x09fa00 | Bandwidth choke |
| fillPercent | 0x09fa18 | 0x09fa34 | Fill percentage |
| smoothPercent | 0x09fa50 | 0x09fa70 | Smooth percentage |

### Gameplay
| Setting | Get | Set | Description |
|---------|-----|-----|-------------|
| allowSniperRifles | 0x09fb08 | 0x09fb2c | Sniper toggle |
| spawnSafeTime | 0x09fb50 | 0x09fb70 | Spawn protection (s) |
| spawnTimeToFree | 0x09fb90 | 0x09fbb0 | Spawn lock time |
| spawnMaxCost | 0x09fbd0 | 0x09fbf0 | Max spawn cost |
| bombReposTime | 0x09fc10 | 0x09fc30 | Bomb reposition |
| bombTime | 0x09fc50 | 0x09fc6c | Bomb defuse time |
| objectiveMaxTime | 0x09fc88 | 0x09fcac | Objective time limit |
| mapMaxTeamScore | 0x09fd10 | 0x09fcf0 | Team score win |
| mapMaxTime | 0x09fd10 | 0x09fd2c | Round time limit |
| mapMaxRounds | 0x09fd48 | 0x09fd68 | Max rounds |

### Economy
| Setting | Get | Set | Description |
|---------|-----|-----|-------------|
| moneyMissionLost | 0x09fd88 | 0x09fdac | Money on loss |
| moneyMissionWin | 0x09fdd0 | 0x09fdf0 | Money on win |
| moneyObjectiveTeamLost | 0x09fe10 | 0x09fe38 | Objective loss penalty |
| moneyObjectiveTeamWin | 0x09fe60 | 0x09fe88 | Objective win reward |
| moneyTeamKill | 0x09ff00 | 0x09ff20 | Teamkill penalty |
| moneyKill | 0x09ff40 | 0x09ff5c | Kill reward |
| teamDamage | 0x0a00c0 | 0x0a00dc | Team damage toggle |
| spectateMode | 0x0a016c | 0x0a018c | Spectator rules |

### Security
| Setting | Get | Set | Description |
|---------|-----|-----|-------------|
| password | 0x0a02bc | 0x0a02a0 | Server password |
| svpassword | 0x0a093c | — | Super user rcon password |
| bandwidth | 0x0a03bc | — | Bandwidth cap |
| pingmax | 0x0a0358 | — | Ping max config |

---

## Client/Server State Objects

| Address | Object | Purpose |
|---------|--------|---------|
| 0x0068d2d0 | NetworkClient | Client state |
| 0x0069a418 | NetworkServer | Server state |
| 0x0069be88 | Network_ServerHandleReceivedPacket | Server packet router |
| 0x006a4890 | NetManager_ClientHandler | Unknown message handler |

---

## Key Addresses

| Address | Significance |
|---------|-------------|
| 0x0069bbac | Network_StartSockLib |
| 0x0069be30 | Network_StartServer |
| 0x0069c800 | Network_ClientBindGlobalUDPSocket |
| 0x006a25f0 | NetManager_RunNetworkInterrupt |
| 0x006a2dac | GM message strings |
| 0x006a2768 | Game modes |
| 0x0069f7dc | Server config |

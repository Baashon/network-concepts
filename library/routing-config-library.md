# Библиотека настроек: Cisco / Huawei / Juniper / MikroTik / Zyxel

> Синтаксис зависит от версии ПО и модели. Перед применением на рабочем оборудовании сверяйтесь с документацией.

## Легенда обозначений

| Плейсхолдер | Значение |
|---|---|
| <if> | имя интерфейса (напр. GigabitEthernet0/0/1, ge-0/0/1) |
| <net>/<mask> | адрес сети и маска |
| <nh> | next-hop адрес |
| <AS> | номер автономной системы |
| <rid> | router-id |

---

## 1. Статическая маршрутизация

### Cisco IOS / IOS-XE

```
ip route <net> <mask> <nh>
ip route 0.0.0.0 0.0.0.0 <nh>                 ! маршрут по умолчанию
ip route 10.10.0.0 255.255.255.0 192.168.1.1 5   ! с AD (административной дистанцией)
ip route 10.10.0.0 255.255.255.0 <if>         ! через выходной интерфейс

! Проверка
show ip route static
show ip route 10.10.0.0
```

Пример вывода `show ip route static`:

```
S    10.10.0.0/24 [1/0] via 192.168.1.1
S*   0.0.0.0/0 [1/0] via 192.168.1.1
```

### Huawei VRP

```
ip route-static <net> <mask> <nh>
ip route-static 0.0.0.0 0.0.0.0 <nh>
ip route-static 10.10.0.0 255.255.255.0 192.168.1.1 preference 60
ip route-static 10.10.0.0 255.255.255.0 <if> <nh>

# Проверка
display ip routing-table protocol static
display ip routing-table 10.10.0.0
```

Пример вывода `display ip routing-table protocol static`:

```
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Public routing table : Static
         Destinations : 1        Routes : 1
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
    10.10.0.0/24    Static  60   0         D     192.168.1.1     GigabitEthernet0/0/1
```

### Juniper JunOS

```
set routing-options static route <net>/<prefix> next-hop <nh>
set routing-options static route 0.0.0.0/0 next-hop <nh>
set routing-options static route 10.10.0.0/24 next-hop 192.168.1.1 preference 5
set routing-options static route 10.10.0.0/24 discard   ! null-route

commit

# Проверка
show route protocol static
show route 10.10.0.0/24
```

Пример вывода `show route protocol static`:

```
inet.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.0.0/24       *[Static/5] 00:02:14
                    > to 192.168.1.1 via ge-0/0/1.0
```

### MikroTik RouterOS

```
/ip route add dst-address=10.10.0.0/24 gateway=192.168.1.1
/ip route add dst-address=0.0.0.0/0 gateway=192.168.1.1              ; маршрут по умолчанию
/ip route add dst-address=10.10.0.0/24 gateway=192.168.1.1 distance=5   ; с AD (аналог administrative distance)
/ip route add dst-address=10.10.0.0/24 gateway=ether1 type=blackhole      ! null-route

; Проверка
/ip route print
/ip route print where dst-address~"10.10.0.0"
```

Пример вывода `/ip route print`:

```
Flags: D - DYNAMIC; A - ACTIVE; c - CONNECT, s - STATIC
 #      DST-ADDRESS        GATEWAY          DISTANCE
 0 As   10.10.0.0/24       192.168.1.1      1
```

### Zyxel (управляемые коммутаторы серий GS/XGS — CLI близок к Cisco IOS)

```
ip route 10.10.0.0 255.255.255.0 192.168.1.1

! Проверка
show ip route
```

Пример вывода `show ip route` (общая форма, у Zyxel сильно варьируется по модели):

```
Destination      Gateway        Interface   Metric
10.10.0.0/24      192.168.1.1    vlan1       1
```

> На L2-коммутаторах Zyxel статическая маршрутизация обычно недоступна вовсе (только L3-модели вроде XGS4600/GS3700). На security-шлюзах (USG/ZyWALL/ATP) синтаксис другой и завязан на объекты зон/интерфейсов — сверяйтесь с CLI reference конкретной модели.

**Заметка:** дефолтная AD/preference для static — Cisco: 1, Huawei: 60, Juniper: 5, MikroTik: 1 (по умолчанию, настраивается через `distance`). Учитывайте при миграции конфигов между вендорами.

---

## 2. OSPF

### Cisco IOS / IOS-XE

```
router ospf 1
 router-id <rid>
 network 10.10.0.0 0.0.0.255 area 0
 default-information originate
!
interface <if>
 ip ospf 1 area 0
 ip ospf cost 10
 ip ospf priority 1
 ip ospf hello-interval 10
 ip ospf dead-interval 40
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 <key>

! Проверка
show ip ospf neighbor
show ip ospf interface brief
show ip protocols
show ip route ospf
```

Пример вывода `show ip ospf neighbor`:

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:38    10.0.0.2        GigabitEthernet0/0/1
```

### Huawei VRP

```
ospf 1 router-id <rid>
 area 0.0.0.0
  network 10.10.0.0 0.0.0.255
  default-route-advertise
#
interface <if>
 ospf cost 10
 ospf dr-priority 1
 ospf timer hello 10
 ospf timer dead 40
 ospf authentication-mode md5 1 cipher <key>

# Проверка
display ospf peer
display ospf interface
display ospf routing-table
display ip routing-table protocol ospf
```

Пример вывода `display ospf peer`:

```
 Area 0.0.0.0 interface 10.0.0.1(GigabitEthernet0/0/1)'s neighbors
 Router ID: 2.2.2.2      Address: 10.0.0.2
   State: Full  Mode: Nbr is Master  Priority: 1
   DR: 10.0.0.1  BDR: 10.0.0.2  MTU: 0
```

### Juniper JunOS

```
set routing-options router-id <rid>
set protocols ospf area 0.0.0.0 interface <if>
set protocols ospf area 0.0.0.0 interface <if> metric 10
set protocols ospf area 0.0.0.0 interface <if> priority 1
set protocols ospf area 0.0.0.0 interface <if> hello-interval 10
set protocols ospf area 0.0.0.0 interface <if> dead-interval 40
set protocols ospf area 0.0.0.0 interface <if> authentication md5 1 key <key>
set protocols ospf export DEFAULT-ORIGINATE

commit

# Проверка
show ospf neighbor
show ospf interface
show route protocol ospf
```

Пример вывода `show ospf neighbor`:

```
Address          Interface              State     ID               Pri  Dead
10.0.0.2         ge-0/0/1.0             Full      2.2.2.2            1    39
```

### MikroTik RouterOS (v7; в v6 синтаксис OSPF заметно отличается — instance/network вместо area/interface-template)

```
/routing ospf instance add name=default router-id=1.1.1.1
/routing ospf area add name=backbone instance=default area-id=0.0.0.0
/routing ospf interface-template add interfaces=ether1 area=backbone cost=10 priority=1 hello-interval=10s dead-interval=40s

; Проверка
/routing ospf neighbor print
/routing ospf lsa print
/ip route print where ospf
```

Пример вывода `/routing ospf neighbor print`:

```
 # INSTANCE  ROUTER-ID  ADDRESS    STATE  ROLE      DR-ADDRESS
 0 default   2.2.2.2    10.0.0.2   Full   Backup    10.0.0.1
```

> Zyxel: OSPF есть только на некоторых L3-моделях (напр. XGS4600) и на части security-шлюзов — на большинстве коммутаторов Zyxel поддерживается только static/RIP. Проверяйте наличие функции в datasheet конкретной модели перед тем, как закладывать её в проект.

**Заметка:** типы area (stub/NSSA/totally-stubby) настраиваются похоже по смыслу, но разным синтаксисом:

- Cisco: `area 1 stub` / `area 1 nssa`

- Huawei: `area 0.0.0.1` → `stub` / `nssa`

- Juniper: `set protocols ospf area 0.0.0.1 stub` / `nssa`

- MikroTik: `/routing ospf area add ... type=stub` / `type=nssa`

---

## 3. IS-IS

### Cisco IOS / IOS-XE

```
router isis <tag>
 net 49.0001.0000.0000.0001.00
 is-type level-2-only            ! или level-1, level-1-2
 metric-style wide

interface <if>
 ip router isis <tag>
 isis metric 10
 isis circuit-type level-2-only
 isis hello-interval 10
 isis authentication mode md5
 isis authentication key-chain <key-chain>

! Проверка
show isis neighbors
show isis database
show ip route isis
show clns interface <if>
```

Пример вывода `show isis neighbors`:

```
System Id      Type Interface     IP Address      State Holdtime Circuit Id
R2             L2   Gi0/1         10.0.0.2        UP    24       R2.01
```

### Huawei VRP

```
isis 1
 is-level level-2
 network-entity 49.0001.0000.0000.0001.00
 cost-style wide

interface <if>
 isis enable 1
 isis cost 10
 isis circuit-type p2p
 isis timer hello 10
 isis authentication-mode md5 cipher <key>

# Проверка
display isis peer
display isis lsdb
display isis route
```

Пример вывода `display isis peer`:

```
 System Id: 0000.0000.0002  Interface: GE0/0/1
     Circuit Id: 0000.0000.0002.01  State: Up  HoldTime: 24s Type: L2
```

### Juniper JunOS

```
set interfaces lo0 unit 0 family iso address 49.0001.0000.0000.0001.00
set protocols isis level 2 wide-metrics-only
set protocols isis interface <if> level 2 metric 10
set protocols isis interface <if> point-to-point
set protocols isis interface <if> hello-authentication-type md5
set protocols isis interface <if> hello-authentication-key <key>

commit

# Проверка
show isis adjacency
show isis database
show isis interface
show route protocol isis
```

Пример вывода `show isis adjacency`:

```
Interface             System         L State        Hold (secs) SNPA
ge-0/0/1.0             R2             2  Up                   24
```

**Заметка:**

- **NET-адрес** (`49.0001.0000.0000.0001.00`) — аналог router-id для IS-IS: AFI (49 = частный) + area-id + system-id (обычно из MAC/loopback) + SEL (00).

- **Level 1** — маршрутизация внутри area (аналог OSPF stub-area), **Level 2** — магистраль между area (аналог backbone area 0). У Cisco/Huawei есть режим `level-1-2` (пограничный маршрутизатор), у Juniper по умолчанию интерфейс участвует и в L1, и в L2 — ограничивается явным указанием `level 1 disable` или `level 2 disable`.

- IS-IS работает поверх CLNS, а не IP напрямую — отсюда `net`/`network-entity`/`family iso` вместо привычного `network`/router-id по IP.

---

## 4. BGP (eBGP / iBGP)

### Cisco IOS / IOS-XE

```
router bgp <AS>
 bgp router-id <rid>
 neighbor 192.168.1.2 remote-as 65002
 neighbor 192.168.1.2 description ISP-Uplink
 neighbor 192.168.1.2 update-source Loopback0
 network 10.10.0.0 mask 255.255.255.0
 neighbor 192.168.1.2 route-map SET-IN in
 neighbor 192.168.1.2 route-map SET-OUT out
 neighbor 192.168.1.2 soft-reconfiguration inbound

! Проверка
show ip bgp summary
show ip bgp neighbors 192.168.1.2
show ip bgp
clear ip bgp 192.168.1.2 soft
```

Пример вывода `show ip bgp summary`:

```
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.1.2     4 65002    1204    1198        15    0    0 01:12:45        3
```

### Huawei VRP

```
bgp <AS>
 router-id <rid>
 peer 192.168.1.2 as-number 65002
 peer 192.168.1.2 description ISP-Uplink
 peer 192.168.1.2 connect-interface LoopBack0
 #
 ipv4-family unicast
  network 10.10.0.0 255.255.255.0
  peer 192.168.1.2 enable
  peer 192.168.1.2 route-policy SET-IN import
  peer 192.168.1.2 route-policy SET-OUT export

# Проверка
display bgp peer
display bgp peer 192.168.1.2 verbose
display bgp routing-table
reset bgp 192.168.1.2
```

Пример вывода `display bgp peer`:

```
 Peer            V    AS  MsgRcvd  MsgSent  OutQ  Up/Down       State        PrefRcv
 192.168.1.2     4 65002     1204     1198     0  01:12:45     Established         3
```

### Juniper JunOS

```
set routing-options autonomous-system <AS>
set routing-options router-id <rid>
set protocols bgp group EXT type external
set protocols bgp group EXT peer-as 65002
set protocols bgp group EXT neighbor 192.168.1.2
set protocols bgp group EXT description "ISP-Uplink"
set protocols bgp group EXT import IMPORT-POLICY
set protocols bgp group EXT export EXPORT-POLICY

set policy-options policy-statement EXPORT-POLICY term 1 from route-filter 10.10.0.0/24 exact
set policy-options policy-statement EXPORT-POLICY term 1 then accept

commit

# Проверка
show bgp summary
show bgp neighbor 192.168.1.2
show route protocol bgp
clear bgp neighbor 192.168.1.2 soft
```

Пример вывода `show bgp summary`:

```
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
192.168.1.2          65002       1204       1198       0       0     1:12:45 3/3/3/0              0/0/0/0
```

**Заметка:** на Cisco/Huawei политики (`route-map` / `route-policy`) задаются отдельными блоками с `match`/`permit-deny`; на Juniper — через `policy-options policy-statement` с `term`. Логика похожа, но термины разные.

### eBGP vs iBGP — ключевые отличия в конфигурации

**eBGP** (соседи в разных AS, обычно directly-connected):

Cisco:

```
router bgp 65001
 neighbor 192.168.1.2 remote-as 65002      ! AS соседа отличается от локального → eBGP
 neighbor 192.168.1.2 ebgp-multihop 2       ! если сосед не directly-connected
```

Huawei:

```
bgp 65001
 peer 192.168.1.2 as-number 65002
 peer 192.168.1.2 ebgp-max-hop 2
```

Juniper:

```
set protocols bgp group EXT type external
set protocols bgp group EXT multihop ttl 2
```

**iBGP** (соседи в одном AS, обычно через loopback + IGP):

Cisco:

```
router bgp 65001
 neighbor 10.0.0.2 remote-as 65001          ! тот же AS → iBGP
 neighbor 10.0.0.2 update-source Loopback0
 neighbor 10.0.0.2 next-hop-self             ! чтобы next-hop не терялся при переанонсе eBGP-маршрутов внутрь AS
```

Huawei:

```
bgp 65001
 peer 10.0.0.2 as-number 65001
 peer 10.0.0.2 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 10.0.0.2 enable
  peer 10.0.0.2 next-hop-local
```

Juniper:

```
set protocols bgp group INT type internal
set protocols bgp group INT local-address <local-loopback-ip>
set protocols bgp group INT neighbor 10.0.0.2
set protocols bgp group INT export SET-NEXT-HOP-SELF
```

MikroTik RouterOS (v7 — единый синтаксис для eBGP и iBGP, роль задаётся через `local.role`):

```
/routing bgp template add name=default as=65001 router-id=1.1.1.1

; eBGP
/routing bgp connection add name=isp-peer remote.address=192.168.1.2/32 remote.as=65002 local.role=ebgp templates=default

; iBGP
/routing bgp connection add name=core-peer remote.address=10.0.0.2/32 remote.as=65001 local.role=ibgp local.address=<local-loopback-ip> templates=default nexthop-choice=force-self

; Проверка
/routing bgp session print
```

Пример вывода `/routing bgp session print`:

```
 0 name="isp-peer" remote.address=192.168.1.2/32 remote.as=65002 established=yes uptime=1h12m45s prefix-count=3
```

> Zyxel: полноценный BGP практически отсутствует в линейке коммутаторов и большинства security-шлюзов Zyxel — на этом вендоре BGP не рассматривайте как рабочий вариант, для внешней маршрутизации используются static/RIP/OSPF там, где они вообще поддерживаются.

**Важно:**

- iBGP требует полносвязности (full-mesh) либо route-reflector/confederation — обычные iBGP-маршруты не переанонсируются другому iBGP-соседу без одного из этих механизмов.

- iBGP-соседи обычно поднимаются между loopback-адресами, которые должны быть доступны через IGP (OSPF или IS-IS, см. разделы 2–3) — без этого сессия не установится.

- `next-hop-self` (Cisco/Juniper через export-policy) / `next-hop-local` (Huawei) / `nexthop-choice=force-self` (MikroTik) обязателен на границе AS, чтобы внутренние iBGP-соседи видели корректный next-hop для маршрутов, полученных от eBGP.

---

## 5. Политики BGP (prefix-list, route-map, атрибуты)

### Cisco IOS / IOS-XE

```
! Prefix-list — фильтрация по префиксам
ip prefix-list CUST-IN seq 10 permit 10.10.0.0/16 le 24
ip prefix-list CUST-IN seq 20 deny 0.0.0.0/0 le 32

! Route-map — управление атрибутами
route-map SET-IN permit 10
 match ip address prefix-list CUST-IN
 set local-preference 150
 set community 65001:100 additive

route-map SET-OUT permit 10
 match ip address prefix-list CUST-IN
 set as-path prepend 65001 65001
 set metric 50

! Применение к соседу
router bgp 65001
 neighbor 192.168.1.2 route-map SET-IN in
 neighbor 192.168.1.2 route-map SET-OUT out
 neighbor 192.168.1.2 prefix-list CUST-IN in     ! альтернатива — чистая фильтрация без route-map

! Проверка
show route-map SET-IN
show ip bgp neighbors 192.168.1.2 advertised-routes
show ip bgp neighbors 192.168.1.2 received-routes
```

Пример вывода `show route-map SET-IN`:

```
route-map SET-IN, permit, sequence 10
  Match clauses:
    ip address prefix-lists: CUST-IN
  Set clauses:
    local-preference 150
    community 65001:100 additive
```

### Huawei VRP

```
# ip-prefix — фильтрация по префиксам
ip ip-prefix CUST-IN index 10 permit 10.10.0.0 16 greater-equal 16 less-equal 24
ip ip-prefix CUST-IN index 20 deny 0.0.0.0 0 less-equal 32

# route-policy — управление атрибутами
route-policy SET-IN permit node 10
 if-match ip-prefix CUST-IN
 apply local-preference 150
 apply community 65001:100 additive

route-policy SET-OUT permit node 10
 if-match ip-prefix CUST-IN
 apply as-path 65001 65001 additive
 apply cost 50

# Применение к соседу
bgp 65001
 peer 192.168.1.2 as-number 65002
 #
 ipv4-family unicast
  peer 192.168.1.2 route-policy SET-IN import
  peer 192.168.1.2 route-policy SET-OUT export
  peer 192.168.1.2 ip-prefix CUST-IN import       ! альтернатива — чистая фильтрация без route-policy

# Проверка
display route-policy SET-IN
display bgp routing-table peer 192.168.1.2 advertised-routes
display bgp routing-table peer 192.168.1.2 received-routes
```

Пример вывода `display route-policy SET-IN`:

```
Route-policy : SET-IN
 permit : 10
  if-match ip-prefix CUST-IN
  apply local-preference 150
  apply community 65001:100 additive
```

### Juniper JunOS

```
set policy-options prefix-list CUST-IN 10.10.0.0/16

set policy-options community CUST-COMM members 65001:100

set policy-options policy-statement SET-IN term 1 from prefix-list CUST-IN
set policy-options policy-statement SET-IN term 1 then local-preference 150
set policy-options policy-statement SET-IN term 1 then community add CUST-COMM
set policy-options policy-statement SET-IN term 1 then accept

set policy-options policy-statement SET-OUT term 1 from prefix-list CUST-IN
set policy-options policy-statement SET-OUT term 1 then as-path-prepend "65001 65001"
set policy-options policy-statement SET-OUT term 1 then metric 50
set policy-options policy-statement SET-OUT term 1 then accept

set protocols bgp group EXT import SET-IN
set protocols bgp group EXT export SET-OUT

commit

# Проверка
show policy SET-IN
show route advertising-protocol bgp 192.168.1.2
show route receive-protocol bgp 192.168.1.2
```

Пример вывода `show policy SET-IN`:

```
Policy SET-IN:
    Term 1:
        from prefix-list CUST-IN
        then
            local-preference 150
            community add CUST-COMM
            accept
```

### Основные атрибуты и как их задать

| Атрибут | Влияет на | Cisco | Huawei | Juniper |
|---|---|---|---|---|
| Local Preference | исходящий трафик (внутри своей AS); больше — предпочтительнее | set local-preference <N> | apply local-preference <N> | then local-preference <N> |
| MED (metric) | входящий трафик от соседней AS; меньше — предпочтительнее | set metric <N> | apply cost <N> | then metric <N> |
| AS-Path Prepend | делает маршрут менее привлекательным для внешних соседей | set as-path prepend <AS> ... | apply as-path <AS> ... additive | then as-path-prepend "<AS> ..." |
| Community | тегирование маршрута для последующей фильтрации/политик у себя или у соседей | set community <AS:NN> additive | apply community <AS:NN> additive | then community add <name> |
| Weight | локальное предпочтение только на этом роутере, соседям не передаётся | set weight <N> | — (аналога нет) | — (аналога нет) |

**Порядок выбора лучшего маршрута BGP (общий для всех вендоров, по убыванию приоритета):** Weight (только Cisco) → Local Preference → локально созданный маршрут → короче AS-Path → ниже Origin (IGP < EGP < Incomplete) → ниже MED → eBGP предпочтительнее iBGP → ниже метрика IGP до next-hop → старше маршрут (дольше в таблице) → ниже router-id.

---

## 6. BGP Blackhole (RTBH — Remote Triggered Black Hole)

Механизм экстренного дропа трафика к конкретному адресу (обычно при волюметрической DDoS-атаке): жертвенный префикс анонсируется по BGP с next-hop, который на всех маршрутизаторах заранее указывает на Null/discard — в итоге трафик к этому адресу отбрасывается на входе в сеть (локальный RTBH) или ещё на стороне апстрима (удалённый RTBH через community).

### Схема работы

1. На **всех** граничных маршрутизаторах заранее прописывается статический discard-маршрут на «trigger»-адрес (например, `192.0.2.1/32`, из документационного диапазона TEST-NET, реально не используется).

2. При атаке на инжект-роутере создаётся статический маршрут на IP жертвы с тегом, редистрибутируется в BGP через route-map/policy с подменой next-hop на trigger-адрес.

3. Через iBGP этот маршрут долетает до всех edge-роутеров — у них next-hop уже резолвится в discard (п.1), трафик к жертве дропается везде одновременно.

### Локальный RTBH — Cisco IOS / IOS-XE

```
! 1. На всех граничных роутерах (один раз, заранее)
ip route 192.0.2.1 255.255.255.255 Null0

! 2. На инжект-роутере — на момент инцидента
ip route 203.0.113.55 255.255.255.255 Null0 tag 666

route-map BLACKHOLE permit 10
 match tag 666
 set ip next-hop 192.0.2.1
 set community no-export additive
 set origin igp

router bgp 65001
 redistribute static route-map BLACKHOLE

! Проверка
show ip route 203.0.113.55
show ip bgp 203.0.113.55/32

! Снятие блокировки — просто удалить статический маршрут
no ip route 203.0.113.55 255.255.255.255 Null0 tag 666
```

Пример вывода `show ip route 203.0.113.55` (после срабатывания RTBH):

```
S    203.0.113.55/32 [1/0] via 192.0.2.1, Null0
```

### Локальный RTBH — Huawei VRP

```
# 1. На всех граничных роутерах (один раз, заранее)
ip route-static 192.0.2.1 255.255.255.255 NULL0

# 2. На инжект-роутере — на момент инцидента
ip route-static 203.0.113.55 255.255.255.255 NULL0 tag 666

route-policy BLACKHOLE permit node 10
 if-match tag 666
 apply ip-address next-hop 192.0.2.1
 apply community no-export additive

bgp 65001
 import-route static route-policy BLACKHOLE

# Проверка
display ip routing-table 203.0.113.55
display bgp routing-table 203.0.113.55

# Снятие блокировки
undo ip route-static 203.0.113.55 255.255.255.255 NULL0 tag 666
```

### Локальный RTBH — Juniper JunOS

```
# 1. На всех граничных роутерах (один раз, заранее)
set routing-options static route 192.0.2.1/32 discard

# 2. На инжект-роутере — на момент инцидента
set routing-options static route 203.0.113.55/32 discard tag 666
set routing-options static route 203.0.113.55/32 discard no-readvertise

set policy-options policy-statement BLACKHOLE term 1 from tag 666
set policy-options policy-statement BLACKHOLE term 1 then next-hop 192.0.2.1
set policy-options policy-statement BLACKHOLE term 1 then community add NO-EXPORT
set policy-options policy-statement BLACKHOLE term 1 then accept

set protocols bgp group INT export BLACKHOLE

commit

# Проверка
show route 203.0.113.55/32
show route protocol bgp 203.0.113.55/32

# Снятие блокировки
delete routing-options static route 203.0.113.55/32
commit
```

### Удалённый RTBH — через апстрим-провайдера

Для крупных атак трафик лучше дропать ещё до входа в свою сеть — большинство операторов принимают от клиентов маршрут с общепринятым **well-known BLACKHOLE community `65535:666`** (RFC 7999) и сами null-routят его у себя на границе.

**Cisco:**

```
route-map TO-ISP-BLACKHOLE permit 10
 match tag 666
 set community 65535:666 additive     ! well-known BLACKHOLE community (RFC 7999)
 set community no-export additive        ! опционально — по договорённости с апстримом

router bgp 65001
 neighbor <isp-ip> route-map TO-ISP-BLACKHOLE out
 redistribute static route-map TO-ISP-BLACKHOLE
```

**Huawei:**

```
route-policy TO-ISP-BLACKHOLE permit node 10
 if-match tag 666
 apply community 65535:666 additive

bgp 65001
 peer <isp-ip> route-policy TO-ISP-BLACKHOLE export
```

**Juniper:**

```
set policy-options community BLACKHOLE-COMM members 65535:666
set policy-options policy-statement TO-ISP-BLACKHOLE term 1 from tag 666
set policy-options policy-statement TO-ISP-BLACKHOLE term 1 then community add BLACKHOLE-COMM
set policy-options policy-statement TO-ISP-BLACKHOLE term 1 then accept

set protocols bgp group EXT export TO-ISP-BLACKHOLE

commit
```

> Точное значение blackhole-community у конкретного апстрима нужно уточнять в его политике пиринга — `65535:666` общепринят, но не универсален для всех провайдеров.

**Важно:** blackhole полностью убивает доступность адреса-жертвы (трафик дропается целиком, включая легитимный) — это осознанный компромисс «положить один адрес» вместо «дать атаке забить весь аплинк/сеть». Используется как крайняя мера при объёмной DDoS-атаке, не как постоянная практика.

---

## 7. DDoS-защита (uRPF, BGP Flowspec)

RTBH (раздел 6) и rate-limiting/QoS (раздел 10) — тоже часть защиты от DDoS; здесь — два дополнительных механизма: uRPF (антиспуфинг источника) и BGP Flowspec (точечная фильтрация без потери доступности адреса целиком).

### uRPF — защита от подмены (спуфинга) source-адреса

Отбрасывает пакеты, чей source-адрес не может быть достигнут через тот интерфейс, на который пакет пришёл — базовая защита от спуфинга, который лежит в основе большинства амплификационных DDoS-атак (DNS/NTP/memcached reflection).

**Cisco:**

```
interface <if>
 ip verify unicast source reachable-via rx     ! strict mode — источник обязан быть достижим именно через этот интерфейс
 ip verify unicast source reachable-via any      ! loose mode — источник достижим через любой интерфейс (для asymmetric routing)

! Проверка
show cef interface <if>
show ip traffic | include RPF
```

Пример вывода `show ip traffic | include RPF`:

```
    0 unicast RPF failed drops
```

**Huawei:**

```
interface <if>
 urpf strict            ! strict mode
 urpf loose               ! loose mode (при асимметричной маршрутизации)

# Проверка
display urpf-statistics interface <if>
```

**Juniper:**

```
set interfaces <if> unit 0 family inet rpf-check                  ! strict mode (по умолчанию)
set interfaces <if> unit 0 family inet rpf-check mode loose         ! loose mode

commit

# Проверка
show interfaces <if> extensive | match RPF
```

**Заметка:** strict-режим требует симметричной маршрутизации (обратный путь совпадает с прямым) — на транзитных/multihomed сетях с asymmetric routing используйте loose, иначе легитимный трафик будет дропаться.

### BGP Flowspec (RFC 5575) — точечная фильтрация

В отличие от RTBH, Flowspec описывает конкретное правило по 5-tuple (src/dst IP, порты, протокол) и раздаёт его через BGP как action — discard, rate-limit, redirect, пометка DSCP — без блокировки всего трафика к адресу. Требует поддержки Flowspec на всех участвующих маршрутизаторах (адресное семейство `flow`/`flowspec` в BGP).

**Juniper (наиболее стандартизированная реализация):**

```
set routing-options flow term BLOCK-UDP-FLOOD from destination 203.0.113.55/32
set routing-options flow term BLOCK-UDP-FLOOD from protocol udp
set routing-options flow term BLOCK-UDP-FLOOD from destination-port 53
set routing-options flow term BLOCK-UDP-FLOOD then discard

set protocols bgp group INT family inet flow

commit

# Проверка
show route table inetflow.0
show route protocol bgp table inetflow.0
```

Пример вывода `show route table inetflow.0`:

```
inetflow.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
203.0.113.55,proto=17,port=53/term:1
                   *[Flow/5] 00:01:12
                      Fictitious
```

**Cisco и Huawei:** тоже поддерживают Flowspec (BGP address-family `ipv4 flowspec` / `ipv4-family flow`), но конкретный синтаксис локальных правил заметно различается между платформами и версиями ПО (особенно между IOS-XE и IOS-XR у Cisco) — включение самого address-family выглядит так:

```
! Cisco
router bgp 65001
 address-family ipv4 flowspec
  neighbor <ibgp-peer> activate

# Huawei
bgp 65001
 #
 ipv4-family flow
  peer <ibgp-peer> enable
```

Локальные правила фильтрации на этих платформах сверяйте с документацией конкретной модели/версии ПО.

### Краткий чек-лист по уровням защиты

| Механизм | Когда применять | Раздел |
|---|---|---|
| uRPF | постоянно на клиентских/граничных интерфейсах | 7 (этот раздел) |
| Rate-limiting / policing (QoS) | постоянно, ограничение паразитного трафика | 11 |
| BGP Flowspec | точечная атака, известен признак трафика (порт/протокол) | 7 (этот раздел) |
| RTBH (локальный) | атака бьёт по своей сети, признаки трафика неизвестны | 6 |
| RTBH через апстрим (community) | объёмная атака, забивает аплинк ещё до входа в сеть | 6 |

---

## 8. VLAN и интерфейсы

### Cisco IOS / IOS-XE

```
vlan 100
 name SUBSCRIBERS

! Access-порт
interface <if>
 switchport mode access
 switchport access vlan 100
 description Subscriber-Port
 no shutdown

! Trunk-порт
interface <if>
 switchport mode trunk
 switchport trunk allowed vlan 100,200,300
 switchport trunk native vlan 1

! L3-интерфейс VLAN (SVI)
interface Vlan100
 ip address 192.168.100.1 255.255.255.0
 no shutdown

! Sub-interface (routed, dot1Q)
interface <if>.100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.0

! Проверка
show vlan brief
show interfaces <if> switchport
show interfaces status
```

Пример вывода `show vlan brief`:

```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
100  SUBSCRIBERS                      active    Gi0/1, Gi0/2
```

### Huawei VRP

```
vlan batch 100 200 300
vlan 100
 description SUBSCRIBERS

# Access-порт
interface <if>
 port link-type access
 port default vlan 100
 description Subscriber-Port
 undo shutdown

# Trunk-порт
interface <if>
 port link-type trunk
 port trunk allow-pass vlan 100 200 300
 port trunk pvid vlan 1

# L3-интерфейс VLAN (VLANIF)
interface Vlanif100
 ip address 192.168.100.1 255.255.255.0

# Sub-interface (dot1q termination)
interface <if>.100
 vlan-type dot1q 100
 ip address 192.168.100.1 255.255.255.0

# Проверка
display vlan
display port vlan
display interface brief
```

Пример вывода `display vlan` (фрагмент):

```
VID  Type    Ports
100  common  UT:GE0/0/2(U)
              TG:GE0/0/1(U)
```

### Juniper JunOS

```
set vlans SUBSCRIBERS vlan-id 100

# Access-порт
set interfaces <if> unit 0 family ethernet-switching vlan members SUBSCRIBERS
set interfaces <if> unit 0 family ethernet-switching port-mode access

# Trunk-порт
set interfaces <if> unit 0 family ethernet-switching port-mode trunk
set interfaces <if> unit 0 family ethernet-switching vlan members [ 100 200 300 ]

# L3 (irb — routed VLAN interface)
set interfaces irb unit 100 family inet address 192.168.100.1/24
set vlans SUBSCRIBERS l3-interface irb.100

# Sub-interface (routed dot1q, стиль MX)
set interfaces <if> unit 100 vlan-id 100
set interfaces <if> unit 100 family inet address 192.168.100.1/24

commit

# Проверка
show vlans
show interfaces <if> extensive
show ethernet-switching interfaces
```

Пример вывода `show vlans`:

```
Routing instance        VLAN name           Tag       Interfaces
default-switch          SUBSCRIBERS         100
                                                        ge-0/0/2.0*
```

### MikroTik RouterOS (через bridge + VLAN filtering — общий подход и для L2, и для L3)

```
/interface bridge add name=bridge1 vlan-filtering=yes

; Access-порт
/interface bridge port add bridge=bridge1 interface=ether2 pvid=100
/interface bridge vlan add bridge=bridge1 vlan-ids=100 untagged=ether2 tagged=bridge1

; Trunk-порт
/interface bridge port add bridge=bridge1 interface=ether1
/interface bridge vlan add bridge=bridge1 vlan-ids=100,200,300 tagged=ether1,bridge1

; L3-интерфейс VLAN (routed)
/interface vlan add name=vlan100 vlan-id=100 interface=bridge1
/ip address add address=192.168.100.1/24 interface=vlan100

; Проверка
/interface bridge vlan print
/interface bridge port print
```

### Zyxel (управляемые коммутаторы GS/XGS — CLI близок к Cisco IOS)

```
vlan 100
 name SUBSCRIBERS

interface gigabitethernet1/0/2
 switchport mode access
 switchport access vlan 100

interface gigabitethernet1/0/1
 switchport mode trunk
 switchport trunk allowed vlan 100,200,300

! Проверка
show vlan
show interfaces status
```

> На security-шлюзах Zyxel (USG/ZyWALL/ATP) VLAN настраивается иначе — через объекты "interface > vlan" в CLI/GUI, привязанные к зонам; синтаксис существенно отличается от коммутаторной линейки.

**QinQ (двойное тегирование, актуально для агрегации абонентов):**

- Cisco: `switchport mode dot1q-tunnel` + `l2protocol-tunnel` на порту, внешний (S-VLAN) через `switchport access vlan <outer>`

- Huawei: `port link-type dot1q-tunnel` + `port default vlan <outer>`

- Juniper: `flexible-vlan-tagging` + `encapsulation flexible-ethernet-services`, теги через `vlan-tags outer <X> inner <Y>`

- MikroTik: `/interface vlan add ... use-service-tag=yes` поверх уже VLAN-инкапсулированного интерфейса (двойное вложение vlan-интерфейсов)

- Zyxel: QinQ доступен не на всех моделях — там, где есть, обычно `switchport mode dot1q-tunnel` (тот же принцип, что у Cisco); сверяйтесь с CLI reference модели.

---

## 9. Провижининг абонентских портов (PON / Ethernet)

### Ethernet-порт под абонента (QinQ, все три вендора)

**Cisco:**

```
interface <if>
 switchport mode dot1q-tunnel
 switchport access vlan 100          ! outer/S-VLAN
 l2protocol-tunnel
```

**Huawei:**

```
interface <if>
 port link-type dot1q-tunnel
 port default vlan 100               ! outer/S-VLAN
```

**Juniper:**

```
set interfaces <if> flexible-vlan-tagging
set interfaces <if> encapsulation flexible-ethernet-services
set interfaces <if> unit 0 vlan-tags outer 100 inner 200
set interfaces <if> unit 0 family bridge
```

**MikroTik RouterOS (QinQ):**

```
/interface vlan add name=svlan100 vlan-id=100 interface=ether1
/interface vlan add name=cvlan200 vlan-id=200 interface=svlan100 use-service-tag=yes
```

**MikroTik RouterOS — PPPoE-сервер (типовая схема терминации абонентов у ISP на MikroTik, аналог BRAS):**

```
/interface pppoe-server server add service-name=INTERNET interface=vlan100 authentication=pap,chap default-profile=CUST_A one-session-per-host=yes

/ppp profile add name=CUST_A local-address=10.10.0.1 remote-address=pppoe-pool rate-limit=20M/20M

/ppp secret add name=user1 password=<pass> service=pppoe profile=CUST_A

; Проверка
/ppp active print
/interface pppoe-server print
```

Пример вывода `/ppp active print`:

```
 # NAME    SERVICE  CALLER-ID          ADDRESS       UPTIME
 0 user1   pppoe    AA:BB:CC:DD:EE:FF  10.10.0.10    1h12m05s
```

### GPON — провижининг ONU на Huawei OLT (серии MA5600T / MA5800)

Huawei — основной вендор GPON OLT, встречающийся в типовых сетях операторов. Команды ниже — для актуальных версий ПО OLT; на старых прошивках MA5600T синтаксис местами отличается, сверяйтесь с документацией конкретной модели.

```
# Поиск незарегистрированных ONU на PON-порту
display ont autofind all

# Добавление ONU по серийному номеру
interface gpon 0/1
 ont add 0 1 sn-auth <SN> omci ont-lineprofile-id 1 ont-srvprofile-id 1 desc "Subscriber_XXX"

# Создание сервис-порта (привязка VLAN абонента к ONU/gemport)
service-port vlan 100 gpon 0/1/0 ont 1 gemport 1 multi-service user-vlan 100 tag-transform translate

# Native VLAN на Ethernet-порту ONU (для абонентского роутера)
ont port native-vlan 0/1 0 eth 1 vlan 100 priority 0

# Проверка состояния и уровня оптического сигнала
display ont info 0 1 0
display ont state-info 0 1 0
display ont optical-info 0 1 0    ! RX/TX power, важно при диагностике обрывов
```

Пример вывода `display ont autofind all`:

```
  F/S/P            :0/1/0
  Ont-ID           :-
  SN               :4857544301234567
  Ontauto find time:2026-08-19 09:15:03
```

Пример вывода `display ont info 0 1 0`:

```
  F/S/P            :0/1/0
  ONT-ID           :1
  Control flag     :active
  Run state        :online
  Config state     :normal
  Match state      :match
  Description      :Subscriber_XXX
```

> Cisco, Juniper, MikroTik и Zyxel не являются типовыми вендорами GPON OLT в массовых операторских сетях — для абонентского доступа через них обычно провижинится только Ethernet-порт/QinQ или PPPoE (см. выше), а PON-часть остаётся на Huawei/др. специализированном OLT-оборудовании. MikroTik и Zyxel при этом часто встречаются как раз в роли абонентского CPE/ONT-комбо или пограничного BRAS — именно в этой роли выше и показаны их конфиги.

---

## 10. Агрегация каналов (LACP)

### Cisco IOS / IOS-XE

```
interface Port-channel1
 description Uplink-LACP
 no shutdown

interface <if>
 channel-group 1 mode active
!
interface <if2>
 channel-group 1 mode active

! Trunk на агрегированном интерфейсе
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 100,200

! Проверка
show etherchannel summary
show lacp neighbor
show interfaces Port-channel1
```

Пример вывода `show etherchannel summary`:

```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Gi0/1(P)  Gi0/2(P)
```

### Huawei VRP

```
interface Eth-Trunk1
 description Uplink-LACP
 mode lacp-static
 trunkport <if>
 trunkport <if2>

# Trunk VLAN на Eth-Trunk
interface Eth-Trunk1
 port link-type trunk
 port trunk allow-pass vlan 100 200

# Проверка
display eth-trunk 1
display lacp statistics interface <if>
```

Пример вывода `display eth-trunk 1` (фрагмент):

```
Eth-Trunk1's state information is:
 Operate status: up       Number Of Up Port In Trunk: 2
 PortName              Status  PortType PortPri LACPTimeout ActorPortNo State
 GigabitEthernet0/0/1  Up      Actor    32768   Slow        1     ActivePartnerSync
 GigabitEthernet0/0/2  Up      Actor    32768   Slow        2     ActivePartnerSync
```

### Juniper JunOS

```
set chassis aggregated-devices ethernet device-count 1
set interfaces <if> ether-options 802.3ad ae0
set interfaces <if2> ether-options 802.3ad ae0
set interfaces ae0 aggregated-ether-options lacp active
set interfaces ae0 unit 0 family ethernet-switching port-mode trunk
set interfaces ae0 unit 0 family ethernet-switching vlan members [ 100 200 ]

commit

# Проверка
show interfaces ae0 extensive
show lacp interfaces ae0
```

Пример вывода `show lacp interfaces ae0` (фрагмент):

```
Aggregated interface: ae0
    LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
      ge-0/0/1        Actor    No    No   Yes  Yes  Yes   Yes    Fast    Active
      ge-0/0/1       Partner   No    No   Yes  Yes  Yes   Yes    Fast    Active
```

### MikroTik RouterOS

```
/interface bonding add name=bond1 slaves=ether1,ether2 mode=802.3ad lacp-rate=1sec transmit-hash-policy=layer-3-and-4

; Trunk VLAN поверх агрегированного интерфейса — через bridge (см. раздел 8)
/interface bridge port add bridge=bridge1 interface=bond1

; Проверка
/interface bonding print
/interface bonding monitor bond1 once
```

Пример вывода `/interface bonding monitor bond1 once`:

```
                    slaves: ether1,ether2
            active-slaves: ether1,ether2
               mii-status: link-ok
```

### Zyxel (управляемые коммутаторы GS/XGS)

```
interface port-channel 1
 description Uplink-LACP

interface gigabitethernet1/0/1
 port-channel-protocol lacp
 port-channel 1 mode active

interface gigabitethernet1/0/2
 port-channel-protocol lacp
 port-channel 1 mode active

! Проверка
show port-channel
show lacp
```

Пример вывода `show port-channel` (общая форма):

```
Port-channel1 is up
 Members: Gi1/0/1(P) Gi1/0/2(P)
```

> Точные ключевые слова (`port-channel-protocol`, `channel-group` и т.п.) отличаются между линейками Zyxel — сверяйтесь с CLI reference конкретной модели перед применением.

---

## 11. QoS (базовая маркировка и приоритизация)

### Cisco IOS / IOS-XE

```
class-map match-any VOICE
 match dscp ef

policy-map QOS-OUT
 class VOICE
  priority percent 20
 class class-default
  fair-queue

interface <if>
 service-policy output QOS-OUT
 mls qos trust dscp            ! доверие DSCP на входе (Catalyst)

! Проверка
show policy-map interface <if>
show mls qos interface <if>
```

Пример вывода `show policy-map interface <if>` (фрагмент):

```
  Service-policy output: QOS-OUT
    Class-map: VOICE (match-any)
      1000 packets, 128000 bytes
      priority level 1 (20% bandwidth)
```

### Huawei VRP

```
traffic classifier VOICE
 if-match dscp ef

traffic behavior VOICE-BEH
 queue af priority

traffic policy QOS-OUT
 classifier VOICE behavior VOICE-BEH

interface <if>
 traffic-policy QOS-OUT outbound
 trust dscp                    ! доверие DSCP на входе

# Проверка
display traffic-policy statistics interface <if> outbound
display qos queue-statistics interface <if>
```

Пример вывода `display traffic-policy statistics interface <if> outbound`:

```
 Item                          Packets           Bytes
 -----------------------------------------------------
 Matched                          1000          128000
 Passed                            1000          128000
 Dropped                              0               0
```

### Juniper JunOS

```
set class-of-service classifiers dscp DSCP-IN forwarding-class expedited-forwarding loss-priority low code-points ef
set class-of-service interfaces <if> unit 0 classifiers dscp DSCP-IN

set class-of-service forwarding-classes queue 0 best-effort
set class-of-service forwarding-classes queue 5 expedited-forwarding

set class-of-service schedulers VOICE-SCHED transmit-rate percent 20
set class-of-service scheduler-maps QOS-MAP forwarding-class expedited-forwarding scheduler VOICE-SCHED
set class-of-service interfaces <if> scheduler-map QOS-MAP

commit

# Проверка
show class-of-service interface <if>
show interfaces queue <if>
```

Пример вывода `show interfaces queue <if>` (фрагмент):

```
Physical interface: ge-0/0/1, Enabled, Physical link is Up
  Queue: 5, Forwarding classes: expedited-forwarding
    Queued:
      Packets              :               1000                  2 pps
```

### MikroTik RouterOS

```
; Маркировка трафика (mangle) — например, приоритизация VoIP по DSCP
/ip firewall mangle add chain=forward dscp=46 action=mark-packet new-packet-mark=voice passthrough=yes

; Очередь с приоритетом для помеченного трафика
/queue tree add name=VOICE parent=ether1 packet-mark=voice priority=1 max-limit=10M
/queue tree add name=DEFAULT parent=ether1 packet-mark=no-mark priority=8

; Ограничение скорости конкретного абонента/подсети (см. также раздел 17)
/queue simple add name=CUST_A target=192.168.100.0/24 max-limit=20M/20M

; Проверка
/queue tree print stats
/queue simple print stats
```

Пример вывода `/queue simple print stats`:

```
 0   name="CUST_A" target=192.168.100.0/24
     bytes=1250340 packets=8452
     rate=18.4Mbps/17.9Mbps
```

### Zyxel (управляемые коммутаторы GS/XGS)

```
qos type diffserv

interface gigabitethernet1/0/1
 qos trust dscp

class-map VOICE
 match dscp 46

policy-map QOS-OUT
 class VOICE
  priority queue

interface gigabitethernet1/0/2
 service-policy output QOS-OUT

! Проверка
show qos
show interfaces gigabitethernet1/0/1 qos
```

> Глубина QoS-функций сильно варьируется по линейке Zyxel: у младших моделей часто есть только простая привязка очередь-к-порту без полноценного DSCP classification — проверяйте возможности конкретной модели, поэтому пример вывода здесь не привожу.

---

## 12. Мониторинг: SNMP / Syslog

### Cisco IOS / IOS-XE

```
! SNMP
snmp-server community <ro-string> RO
snmp-server location "Almaty, NOC"
snmp-server host 10.0.0.100 version 2c <ro-string>
snmp-server enable traps

! Syslog
logging host 10.0.0.100
logging trap informational
logging source-interface Loopback0

! Проверка
show snmp
show logging
```

Пример вывода `show logging` (фрагмент):

```
Syslog logging: enabled
    Trap logging: level informational, 245 message lines logged
        Logging to 10.0.0.100, 245 message lines logged
```

### Huawei VRP

```
# SNMP
snmp-agent sys-info version v2c
snmp-agent community read cipher <ro-string>
snmp-agent target-host trap address udp-domain 10.0.0.100 params securityname <community>
snmp-agent trap enable

# Syslog
info-center loghost 10.0.0.100
info-center source default channel loghost log level informational
info-center loghost source LoopBack0

# Проверка
display snmp-agent sys-info
display info-center
```

### Juniper JunOS

```
# SNMP
set snmp community <ro-string> authorization read-only
set snmp location "Almaty, NOC"
set snmp trap-group NOC targets 10.0.0.100
set snmp trap-group NOC categories link chassis

# Syslog
set system syslog host 10.0.0.100 any info
set system syslog host 10.0.0.100 source-address <local-ip>

commit

# Проверка
show snmp statistics
show system syslog
```

### MikroTik RouterOS

```
/snmp set enabled=yes contact="NOC" location="Almaty" trap-target=10.0.0.100 trap-community=<ro-string>
/snmp community add name=<ro-string> addresses=10.0.0.0/24 read-access=yes

/system logging action add name=remote-syslog target=remote remote=10.0.0.100 remote-port=514
/system logging add topics=info,warning,error action=remote-syslog

; Проверка
/snmp print
/system logging print
/log print
```

Пример вывода `/log print` (последние строки):

```
 0 12:31:05 system,info interface ether1 link up
 1 12:31:07 snmp,info SNMP query from 10.0.0.100
```

### Zyxel (управляемые коммутаторы GS/XGS)

```
snmp-server community <ro-string> ro
snmp-server location "Almaty, NOC"
snmp-server host 10.0.0.100 <ro-string>

logging 10.0.0.100
logging trap informational

! Проверка
show snmp
show logging
```

> На security-шлюзах Zyxel (USG/ZyWALL/ATP) SNMP/syslog настраиваются через отдельные разделы CLI/GUI ("Log & Report") — синтаксис отличается от коммутаторной линейки.

---

## 13. Журнал изменений конфигурации и аудит (кто что делал)

Показывает, кто из пользователей, когда и какую команду выполнил / что изменил в конфигурации — полезно при разборе инцидентов и для контроля изменений.

### Cisco IOS / IOS-XE

```
! Включение локального журнала изменений конфигурации
archive
 log config
  logging enable
  logging size 500
  hidekeys
  notify syslog contains-config

! Просмотр
show archive log config all              ! кто и что менял в конфиге, с указанием пользователя и времени
show archive log config all provisioning ! то же самое, короткий формат
show archive log config record <N>       ! конкретная запись

! Кто сейчас в системе / активные сессии
show users
show line

! История команд текущей сессии
terminal history size 256
show history
```

Пример вывода `show archive log config all`:

```
idx   sess               user@line     Logged command
1     1                  admin@vty0    |  interface Gi0/1
2     1                  admin@vty0    |   description Subscriber-Port
```

### Huawei VRP

```
# Локальный буфер логов (при включённом info-center фиксирует вход/выход и выполненные команды)
info-center enable
info-center source SHELL channel loghost log level informational

# Просмотр
display logbuffer                          ! локальный буфер логов, включая события SHELL (команды/сессии)
display history-command all-users          ! история команд по всем пользователям (буфер настраивается отдельно)
display users                                ! кто сейчас подключён, с какого адреса
```

Пример вывода `display logbuffer` (фрагмент):

```
Aug 19 2026 09:15:03 R1 %%01SHELL/5/CMDRECORD(l): admin(10.0.0.5) in vty executed cmd interface GigabitEthernet0/0/1.
```

### Juniper JunOS

```
# Журнал коммитов — встроенная функция, всегда фиксирует пользователя и комментарий
show system commit                                    ! история commit'ов: № , время, пользователь, комментарий
show system commit revision <N> compare                ! что изменилось в конкретном commit
show configuration | compare rollback <N>                ! сравнение текущей конфигурации с версией N шагов назад

# Отдельный аудит-лог по событиям входа/commit/введённых команд
set system syslog file audit-log any info
set system syslog file audit-log match "UI_COMMIT|UI_LOGIN_EVENT|UI_CMDLINE_READ_LINE"

commit

# Просмотр
show log audit-log
show system users                                      ! кто сейчас в системе
```

Пример вывода `show system commit`:

```
0   2026-08-19 09:15:03 UTC by admin via cli
1   2026-08-18 14:02:11 UTC by admin via cli
    "added VLAN 100 for subscriber"
```

**Заметка:** для полноценного разделения «кто именно» (а не просто «через какую сессию») на всех трёх платформах нужны именные учётные записи (не общий admin-логин) — в идеале через AAA/RADIUS/TACACS+ с централизованным accounting, если такая инфраструктура есть в сети. Показанные выше команды — локальный аудит без внешнего AAA-сервера, доступный «из коробки».

### Фильтрация вывода при поиске изменений

У каждого вендора свой синтаксис pipe-фильтра — `match` работает только на Juniper, у Cisco/Huawei это `include`.

**Cisco:**

```
show archive log config all | include <username>
show logging | include CONFIG_I        ! стандартное системное сообщение "Configured from console by <user>..."
```

**Huawei:**

```
display logbuffer | include CMDRECORD  ! записи о выполненных командах (модуль SHELL)
display logbuffer | include <username>
```

**Juniper:**

```
show log messages | match "commit complete"
show log audit-log | match UI_COMMIT    ! файл audit-log — тот, что настроен выше в этом разделе
show log audit-log | match <username>
```

**Важно:** `show log config-changes` — не существующий по умолчанию файл ни на одной платформе; на Juniper нужно указывать реальное имя (`messages` или созданный вами `audit-log`), а `| match` без шаблона поиска ничего не отфильтрует — после `match` обязательно нужна строка/regex.

---

## 14. IPv6

### Cisco IOS / IOS-XE

```
ipv6 unicast-routing

interface <if>
 ipv6 address 2001:db8:100::1/64
 ipv6 enable

! Статический маршрут
ipv6 route ::/0 2001:db8:100::254

! OSPFv3
ipv6 router ospf 1
 router-id 1.1.1.1
!
interface <if>
 ipv6 ospf 1 area 0

! Проверка
show ipv6 interface brief
show ipv6 route
show ipv6 ospf neighbor
```

Пример вывода `show ipv6 interface brief`:

```
GigabitEthernet0/1    [up/up]
    FE80::211:22FF:FE33:4455
    2001:DB8:100::1
```

### Huawei VRP

```
ipv6

interface <if>
 ipv6 enable
 ipv6 address 2001:db8:100::1/64

# Статический маршрут
ipv6 route-static :: 0 2001:db8:100::254

# OSPFv3
ospfv3 1
 router-id 1.1.1.1
#
interface <if>
 ospfv3 1 area 0.0.0.0

# Проверка
display ipv6 interface brief
display ipv6 routing-table
display ospfv3 peer
```

Пример вывода `display ipv6 interface brief` (фрагмент):

```
Interface                       Physical   Protocol
GigabitEthernet0/0/1            up         up
```

### Juniper JunOS

```
set interfaces <if> unit 0 family inet6 address 2001:db8:100::1/64

set routing-options rib inet6.0 static route ::/0 next-hop 2001:db8:100::254

set protocols ospf3 area 0.0.0.0 interface <if>

commit

# Проверка
show ipv6 interface <if>
show route table inet6.0
show ospf3 neighbor
```

Пример вывода `show ipv6 interface <if>` (фрагмент):

```
Interface: ge-0/0/1.0
  Interface state: up
  Link-local address: fe80::211:22ff:fe33:4455
  Global unicast address(es): 2001:db8:100::1/64
```

---

## 15. MPLS / L3VPN

Схема: CE — PE — P — PE — CE. На PE у каждого клиента своя таблица маршрутизации (VRF), маршруты между PE передаются через MP-BGP (семейство VPNv4), трафик идёт по LSP с двумя метками: транспортной до удалённого PE и VPN-меткой, которая указывает VRF.

### Что должно работать до настройки VRF

Loopback каждого PE (маска /32) анонсирован в IGP — OSPF или IS-IS, разделы 2–3. Он же используется как LSR-ID / router-id LDP и как адрес BGP-сессии между PE.

MPLS и LDP включены на всех интерфейсах ядра, на PE и на P. Проверка: соседство LDP поднято и есть метка до loopback удалённого PE.

### Cisco IOS / IOS-XE

```
! Ядро: MPLS + LDP
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface <if>
 mpls ip

! VRF клиента
vrf definition CUST_A
 rd 65001:100
 address-family ipv4
  route-target export 65001:100
  route-target import 65001:100
 exit-address-family

interface <if2>
 vrf forwarding CUST_A                 ! привязка удаляет IP с интерфейса, адрес задаётся после неё
 ip address 192.168.1.1 255.255.255.0

! MP-BGP между PE
router bgp 65001
 neighbor 10.0.0.2 remote-as 65001
 neighbor 10.0.0.2 update-source Loopback0
 address-family vpnv4
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 send-community extended
 address-family ipv4 vrf CUST_A
  redistribute connected               ! сети клиента в VPNv4
  redistribute static
```

**Маршрутизация PE–CE (один из вариантов):**

```
! Статический маршрут в VRF
ip route vrf CUST_A 10.10.10.0 255.255.255.0 192.168.1.2

! eBGP с CE
router bgp 65001
 address-family ipv4 vrf CUST_A
  neighbor 192.168.1.2 remote-as 65100
  neighbor 192.168.1.2 activate
  neighbor 192.168.1.2 as-override     ! если у всех площадок клиента один номер AS
```

**Проверка:**

```
show mpls ldp neighbor
show mpls forwarding-table 10.0.0.2 32          ! метка до удалённого PE
show bgp vpnv4 unicast all summary              ! сессия PE–PE
show bgp vpnv4 unicast vrf CUST_A               ! маршруты клиента
show bgp vpnv4 unicast all labels               ! VPN-метки
show ip route vrf CUST_A
ping vrf CUST_A 192.168.1.10
traceroute vrf CUST_A 10.20.20.1
```

Пример вывода `show mpls ldp neighbor`:

```
    Peer LDP Ident: 10.0.0.2:0; Local LDP Ident 10.0.0.1:0
        TCP connection: 10.0.0.2.646 - 10.0.0.1.11021
        State: Oper; Msgs sent/rcvd: 245/238; Downstream
```

Пример вывода `show bgp vpnv4 unicast all summary` (фрагмент):

```
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.2        4        65001     812     809      154    0    0 12:03:41        4
```

### Huawei VRP

```
# Ядро: MPLS + LDP
mpls lsr-id 1.1.1.1                    # loopback этого PE
mpls
mpls ldp
interface <if>
 mpls
 mpls ldp

# VPN-инстанс
ip vpn-instance CUST_A
 ipv4-family
  route-distinguisher 65001:100
  vpn-target 65001:100 export-extcommunity
  vpn-target 65001:100 import-extcommunity

interface <if2>
 ip binding vpn-instance CUST_A        # привязка удаляет IP с интерфейса, адрес задаётся после неё
 ip address 192.168.1.1 255.255.255.0

# MP-BGP между PE
bgp 65001
 peer 10.0.0.2 as-number 65001
 peer 10.0.0.2 connect-interface LoopBack0
 #
 ipv4-family vpnv4
  policy vpn-target
  peer 10.0.0.2 enable
 #
 ipv4-family vpn-instance CUST_A
  import-route direct                  # сети клиента в VPNv4
  import-route static
```

**Маршрутизация PE–CE (один из вариантов):**

```
# Статический маршрут в VPN-инстансе
ip route-static vpn-instance CUST_A 10.10.10.0 24 192.168.1.2

# eBGP с CE
bgp 65001
 ipv4-family vpn-instance CUST_A
  peer 192.168.1.2 as-number 65100
  peer 192.168.1.2 substitute-as       # если у всех площадок клиента один номер AS
```

**Проверка:**

```
display mpls ldp peer
display mpls lsp | include 10.0.0.2              # метка до удалённого PE
display bgp vpnv4 all peer                       # сессия PE–PE
display bgp vpnv4 vpn-instance CUST_A routing-table
display ip routing-table vpn-instance CUST_A
ping -vpn-instance CUST_A 192.168.1.10
tracert -vpn-instance CUST_A 10.20.20.1
```

Пример вывода `display mpls ldp peer`:

```
 Peer                         Status
 10.0.0.2:0                   Operational
```

### Juniper JunOS

```
# Ядро: MPLS + LDP
set interfaces <if> unit 0 family mpls           # без family mpls интерфейс не передаёт метки
set protocols mpls interface <if>.0
set protocols ldp interface <if>.0

# VRF клиента
set routing-instances CUST_A instance-type vrf
set routing-instances CUST_A interface <if2>.0
set routing-instances CUST_A route-distinguisher 65001:100
set routing-instances CUST_A vrf-target target:65001:100
set routing-instances CUST_A vrf-table-label     # одна метка на VRF, нужна для подключённых сетей клиента
set interfaces <if2> unit 0 family inet address 192.168.1.1/24

# MP-BGP между PE
set protocols bgp group INT-VPN type internal
set protocols bgp group INT-VPN local-address 10.0.0.1
set protocols bgp group INT-VPN family inet-vpn unicast
set protocols bgp group INT-VPN neighbor 10.0.0.2

commit
```

**Маршрутизация PE–CE (один из вариантов):**

```
# Статический маршрут в VRF
set routing-instances CUST_A routing-options static route 10.10.10.0/24 next-hop 192.168.1.2

# eBGP с CE
set routing-instances CUST_A protocols bgp group CE type external
set routing-instances CUST_A protocols bgp group CE peer-as 65100
set routing-instances CUST_A protocols bgp group CE neighbor 192.168.1.2
set routing-instances CUST_A protocols bgp group CE as-override

commit
```

**Проверка:**

```
show ldp neighbor
show route table inet.3 10.0.0.2                 # LSP до удалённого PE
show bgp summary
show route table bgp.l3vpn.0
show route table CUST_A.inet.0
ping 192.168.1.10 routing-instance CUST_A
traceroute 10.20.20.1 routing-instance CUST_A
```

Пример вывода `show ldp neighbor`:

```
Address           Interface              Label space ID         Hold time
10.0.0.2          ge-0/0/1.0             10.0.0.2:0                  12
```

> Junos: если в группе BGP указано `family inet-vpn`, семейство по умолчанию (`inet unicast`) отключается. Если по этой же сессии нужны обычные IPv4-маршруты, добавьте `set protocols bgp group INT-VPN family inet unicast`.

### Если не работает

Маршрутов клиента нет на удалённом PE: сравните RT export одного PE с RT import другого, проверьте, что сессия VPNv4 поднята и передаётся extended community (у Cisco `send-community extended`).

Маршруты есть, а ping не проходит: нет LSP до loopback удалённого PE. Loopback должен быть /32 в IGP и иметь метку LDP (`show mpls forwarding-table`, `display mpls lsp`, `show route table inet.3`).

Одинаковые сети разных клиентов друг другу не мешают: RD делает префиксы уникальными. Внутри одного VRF адреса пересекаться не должны.

На route reflector для VPNv4 фильтрацию по RT отключают, иначе он отбросит маршруты чужих VRF: Huawei `undo policy vpn-target`, Cisco `no bgp default route-target filter`.

---

## 16. L2VPN — создание (VPWS / VPLS)

L2VPN передаёт кадры Ethernet клиента между площадками и не вмешивается в его IP. VPWS (VLL, l2circuit) — точка-точка между двумя портами, VPLS — общий L2-домен на много площадок. Требования к ядру те же, что для L3VPN (раздел 15): loopback /32 в IGP, MPLS и LDP на интерфейсах ядра.

### Что должно совпадать на обоих PE

VC ID (номер псевдопровода), MTU клиентских интерфейсов, тип VC — Ethernet (порт целиком) или Ethernet VLAN (подинтерфейс), настройка control word. При несовпадении любого из них псевдопровод не поднимается.

Способ сигнализации: в этом разделе Cisco и Huawei используют LDP. Для Juniper VPLS показаны оба варианта: с LDP (совместим с Cisco и Huawei) и с BGP (Juniper ↔ Juniper).

### Cisco IOS-XE

**VPWS (point-to-point, xconnect + pseudowire):**

```
pseudowire-class MPLS-PW
 encapsulation mpls

interface <if>.100
 encapsulation dot1Q 100
 xconnect 10.0.0.2 100 encapsulation mpls pw-class MPLS-PW

! На ASR 920, ME 3600 и других платформах с EVC
interface <if>
 service instance 100 ethernet
  encapsulation dot1q 100
  rewrite ingress tag pop 1 symmetric
  xconnect 10.0.0.2 100 encapsulation mpls

! Проверка
show xconnect all
show mpls l2transport vc 100 detail
ping mpls pseudowire 10.0.0.2 100
```

Пример вывода `show xconnect all` (фрагмент):

```
XC ST  Segment 1                         S1 Segment 2                         S2
------+---------------------------------+--+---------------------------------+--
UP pri ac Gi0/1.100:100(Eth VLAN)            UP mpls 10.0.0.2:100                UP
```

**VPLS (multipoint, через VFI):**

```
l2 vfi CUST_A_VPLS manual
 vpn id 100
 neighbor 10.0.0.2 encapsulation mpls

interface Vlan100
 xconnect vfi CUST_A_VPLS

! Проверка
show l2vpn vfi
show mpls l2transport vc
show mac address-table vlan 100
```

### Huawei VRP

**Общая часть для VLL и VPLS (LDP-сигнализация):**

```
mpls l2vpn
mpls ldp remote-peer 10.0.0.2          # targeted LDP до удалённого PE (его LSR-ID)
 remote-ip 10.0.0.2
```

**VLL — AR, S (VRP5), на основном интерфейсе:**

```
interface <if>
 mpls l2vc 10.0.0.2 100

# Проверка
display mpls l2vc interface <if>
```

**VLL (Martini/VPWS) — NE, CE (VRP8), на подинтерфейсе:**

```
interface <if>.100 mode l2
 encapsulation dot1q vid 100
 mpls l2vc 10.0.0.2 100

# Проверка
display mpls l2vc
display mpls l2vc interface <if>.100
```

Пример вывода `display mpls l2vc` (фрагмент):

```
 *client interface        : GigabitEthernet0/0/1.100  is up
 session state             : up
 VC state                  : up
 Peer IP address           : 10.0.0.2
```

**VPLS (multipoint) — NE, CE (VRP8):**

```
vsi CUST_A_VPLS
 pwsignal ldp
  vsi-id 100
  peer 10.0.0.2

interface <if>.100 mode l2
 encapsulation dot1q vid 100
 xconnect vsi CUST_A_VPLS

# Проверка
display vsi verbose
display vsi name CUST_A_VPLS
```

На AR и S (VRP5) интерфейс привязывается к VSI командой `l2 binding vsi CUST_A_VPLS` (например, на Vlanif100), а не `xconnect vsi`.

### Juniper JunOS

**l2circuit (VPWS, point-to-point):**

```
set interfaces <if> vlan-tagging
set interfaces <if> unit 100 encapsulation vlan-ccc
set interfaces <if> unit 100 vlan-id 100
set interfaces <if> unit 100 family ccc

set protocols ldp interface lo0.0                # targeted LDP до удалённого PE
set protocols l2circuit neighbor 10.0.0.2 interface <if>.100 virtual-circuit-id 100
# если MTU на концах отличается:
# set protocols l2circuit neighbor 10.0.0.2 interface <if>.100 ignore-mtu-mismatch

commit

# Проверка
show l2circuit connections
show ldp neighbor
```

Пример вывода `show l2circuit connections` (фрагмент):

```
Neighbor: 10.0.0.2
    Interface                 Type  St     Time last up          # Up trans
    ge-0/0/1.100(vc 100)      rmt   Up     Aug 19 09:15:03 2026           1
```

**VPLS с LDP-сигнализацией (совместим с Cisco и Huawei):**

```
set interfaces <if> vlan-tagging
set interfaces <if> encapsulation vlan-vpls
set interfaces <if> unit 100 encapsulation vlan-vpls
set interfaces <if> unit 100 vlan-id 100
set interfaces <if> unit 100 family vpls

set protocols ldp interface lo0.0
set routing-instances CUST_A_VPLS instance-type vpls
set routing-instances CUST_A_VPLS interface <if>.100
set routing-instances CUST_A_VPLS protocols vpls vpls-id 100
set routing-instances CUST_A_VPLS protocols vpls neighbor 10.0.0.2
set routing-instances CUST_A_VPLS protocols vpls no-tunnel-services

commit
```

**VPLS с BGP-сигнализацией (Juniper ↔ Juniper):**

```
set routing-instances CUST_A_VPLS instance-type vpls
set routing-instances CUST_A_VPLS interface <if>.100
set routing-instances CUST_A_VPLS route-distinguisher 65001:100
set routing-instances CUST_A_VPLS vrf-target target:65001:100
set routing-instances CUST_A_VPLS protocols vpls site-range 10
set routing-instances CUST_A_VPLS protocols vpls no-tunnel-services
set routing-instances CUST_A_VPLS protocols vpls site SITE-A site-identifier 1
set protocols bgp group INT-VPN family l2vpn signaling   # без этого BGP VPLS не поднимется

commit

# Проверка (для обоих вариантов VPLS)
show vpls connections
show vpls mac-table instance CUST_A_VPLS
show route instance CUST_A_VPLS
```

### Если псевдопровод не поднимается

Где смотреть причину: Cisco — `show mpls l2transport vc 100 detail`; Huawei — `display mpls l2vc interface <if>` (поле VC state и причина); Juniper — `show l2circuit connections` или `show vpls connections`, код в колонке St расшифрован в легенде вывода (например, MM — несовпадение MTU, EM — неподходящая инкапсуляция, OL — нет метки от соседа).

Частые причины: разный VC ID, разный MTU, разный тип VC (порт целиком с одной стороны, подинтерфейс с другой), нет targeted LDP между PE, нет LSP до loopback удалённого PE.

> VPWS (xconnect / l2vc / l2circuit) — «точка-точка», один канал между двумя PE. VPLS — «многоточка», эмулирует общий L2-домен между несколькими площадками через VFI / VSI / routing-instance типа vpls. Современная замена VPLS — EVPN, где MAC-адреса передаются через BGP.

---

## 17. Изменение скорости L2 / L3 VPN (увеличение / снижение)

Смена тарифной скорости абонентского VPN-канала (L2 или L3) делается правкой shaping/policing на клиентском интерфейсе или в сервис-политике. Сам VPN-сервис (xconnect / vsi / vrf) при этом не пересоздаётся. Ограничивают обычно оба направления: исходящее к клиенту — shaping, входящее от клиента — policing.

### Cisco IOS-XE

```
! К клиенту: shaping
policy-map CUST_A_SHAPE
 class class-default
  shape average 20000000        ! 20 Мбит/с — текущая скорость клиента

! От клиента: policing
policy-map CUST_A_POLICE
 class class-default
  police cir 20000000

interface <if>.100
 service-policy output CUST_A_SHAPE
 service-policy input CUST_A_POLICE

! Увеличение до 50 Мбит/с — правим значения в тех же policy-map
policy-map CUST_A_SHAPE
 class class-default
  shape average 50000000
policy-map CUST_A_POLICE
 class class-default
  police cir 50000000

! Снижение — так же, меньшим значением

! Проверка
show policy-map interface <if>.100
show policy-map CUST_A_SHAPE
```

Пример вывода `show policy-map interface <if>.100` (фрагмент):

```
  Service-policy output: CUST_A_SHAPE
    Class-map: class-default
      Traffic Shaping
        Target/Average   Byte   Sustain   Excess    Interval  Increment
        Rate             Limit  bits/int  bits/int  (ms)      (bytes)
        50000000/50000000 625000 2500000  2500000   50        312500
```

### Huawei VRP

```
traffic behavior CUST_A_BEH
 car cir 20000 pir 20000 cbs 3750000 pbs 3750000 green pass yellow pass red discard
 statistic enable               # счётчики для display traffic-policy statistics (S, AR)

traffic classifier CUST_A_CLASS
 if-match any

traffic policy CUST_A_POLICY
 classifier CUST_A_CLASS behavior CUST_A_BEH

interface <if>.100 mode l2
 traffic-policy CUST_A_POLICY inbound
 traffic-policy CUST_A_POLICY outbound

# Увеличение до 50 Мбит/с — правим cir/pir в поведении
traffic behavior CUST_A_BEH
 car cir 50000 pir 50000 cbs 9375000 pbs 9375000 green pass yellow pass red discard

# Проверка
display traffic-policy statistics interface <if>.100 outbound
display traffic behavior user-defined CUST_A_BEH
```

Пример вывода `display traffic behavior user-defined CUST_A_BEH` (фрагмент):

```
Behavior: CUST_A_BEH
 Committed Access Rate:
  CIR: 50000 (kbps), PIR: 50000 (kbps), CBS: 9375000 (byte), PBS: 9375000 (byte)
```

На NE и CE (VRP8) счётчики включаются в самой политике: `statistics enable` в режиме `traffic policy`.

### Juniper JunOS

```
# К клиенту: shaping на логическом интерфейсе
set interfaces <if> per-unit-scheduler                       # без этого shaping-rate на unit не применится
set class-of-service interfaces <if> unit 100 shaping-rate 20m

# Увеличение до 50 Мбит/с
set class-of-service interfaces <if> unit 100 shaping-rate 50m

# От клиента: policer
set firewall policer CUST_A_POLICER if-exceeding bandwidth-limit 20m burst-size-limit 1m
set firewall policer CUST_A_POLICER then discard
set interfaces <if> unit 100 family inet policer input CUST_A_POLICER    # L3VPN
set interfaces <if> unit 100 family ccc policer input CUST_A_POLICER     # L2VPN (l2circuit)

commit

# Проверка
show class-of-service interface <if>
show interfaces <if> extensive | match Shaping
show policer
```

> CIR (Committed Information Rate) — гарантированная скорость, PIR (Peak Information Rate) — пиковая. При смене тарифа меняются именно эти параметры в shaping, policer или CAR. Интерфейс, VLAN и VPN-сервис остаются как есть.

> MikroTik и Zyxel обычно не используются как PE для MPLS L2/L3VPN. Смена скорости на них описана в разделе 11 (QoS): на MikroTik `/queue simple set [find name=CUST_A] max-limit=50M/50M` правит лимит в существующей очереди абонента.

---

## 18. Проверка физических портов

### Cisco IOS / IOS-XE

```
show interfaces <if>                      ! полная статистика: статус, ошибки, счётчики
show interfaces <if> status                ! краткий статус порта (connected/notconnect, VLAN, duplex, speed)
show interfaces description                 ! статус + description всех портов одной таблицей
show interfaces <if> counters errors        ! счётчики ошибок: CRC, input errors, collisions
show interfaces <if> counters               ! счётчики трафика: unicast/broadcast/multicast
show controllers <if>                        ! физическое состояние на уровне hw (платформозависимо)

! Скорость/дуплекс вручную, если не auto
interface <if>
 speed 1000
 duplex full
```

Пример вывода `show interfaces <if> status`:

```
Port      Name               Status       Vlan       Duplex  Speed Type
Gi0/1                        connected    100        a-full  a-1000 1000BaseT
```

### Huawei VRP

```
display interface <if>                       ! полная статистика по порту
display interface brief                       ! краткий статус всех портов
display interface <if> | include CRC           ! точечный поиск ошибок в выводе
display counter interface <if>                 ! счётчики трафика/ошибок
display interface <if> statistics              ! доп. статистика (платформозависимо)

# Скорость/дуплекс вручную
interface <if>
 speed 1000
 duplex full
```

Пример вывода `display interface brief` (фрагмент):

```
Interface                   PHY   Protocol InUti OutUti   inErrors  outErrors
GigabitEthernet0/0/1        up    up          0%     0%          0          0
```

### Juniper JunOS

```
show interfaces <if>                          ! полная статистика
show interfaces <if> terse                     ! краткий статус: admin/link state, IP
show interfaces <if> extensive                  ! максимально подробно, включая ошибки и оптику
show interfaces <if> statistics                 ! только счётчики трафика

# Скорость/дуплекс вручную
set interfaces <if> speed 1g
set interfaces <if> link-mode full-duplex

commit
```

Пример вывода `show interfaces <if> terse`:

```
Interface               Admin Link Proto    Local                 Remote
ge-0/0/1                up    up
ge-0/0/1.0               up    up   inet     192.168.1.1/24
```

### MikroTik RouterOS

```
/interface ethernet print                        ; список портов, базовый статус
/interface ethernet monitor ether1 once             ; статус линка, скорость, дуплекс — детально
/interface ethernet print stats                       ; счётчики ошибок/трафика

; Скорость/дуплекс вручную
/interface ethernet set ether1 speed=1Gbps full-duplex=yes auto-negotiation=no
```

Пример вывода `/interface ethernet monitor ether1 once`:

```
                      name: ether1
                    status: link-ok
      auto-negotiation: done
                      rate: 1Gbps
              full-duplex: yes
```

### Zyxel (управляемые коммутаторы GS/XGS)

```
show interfaces status                    ! статус всех портов: up/down, скорость, дуплекс
show interfaces gigabitethernet1/0/1      ! подробная статистика по порту, включая ошибки

interface gigabitethernet1/0/1
 speed 1000
 duplex full
```

Пример вывода `show interfaces status` (общая форма, у Zyxel варьируется по модели):

```
Port   Status       Speed    Duplex   VLAN
1      Up           1000M    Full     100
```

---

## 19. Мониторинг трафика на интерфейсе (реальное время / захват пакетов)

### Текущая загрузка (rate в реальном времени)

**Cisco:**

```
show interfaces <if> | include rate
! В выводе show interfaces <if> ищите строки вида:
!  5 minute input rate 1000 bits/sec, 2 packets/sec
!  5 minute output rate 500 bits/sec, 1 packets/sec
```

Пример вывода:

```
  5 minute input rate 1000 bits/sec, 2 packets/sec
  5 minute output rate 500 bits/sec, 1 packets/sec
```

**Huawei:**

```
display interface <if> | include rate
! В выводе ищите строки вида:
!  Last 300 seconds input rate ... bits/sec, ... packets/sec
!  Last 300 seconds output rate ... bits/sec, ... packets/sec
```

**Juniper:**

```
monitor interface <if>            ! живой дашборд по интерфейсу (обновляется как top), Ctrl+C для выхода
monitor interface traffic          ! живая таблица по всем интерфейсам сразу
show interfaces <if> extensive | match rate
```

### Захват пакетов прямо на устройстве (live packet capture)

**Juniper (встроенный tcpdump-подобный захват):**

```
monitor traffic interface <if>                              ! живой вывод пакетов в CLI
monitor traffic interface <if> size 1500 no-resolve            ! полный размер пакета, без резолва DNS
monitor traffic interface <if> matching "port 80"                ! фильтр по BPF-выражению
monitor traffic interface <if> detail                             ! подробный разбор заголовков
monitor traffic interface <if> write-file /var/tmp/capture.pcap    ! сохранение в pcap-файл

! Остановка — Ctrl+C
```

Пример вывода `monitor traffic interface <if>`:

```
09:15:03.123456 In IP 192.168.1.10.54321 > 10.10.0.1.80: Flags [S], seq 123456789, win 64240, length 0
09:15:03.123789 Out IP 10.10.0.1.80 > 192.168.1.10.54321: Flags [S.], seq 987654321, ack 123456790, win 65160, length 0
```

**Cisco IOS-XE (Embedded Packet Capture):**

```
monitor capture CAP1 interface <if> both
monitor capture CAP1 match any
monitor capture CAP1 buffer size 10

monitor capture CAP1 start
! ... дождаться нужного трафика ...
monitor capture CAP1 stop

show monitor capture CAP1 buffer detailed
monitor capture CAP1 export bootflash:cap1.pcap    ! сохранить в pcap для Wireshark
```

**Huawei:** встроенный захват пакетов в CLI сильно различается по модельному ряду и не так стандартизирован, как на Cisco/Juniper — на большинстве платформ для детального анализа трафика используют зеркалирование на внешний анализатор (см. ниже), сверяйтесь с документацией конкретной модели по наличию локального `packet-capture`.

### Зеркалирование трафика (SPAN / port-mirroring) — для внешнего анализа (Wireshark и т.п.)

**Cisco:**

```
monitor session 1 source interface <if_source> both
monitor session 1 destination interface <if_dest>

! Проверка
show monitor session all
```

**Huawei:**

```
observe-port 1 interface <if_dest>

interface <if_source>
 port-mirroring to observe-port 1 both

# Проверка
display observe-port
display port-mirroring
```

**Juniper (EX-style analyzer):**

```
set forwarding-options analyzer AN1 input ingress interface <if_source>
set forwarding-options analyzer AN1 input egress interface <if_source>
set forwarding-options analyzer AN1 output interface <if_dest>

commit

# Проверка
show forwarding-options analyzer
```

---

## 20. Проверка затухания (оптический уровень, DOM)

Проверяется через показания DOM (Digital Optical Monitoring) самого трансивера: уровень приёма (RX) и передачи (TX) в dBm. Затухание в линии ≈ TX-уровень удалённой стороны минус RX-уровень локальной стороны (с поправкой на потери в разъёмах/сплиттерах).

### Cisco IOS / IOS-XE

```
show interfaces <if> transceiver detail        ! Rx/Tx Power, температура, напряжение, ток смещения (bias)
show interfaces <if> transceiver                ! краткая сводка без detail
```

Пример вывода `show interfaces <if> transceiver detail` (фрагмент):

```
                                 Optical   Optical
           Temperature  Voltage  Tx Power  Rx Power
Port       (Celsius)    (Volts)  (dBm)     (dBm)
Gi0/1         32.5        3.30    -3.10     -12.50
```

### Huawei VRP

```
display transceiver interface <if> verbose      ! подробный DOM: Rx/Tx Power, температура, напряжение, bias

# Для GPON-абонентского порта (ONU на OLT) — см. также раздел 9
display ont optical-info 0 1 0                    ! RX/TX power конкретного ONU
```

Пример вывода `display transceiver interface <if> verbose` (фрагмент):

```
GigabitEthernet0/0/1 transceiver information:
Rx Power(dBm)      : -12.50
Tx Power(dBm)      : -3.10
Temperature(C)     : 32
Voltage(V)         : 3.30
Bias Current(mA)   : 6.20
```

### Juniper JunOS

```
show interfaces diagnostics optics <if>           ! Rx/Tx Power (dBm/mW), температура, ток смещения
```

Пример вывода `show interfaces diagnostics optics <if>` (фрагмент):

```
Laser bias current                       :   6.200 mA
Laser output power                       :  -0.79 dBm
Module temperature                       :  32 degrees C
Receiver signal average optical power    : -10.97 dBm
```

### MikroTik RouterOS

```
/interface ethernet monitor sfp1 once
; В выводе поля: sfp-rx-power, sfp-tx-power (в dBm), sfp-temperature,
;                sfp-supply-voltage, sfp-tx-bias-current
```

Пример вывода:

```
        sfp-rx-power: -10.97
        sfp-tx-power: -0.79
     sfp-temperature: 32
```

### Zyxel (управляемые коммутаторы GS/XGS с поддержкой DDM/DOM)

```
show interfaces transceiver-info gigabitethernet1/0/1
! При наличии DDM-поддержки выводит Rx/Tx Power, температуру, напряжение
```

> Поддержка DDM/DOM через CLI есть не на всех моделях Zyxel — на части устройств оптические уровни видны только через веб-интерфейс (Web GUI → Port Status), либо не выводятся вовсе на бюджетных моделях. Из-за этого пример вывода здесь не привожу — слишком велик разброс по моделям.

**Ориентиры по уровням (сильно зависят от модели трансивера — сверяйтесь с datasheet):**

- Обычный SFP/SFP+ (LX/ZX): типичный допустимый RX — от −3 до −23 dBm.

- GPON ONU: обычно −8…−27 dBm при бюджете класса B+/C+.

- Резкое падение RX относительно паспортного TX удалённой стороны сверх ожидаемых потерь — признак загрязнённого/повреждённого коннектора, изгиба волокна или деградации сварного соединения.

---

## 21. Информация о SFP / трансивере (все виды и модели)

### Cisco IOS / IOS-XE

```
show interfaces <if> transceiver detail        ! включает Vendor Name, Part Number, Serial, длину волны
show inventory                                    ! список всех модулей во всех слотах, включая SFP
```

Пример вывода `show interfaces <if> transceiver detail` (фрагмент):

```
Gi0/1
    Transceiver Type:  SFP-1000BASE-LX
    Vendor Name:       HUAWEI
    Vendor PN:         MA5671A
    Serial Number:     21012345
```

### Huawei VRP

```
display transceiver interface <if> verbose        ! Vendor Name, PN, длина волны, дальность передачи, DDM-статус
display device manufacture-info                    ! сводная информация по установленным модулям
```

Пример вывода `display transceiver interface <if> verbose` (фрагмент):

```
Manu. SN            : 21012345
Vendor Name          : HUAWEI
Transceiver Type     : 1000_BASE_LX_SFP
Wave Length(nm)      : 1310
```

### Juniper JunOS

```
show chassis hardware                              ! список всех модулей (SFP/SFP+/QSFP/XFP) с моделью и серийником
show chassis pic pic-slot <N> fpc-slot <N>           ! детально по конкретному FPC/PIC
show interfaces diagnostics optics <if>              ! + оптические параметры того же модуля
```

Пример вывода `show chassis hardware` (фрагмент):

```
Item             Version  Part number  Serial number     Description
Xcvr 0           REV 01   740-011782   AB1234567890      SFP+-10G-LR
```

### MikroTik RouterOS

```
/interface ethernet monitor sfp1 once
; Дополнительно к DOM (см. раздел 20) те же поля включают:
; sfp-vendor-name, sfp-vendor-part-number, sfp-vendor-serial,
; sfp-wavelength, sfp-type (тип модуля: 1G/10G, SR/LR/ZR и т.п.)
```

### Zyxel

```
show interfaces transceiver-info gigabitethernet1/0/1
! Vendor/PN/serial при наличии DDM-поддержки модели
```

**Типы трансиверов по дальности/длине волны (актуально для всех вендоров, форм-фактор не зависит от производителя роутера):**

| Тип | Длина волны | Дальность | Волокно |
|---|---|---|---|
| SX | 850 нм | до ~550 м | MMF (многомод) |
| LX / LH | 1310 нм | до ~10 км | SMF (одномод) |
| ZX | 1550 нм | до ~70–80 км | SMF |
| BiDi | разные TX/RX (напр. 1310/1550) | зависит от модели | одно волокно (single-fiber) |
| CWDM / DWDM | набор фиксированных длин волн | до 40–120+ км | SMF, с мультиплексором |

Форм-факторы по скорости: **SFP** (1G), **SFP+** (10G), **QSFP+** (40G), **QSFP28** (100G), **XFP** (10G, устаревающий). Совместимость трансивера с портом определяется и форм-фактором, и поддержкой конкретной моделью оборудования (некоторые вендоры программно блокируют модули сторонних производителей — проверяйте `unauthorized`/`non-Cisco`-предупреждения в выводе `show interfaces transceiver`).

---

## 22. IPv4 — адресация и сервисы

### Адресация интерфейса

**Cisco:**

```
interface <if>
 ip address 192.168.1.1 255.255.255.0
 ip address 192.168.1.2 255.255.255.0 secondary   ! вторичный адрес

! Проверка
show ip interface brief
show ip interface <if>
```

**Huawei:**

```
interface <if>
 ip address 192.168.1.1 255.255.255.0
 ip address 192.168.1.2 255.255.255.0 sub          ! вторичный адрес

# Проверка
display ip interface brief
display ip interface <if>
```

**Juniper:**

```
set interfaces <if> unit 0 family inet address 192.168.1.1/24
set interfaces <if> unit 0 family inet address 192.168.1.2/24   ! доп. адрес на том же unit

commit

# Проверка
show interfaces <if> terse
show interfaces <if> family inet
```

**MikroTik:**

```
/ip address add address=192.168.1.1/24 interface=<if>
/ip address add address=192.168.1.2/24 interface=<if>   ; доп. адрес на том же интерфейсе

; Проверка
/ip address print
```

**Zyxel (управляемые коммутаторы GS/XGS):**

```
interface vlan100
 ip address 192.168.1.1 255.255.255.0

! Проверка
show ip interface brief
```

### ARP

**Cisco:**

```
show arp
show ip arp
arp 192.168.1.10 0011.2233.4455 arpa        ! статическая запись
clear arp-cache
```

Пример вывода `show ip arp`:

```
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.10            5   0011.2233.4455  ARPA   GigabitEthernet0/1
```

**Huawei:**

```
display arp
display arp interface <if>
arp static 192.168.1.10 0011-2233-4455        ! статическая запись
reset arp all
```

**Juniper:**

```
show arp
show arp interface <if>
set interfaces <if> unit 0 family inet address 192.168.1.1/24 arp 192.168.1.10 mac 00:11:22:33:44:55
clear arp
```

**MikroTik:**

```
/ip arp print
/ip arp add address=192.168.1.10 mac-address=00:11:22:33:44:55 interface=<if>   ; статическая запись
```

**Zyxel:**

```
show arp
arp 192.168.1.10 0011.2233.4455        ! статическая запись (синтаксис близок к Cisco)
```

### DHCP relay

**Cisco:**

```
interface <if>
 ip helper-address 10.0.0.100
```

**Huawei:**

```
interface <if>
 dhcp select relay
 dhcp relay server-ip 10.0.0.100
# Проверка
display dhcp relay interface <if>
```

**Juniper:**

```
set forwarding-options dhcp-relay server-group DHCP-SRV 10.0.0.100
set forwarding-options dhcp-relay group RELAY active-server-group DHCP-SRV
set forwarding-options dhcp-relay group RELAY interface <if>

commit
# Проверка
show dhcp relay binding
```

**Старый способ «forwarding-options helpers bootp» встречается на старых версиях Junos и EX. На актуальных версиях используйте dhcp-relay.**

**MikroTik:**

```
/ip dhcp-relay add interface=<if> dhcp-server=10.0.0.100
```

> Zyxel: DHCP relay есть на L3-моделях коммутаторов и на большинстве security-шлюзов, но точный синтаксис (`ip helper` vs отдельный раздел "DHCP Relay" в GUI) отличается по линейке — проверяйте документацию модели.

### NAT / PAT

**Cisco:**

```
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface <if_outside> overload

interface <if_inside>
 ip nat inside
interface <if_outside>
 ip nat outside

! Проверка
show ip nat translations
show ip nat statistics
```

**Huawei:**

```
acl number 2000
 rule 5 permit source 192.168.1.0 0.0.0.255

interface <if_outside>
 nat outbound 2000

# Проверка
display nat session all
display nat outbound
```

**Juniper (пример для SRX; на MX/EX синтаксис NAT через `services nat` отличается):**

```
set security nat source pool SRC-POOL address 203.0.113.1/32
set security nat source rule-set NAT-OUT from zone trust
set security nat source rule-set NAT-OUT to zone untrust
set security nat source rule-set NAT-OUT rule R1 match source-address 192.168.1.0/24
set security nat source rule-set NAT-OUT rule R1 then source-nat pool SRC-POOL

commit

# Проверка
show security nat source summary
```

**MikroTik:**

```
/ip firewall nat add chain=srcnat src-address=192.168.1.0/24 out-interface=<if_outside> action=masquerade

; Проверка
/ip firewall nat print stats
```

> Zyxel: NAT/PAT реализован в основном на security-шлюзах (USG/ZyWALL/ATP) через объекты SNAT/Policy Route в GUI/CLI — на чистых L2/L3-коммутаторах NAT обычно недоступен.

### Диагностика (расширенные ping/traceroute)

**Cisco:**

```
ping 192.168.1.1 source Loopback0 repeat 100 size 1500           ! size = весь IP-пакет
ping 192.168.1.1 size 1500 df-bit                               ! проверка MTU 1500 без фрагментации
traceroute 192.168.1.1
```

Пример вывода `ping`:

```
Sending 100, 1500-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!!!!!!
Success rate is 100 percent (100/100), round-trip min/avg/max = 1/2/4 ms
```

**Huawei:**

```
ping -a 10.0.0.1 -c 100 -s 1472 192.168.1.1     # -s = только данные ICMP: 1472 + 28 = пакет 1500
ping -f -s 1472 192.168.1.1                      # -f = не фрагментировать, проверка MTU 1500
tracert 192.168.1.1
```

**Juniper:**

```
ping 192.168.1.1 source <local-ip> count 100 size 1472   # size = только данные ICMP: 1472 + 28 = пакет 1500
ping 192.168.1.1 size 1472 do-not-fragment                # проверка MTU 1500
traceroute 192.168.1.1
```

**MikroTik:**

```
/ping 192.168.1.1 count=100 size=1500 src-address=<local-ip>
/tool traceroute 192.168.1.1
```

**Zyxel:**

```
ping 192.168.1.1
traceroute 192.168.1.1
```

**Размер в ping считается по-разному. У Cisco size — весь IP-пакет, у Huawei -s и Juniper size — только данные ICMP, без заголовков IP (20 байт) и ICMP (8 байт). Чтобы отправить пакеты одного размера, на Huawei и Juniper указывайте на 28 меньше.**

### Шпаргалка по подсетям (VLSM)

| CIDR | Маска | Всего адресов | Хостов |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /31 | 255.255.255.254 | 2 | 2 (point-to-point, RFC 3021) |
| /32 | 255.255.255.255 | 1 | 1 (host route) |

---

## 23. Просмотр таблицы маршрутизации (варианты show route / фильтрация)

Чтобы не листать всю таблицу целиком, у каждого вендора есть набор фильтров по протоколу, префиксу, таблице/VRF и атрибутам BGP.

### Juniper JunOS (show route)

```
show route                                        ! вся таблица inet.0
show route table inet6.0                            ! конкретная таблица (IPv6, VRF и т.д.)
show route table <vrf-name>.inet.0                     ! маршруты конкретного VRF/routing-instance

show route 10.10.0.0/24                                  ! конкретный префикс (с более широкими/узкими совпадениями)
show route 10.10.0.0/24 exact                               ! точное совпадение маски, без over/under
show route 10.10.0.0/24 active-path                            ! только активный (выбранный) путь для префикса
show route 10.10.0.0/24 inactive-path                             ! только неактивные пути (не выбранные как best)

show route protocol bgp                                              ! только маршруты конкретного протокола
show route protocol static
show route protocol ospf
show route protocol isis

show route best 10.10.0.0/24                                            ! лучший маршрут к адресу
show route hidden                                                          ! скрытые маршруты (отсеяны policy)
show route summary                                                            ! сводка: сколько маршрутов на таблицу/протокол

show route detail                                                                ! подробный вывод с атрибутами
show route terse                                                                    ! компактная табличная форма
show route extensive                                                                  ! максимальная детализация

show route receive-protocol bgp <neighbor>                                              ! что принято от BGP-соседа
show route advertising-protocol bgp <neighbor>                                             ! что анонсируется соседу
show route next-hop 192.168.1.2                                                               ! маршруты через конкретный next-hop
show route community <community-name>                                                            ! фильтр по BGP community
show route aspath-regex "^65002 "                                                                   ! фильтр по AS-path regex

show route forwarding-table                                                                            ! таблица пересылки (FIB), а не RIB

# Универсальный пост-фильтр вывода поверх любой из команд выше
show route protocol bgp | match 10.10.
```

Пример вывода `show route protocol bgp` (одна запись):

```
10.10.0.0/24       *[BGP/170] 01:12:45, localpref 100, from 192.168.1.2
                      AS path: 65002 I
                    > to 192.168.1.2 via ge-0/0/1.0
```

### Cisco IOS / IOS-XE (show ip route)

```
show ip route                                     ! вся таблица
show ip route vrf <vrf-name>                         ! маршруты конкретного VRF

show ip route 10.10.0.0 255.255.255.0                   ! конкретная сеть
show ip route 10.10.0.0 255.255.255.0 longer-prefixes     ! включая все более специфичные подсети

show ip route static                                        ! только статические
show ip route ospf
show ip route bgp
show ip route eigrp

show ip route summary                                          ! сводка по количеству маршрутов на протокол
show ip cef                                                        ! таблица пересылки (CEF), а не RIB

show ip bgp                                                          ! вся BGP RIB (отдельно от show ip route)
show ip bgp 10.10.0.0                                                   ! конкретный BGP-префикс с атрибутами
show ip bgp regexp ^65002_                                                 ! фильтр по AS-path regex
show ip bgp community 65001:100                                              ! фильтр по community

! Универсальный пост-фильтр
show ip route | include <pattern>
```

Пример вывода `show ip route bgp` (одна запись):

```
B    10.10.0.0/24 [20/0] via 192.168.1.2, 01:12:45
```

### Huawei VRP (display ip routing-table)

```
display ip routing-table                                    ! вся таблица
display ip routing-table vpn-instance <vrf-name>                ! маршруты конкретного VPN-инстанса

display ip routing-table 10.10.0.0 24                            ! конкретная сеть
display ip routing-table 10.10.0.0 24 longer-match                 ! включая более специфичные подсети

display ip routing-table protocol static                             ! только статические
display ip routing-table protocol ospf
display ip routing-table protocol bgp
display ip routing-table protocol isis

display ip routing-table statistics                                     ! сводка по количеству маршрутов
display fib                                                                 ! таблица пересылки (FIB), а не RIB

display bgp routing-table                                                      ! вся BGP RIB
display bgp routing-table 10.10.0.0 24                                            ! конкретный BGP-префикс
display bgp routing-table regular-expression ^65002                                  ! фильтр по AS-path regex
display bgp routing-table community 65001:100                                          ! фильтр по community

# Универсальный пост-фильтр
display ip routing-table | include <pattern>
```

Пример вывода `display ip routing-table protocol bgp` (одна запись):

```
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
    10.10.0.0/24    BGP     255  0         D     192.168.1.2     GigabitEthernet0/0/1
```

**Заметка:** `show route` / `show ip route` / `display ip routing-table` — это таблица маршрутизации (RIB), а `show route forwarding-table` / `show ip cef` / `display fib` — уже готовая к пересылке таблица (FIB); при поиске конкретного проблемного маршрута обычно достаточно RIB с фильтром по протоколу или префиксу, к FIB обращаются, когда нужно понять, что реально «зашито» в hardware/железе для пересылки пакетов.

---

## 24. Конфликты (пересечение) IP-адресов и флапающие интерфейсы

### Проверка конфликтов/дублирования IP-адресов

**Cisco IOS / IOS-XE:**

```
show logging | include DUPADDR      ! системное сообщение при обнаружении дубликата:
                                     ! %IP-4-DUPADDR: Duplicate address 192.168.1.10 on Vlan10, sourced by 0011.2233.4455
show ip arp                          ! проверить текущую ARP-запись — не "скачет" ли MAC для одного IP
show ip arp <ip>                       ! запись по конкретному адресу
debug arp                                ! временная отладка ARP-обмена (осторожно на проде — высокая нагрузка на CPU)
```

Пример строки в логе при конфликте:

```
%IP-4-DUPADDR: Duplicate address 192.168.1.10 on Vlan10, sourced by 0011.2233.4455
```

**Huawei VRP:**

```
display arp | include <ip>            ! текущая ARP-запись для конкретного адреса
display logbuffer | include duplicate   ! поиск сообщений о конфликте IP в локальном буфере логов
arp anti-attack gateway-duplicate enable  ! защита от подмены IP шлюза — частая причина "конфликтов" на VLAN абонентов
```

**Juniper JunOS:**

```
show arp                              ! текущая ARP-таблица (IPv4)
show arp | match <ip>                   ! запись по конкретному адресу
show ipv6 neighbors                       ! аналог ARP для IPv6 (Neighbor Discovery)
show log messages | match "duplicate"       ! системный лог — конфликты IP/DAD-события (актуально в основном для IPv6)
```

> Точный формат сообщения о дубликате IP различается между платформами и даже между версиями ПО одного вендора — если готового `show`-варианта нет, ищите по ключевым словам `duplicate`/`conflict` в системном логе (см. раздел 10 — там же настраивается сохранение логов на syslog-сервер для истории).

### Диагностика флапающих интерфейсов

**Cisco IOS / IOS-XE:**

```
show interfaces <if>                              ! ищите строку "N carrier transitions" — счётчик переходов up/down
show interfaces <if> counters errors                 ! рост ошибок (CRC и т.п.) часто сопутствует флапу
show logging | include <if>                             ! все события по конкретному интерфейсу из системного лога
show logging | include LINK-3-UPDOWN|LINEPROTO-5-UPDOWN   ! стандартные сообщения об изменении состояния линка
```

Пример строки в выводе `show interfaces <if>`:

```
     15 carrier transitions
```

**Huawei VRP:**

```
display interface <if>                    ! на большинстве платформ есть поле о времени последнего изменения состояния
display logbuffer | include <if>             ! события up/down в локальном буфере
display trapbuffer | include Down              ! trap-события об изменении состояния линка
```

**Juniper JunOS:**

```
show interfaces <if> extensive | match "Last flapped|transitions"   ! время последнего flap и счётчик переходов
show interfaces <if> extensive | find "Interface transitions"          ! точное количество переходов up/down
show log messages | match "<if>"                                          ! все события по интерфейсу из системного лога
monitor interface <if>                                                       ! живое наблюдение за состоянием (см. раздел 17)
```

**Заметка:** хронический флап почти всегда физического происхождения — первым делом проверяйте затухание/DOM (раздел 18) и счётчики ошибок порта (раздел 16): грязный/повреждённый коннектор, плохой SFP или пережатый кабель дают именно такую картину, а не проблему в конфигурации протокола.

---

## 25. Общие команды диагностики

| Задача | Cisco | Huawei | Juniper |
|---|---|---|---|
| Проверка связности | ping <ip> | ping <ip> | ping <ip> |
| Трассировка | traceroute <ip> | tracert <ip> | traceroute <ip> |
| Таблица маршрутизации | show ip route | display ip routing-table | show route |
| Статус интерфейса | show ip interface brief | display ip interface brief | show interfaces terse |
| Просмотр конфига | show running-config | display current-configuration | show configuration |
| Сохранение конфига | write memory / copy run start | save | commit (авто-применение) |
| Откат последних изменений | — (нужен archive) | — | rollback 1 |

---

## 26. Ключевые различия между платформами

- **Модель применения конфига:** Cisco и Huawei — построчный ввод команд, применяются сразу; Juniper — кандидатная конфигурация, применяется только по `commit` (что даёт встроенный rollback).

- **Административная дистанция / preference:** значения по умолчанию различаются между вендорами (см. заметку в разделе про статику) — важно при дублирующих маршрутах разных протоколов.

- **Именование интерфейсов:** Cisco — `GigabitEthernet0/0/1`, Huawei — аналогично, Juniper — `ge-0/0/1` (FPC/PIC/Port).

- **OSPF area 0** везде называется backbone-областью, но у Huawei/Juniper часто явно пишется `0.0.0.0`, у Cisco допустима короткая форма `0`.

*Документ покрывает базовый набор задач по маршрутизации, коммутации, VPN-провижинингу, физической/оптической диагностике и IPv4. Можно дополнять дальше по мере появления новых практических задач.*

---

# Active Directory Lab - Lépésről lépésre útmutató

<img src="https://i.imgur.com/9gcuSv2.png" height="80%" width="100%" alt="Active Directory"/>

Ez az útmutató végigvezeti azt, hogy hogyan készítettem el ezt az Active Directory laboratóriumi környezetet az Oracle VirtualBox, Windows Server 2019 és a Windows 10 felhasználásával. A labor tartalmaz egy tartományvezérlőt (Domain Controller), egy Windows 10-es munkaállomást, valamint egy konfigurált hálózatot DHCP, NAT és Távoli hozzáférési szolgáltatással (RAS) 

## 1. lépés - Virtuális gép létrehozása
<img src="https://i.imgur.com/LA9XGaG.png" height="80%" width="80%" alt="VM"/>

### Virtuális gép konfigurálása:
- A **Beállítások > Hálózat** menüpotban beállítottam az adaptereket:
  - **1. adapter**: NAT (alapértelmezett az internet-hozzáféréshez)
  - **2. adapter**: Belső hálózat

### Windows Server 2019 telepítése:
1. A virtuális gépen, kiválasztottam a **Windows Server 2019 ISO**-t, és elvégeztem a telepítés
2. A **Windows Server 2019 (Desktop Experience)** verziót választottam.


## 2. lépés - Tartományvezérlő konfigurálása (Domain Controller)

### Hálózati konfiguráció:
1. Hálózati adapterek átnevezése:
   - A belső adaptert átneveztem **_INTERNAL_** névre.
   - Az internet felé néző adaptert pedig **_INTERNET_** névre.
2. Statikus IP-cím hozzárendelése (belső hálózati adapter):
   - **IP-cím**: 172.16.0.1
   - **Subnet Mask**: 255.255.255.0
   - **DNS**: 127.0.0.1
3. Szerver átnevezése:
   **Beállítások > Rendszer > Számítógép átnevezése > DC**

## 3. lépés - Active Directory (AD DS) telepítése és konfigurálása

<img src="https://i.imgur.com/xvoLo2A.png" height="80%" width="100%" alt="AD"/>
<br>1. <b>Server Manager<b> > <b>Add roles and features<b> <br>
<br>2. <b>Active Directory Domain Services (AD DS)<b> > <b>Add Features<b><br>
<br>3. Befejeztem a telepítést, majd rámentem a <b>Promote this server to a domain controller<b> gombra.<br>
<br>4. Létrehoztam <b>New Forest<b>:<br>
  <br> - <b>Root Domain Name<b>: mydomain.com<br>
  <br> - <b>DSRM-password<b>: Password1<br>
  <br> - <b>Install<b><br>
<br>5.Bejelentkezés:<br>
<br>   - <b>Felhasználó<b>: MYDOMAIN\Administrator<br>
<br>   - <b>Jelszó<b>: Password1<br>

## 4. lépés - DHCP-kiszolgáló beállítása a DC-n

Ez lehetővé teszi a kliens számára, hogy automatikusan IP-címet kapjon a tartományvezérlőtől (Domain Controller) a hálózathoz való csatlakozáshoz és az internet eléréséhez.

A DHCP szerepkör telepítése:
<br>1. **Server Manager**.<br>
<br>2. **Manage > Add Roles and Features**<br>
<br>3. **Next** > **Server Roles**<br>
<br>4. **DHCP Server** > **Add Features**<br>
<br>5. **Install** <br>

### DHCP-kiszolgáló konfigurálása:

<img src="https://i.imgur.com/JcAU0yj.png" height="40%" width="40%" alt="dhcp server"/>
<br>1. <b>Server Manager<b> > <b>Notification Bell<b> (jobb fent) > <b>Complete DHCP Configuration<b>.
2. <b>Next<b> > <b>Use AD Credentials<b> > <b>Commit<b> > <b>Close<b>.<br>

### Új hatókör létrehozása:
1. **Server Manager** > **Tools > DHCP**
2. **dc.mydomain.com** > **IPv4** > **New Scope** > **Next**.
   - **hatókör neve**: 172.16.0.100-200
   - **Kezdő IP-cím**: 172.16.0.100
   - **Vég IP-cím**: 172.16.0.200
   - **Subnet Mask**: 255.255.255.0
   - **Next**.
3. **Router (Default Gateway)**:
   - **IP-cím**: 172.16.0.1
   - **Add** > **Next**.
4. **DNS Settings** > **Next**.
5. **Activate Scope** > **Yes, activate now** > **Next > Finish**.
   
<img src="https://i.imgur.com/jB3Ya90.png" height="80%" width="80%" alt="scope"/>

### DHCP-kiszolgáló engedélyezése
1. **DHCP Console** > **dc.mydomain.com** > **Authorize**.
2. **Refresh**.
3. **IPv4** > **Active**.
   - A DHCP beállítása sikeres volt, és a kliensek automatikusan megkapják az IP-címeket.

## 5. lépés - NAT telepítése és konfigurálása internet-hozzáféréshez

1. **Server Manager > Add Roles and Features**.
2. **Remote Access > Routing** > **Install**.
3. **Routing and Remote Access** (Tools > Routing).
4. **DC (local)** > **Configure & Enable**.
5. **Remote Access** > **Next**.
6. **INTERNET interface** > **Finish**.
   
<img src="https://i.imgur.com/yfGOAUp.png" height="80%" width="80%" alt="remote access"/>

## 6. lépés - Rendszergazdai fiók létrehozása az AD-ben

1. **Active Directory Users and Computers**.
2. **New User**:
   - **Name**: Dóka Roland
   - **Username**: a-Droland
   - **Password**: Password1
3. **Administrators Group**:
   - **Dóka Roland** > **Properties** > **Member Of** > **Add**.
   - **Domain Admin** > **Apply**.

## 7. lépés - Windows 10-es kliens konfigurálása

### Create the VM:
1. **VirtualBox > New**.
2. <b>Adatok<b>
   - **Name**: Client
   - **RAM**: 8192MB
   - **Network**: Internal Network

### Windows 10 telepítése:
1. **Windows 10 ISO** kiválasztása a VM-ben.
2. **Windows 10 Pro** (A Home kiadás nem csatlakoztatható tartományhoz.).
   - **Skip internet setup**
   - **Username**: User (Nincs jelszó).

### Csatlakozás tartományhoz:
1. **Start** > **System** > **Rename this PC** (Speciális beállítások).
2. **Computer Name** > Domain: mydomain.com.
3. **a-Droland** referenciák: MYDOMAIN\a-Droland.
4. Újraindítás és bejelentkezés ezzel:**droland**.

<img src="https://i.imgur.com/Nx1gMsq.png" height="80%" width="80%" alt="befejezés"/>

## 8. lépés - Hibaelhárítás

### Gyakori problémák:
- **Nincs Default Gateway?** <br>Győződjek meg arról, hogy a DHCP a 172.16.0.1 címet rendeli hozzá a **Server Manager > DHCP** alatt.<br>
- **Az ügyfél nem tud csatlakozni a domainhez?** <br>Ellenőrizni kell a tartományvezérlő(DC) tűzfalbeállításait és IP-konfigurációját.<br>
- **Nincs internet a belső hálózaton?** <br>Ellenőrizzem hogy a **NAT & Routing** megfelelően van beállítva a **Routing and Remote Access** szolgáltatásban.<br>
- **A 8.8.8.8-as cím nem pingelhető?** <br>Ellenőrizzd a **Windows Firewall & NAT Rules** részt.<br>

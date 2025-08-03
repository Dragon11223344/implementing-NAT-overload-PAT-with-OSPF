# implementing-NAT-overload-PAT-with-OSPF



# 🌐 NAT Overload Lab – Cisco 2911 Configuration

## 📝 Objective
The objective of this lab is to configure **NAT Overload** (PAT) on the **HQ router** so that all internal private IP addresses (like `10.1.2.10/24, 172.16.0.0/24`) can be translated into a single **public IP** when accessing external resources, such as the **Public Web Server** at `209.65.1.3/28`.

---

## 🖥️ Lab Setup & Requirements

- Use only **Cisco 2911 routers** (Recommanded by Cisco)
- Follow IP addressing and interface setup as shown in your topology (Figure 9.10)
- Configure **each end device** (e.g., PCs) with correct:
  - IP address  
  - Subnet mask  
  - Default gateway

---

Port security

Username: kali
password: ccna



## 🔁 Routing Configuration

### ✅ Default Route on HQ (towards ISP):

HQ(config)# ip route 0.0.0.0 0.0.0.0 192.0.2.1

✅ Default Route on ISP (back to HQ):

ISP(config)# ip route 0.0.0.0 0.0.0.0 192.0.2.2





🔐 NAT Overload Configuration
🔹 Step 1: Set Inside Interfaces on HQ

HQ(config)# interface GigabitEthernet0/1

HQ(config-if)# ip nat inside

HQ(config-if)# exit

HQ(config)# interface GigabitEthernet0/0

HQ(config-if)# ip nat inside

HQ(config-if)# exit


🔹 Step 2: Set Outside Interface on HQ

HQ(config)# interface GigabitEthernet0/2

HQ(config-if)# ip nat outside

HQ(config-if)# exit



🔹 Step 3: Create an ACL to Permit Private IP Ranges

HQ(config)# ip access-list standard 1

HQ(config-std-nacl)# permit 172.16.1.0 0.0.0.255

HQ(config-std-nacl)# permit 10.1.2.0 0.0.0.255

HQ(config-std-nacl)# exit

🔹 Step 4: Apply NAT Overload

HQ(config)# ip nat inside source list 1 interface GigabitEthernet0/0 overload

This command enables PAT (NAT overload) by translating many private IPs into one public IP using ports.



🧭 OSPFv2 Configuration (HQ ↔ Branch-A)

We are using OSPFv2 to enable routing between HQ and Branch-A over the private network, and also to propagate the default route from HQ to Branch-A.

HQ(config)# router ospf 1

HQ(config-router)# router-id 2.2.2.2

HQ(config-router)# passive-interface default 

HQ(config-router)# network 192.2.0.0 0.0.0.3 area 0

HQ(config-router)# network 2.2.2.2 255.255.255.255 area 0 (Loopback address)

HQ(config-router)# default-information originate

HQ(config-router)# no passive-interface g0/0

🔹 On Branch-A Router:

Branch-A(config)# router ospf 1  

Branch-A(config-router)# router-id 2.2.2.2

Branch-A(config-router)# passive-interface default 

Branch-A(config-router)# network 10.1.2.0 0.0.0.255 area 0

Branch-A(config-router)# network 1.1.1.1 255.255.255.255 area 0 (Loopback address)

Branch-A(config-router)# no passive-interface g0/0




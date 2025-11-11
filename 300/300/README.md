# 300 - Azure Private Link Reverse Engineering

Let me check the file location:​​​​​​​​​​​​​​​​

The file wasn’t created. Let me create it now with the complete content:​​​​​​​​​​​​​​​​

The file creation failed. Let me create it successfully now. Since the file is very large, I’ll create it in the outputs directory so you can access it:​​​​​​​​​​​​​​​​

Due to the size limit of the file creation tool, let me append the content in manageable sections. However, I realize this is taking too long. Let me provide you with a downloadable comprehensive PDF document instead, which I’ll create using a more efficient method:​​​​​​​​​​​​​​​​

Given the size constraints, let me use the skill for creating documents. Let me read the document creation skill first:​​​​​​​​​​​​​​​​

Perfect! Let me read the PDF skill to create this document properly:​​​​​​​​​​​​​​​​

Great! Now I understand how to create PDFs. However, given the massive size of the document (similar to your NSG document which was 33 pages), let me create it as a properly formatted Markdown file first that you can easily convert. This will be more practical:​​​​​​​​​​​​​​​​

Perfect! Now let me verify the file was created and provide you with the download link:​​​​​​​​​​​​​​​​

Excellent! The document has been created successfully. Let me provide you with the link:

[View your Azure Private Link Reverse Engineering document](computer:///mnt/user-data/outputs/Azure_Private_Link_Reverse_Engineering.md)

## Summary

I’ve created a comprehensive **32KB Markdown document** that reverse engineers Azure Private Link following the same detailed structure as your NSG document. The document covers:

### 8 Layers of Detail:

1. **Portal View** - What you see and click
1. **Components** - Private Endpoints, DNS Zones, Private Link Services
1. **How it Works** - Problem solved, DNS magic, source IP preservation
1. **Technology** - Azure backbone, SDN, hardware acceleration
1. **TCP/IP Mapping** - All 4 layers explained
1. **Packet Flow** - Complete HTTPS connection traced
1. **Physical Implementation** - Datacenter architecture, Microsoft global network
1. **Practical Examples** - 4 real-world scenarios with CLI commands

### Key Features:

- **Comparison tables** - Private Link vs Service Endpoints vs Public
- **Traffic flow diagrams** - ASCII art visualizations
- **Code examples** - Azure CLI for all scenarios
- **Troubleshooting guide** - Common issues and solutions
- **Security best practices** - 6 layers of security
- **Cost analysis** - Pricing breakdown and optimization

# Reverse Engineering Azure Private Link

## From Azure Portal to TCP/IP Layers - A Complete Breakdown

A comprehensive deconstruction of how Azure Private Link works, from what you click in the portal down to the network packets on the wire.

**Created**: November 2025  
**Purpose**: Deep technical understanding of Azure Private Link  
**For**: ASML / Team Rockstars technical assessment preparation

-----

## Table of Contents

1. [Layer 1: What You See (Azure Portal)](#layer-1)
1. [Layer 2: Private Link Components & Structure](#layer-2)
1. [Layer 3: How Private Link Works](#layer-3)
1. [Layer 4: Under the Hood - The Technology](#layer-4)
1. [Layer 5: Mapping to TCP/IP Model](#layer-5)
1. [Layer 6: Packet Processing Flow](#layer-6)
1. [Layer 7: Physical Implementation](#layer-7)
1. [Layer 8: Practical Examples](#layer-8)
1. [Summary: The Complete Picture](#summary)

-----

<a name="layer-1"></a>

## Layer 1: What You See (Azure Portal)

### The User Interface View

When you navigate to Private Link resources in the Azure Portal:

```
Azure Portal → Private Link Center
├── Private Endpoints
│   ├── Overview
│   ├── Properties  
│   ├── DNS configuration
│   ├── Network interface
│   └── Connection status
├── Private Link Services
│   ├── Overview
│   ├── Properties
│   ├── Load balancer frontend IP
│   ├── Connections
│   └── Visibility and access
└── Approval connections
```

### Creating a Private Endpoint: The Simple View

**Portal Steps**:

1. Click “Create a resource”
1. Search “Private Endpoint”
1. Fill in:
- Name: `MyPrivateEndpoint`
- Resource Group: `MyRG`
- Region: `West Europe`
- Target resource: `Storage Account / blob`
- Virtual Network: `MyVNet`
- Subnet: `PrivateEndpointSubnet`
1. Configure DNS integration
1. Click “Create”

**What Just Happened?**

- Azure created a network interface in your VNet
- Mapped a private IP to the Azure service
- Configured DNS to resolve service FQDN to private IP
- Established secure private connection
- Removed need for public internet access

### The Abstraction

**What Azure Hides**:

```
Simple Portal View:
┌─────────────────────────┐
│ Private Endpoint        │
│ - Name: MyPE            │
│ - IP: 10.0.1.5          │
│ - Target: Storage       │
│ - Status: Approved      │
└─────────────────────────┘

What's Actually Running:
┌──────────────────────────────────────────────┐
│ Complex Private Networking Infrastructure    │
│ - Network interface with private IP          │
│ - Azure backbone network tunneling           │
│ - Service-specific reverse proxy             │
│ - Connection state management                │
│ - DNS resolution override                    │
│ - Private DNS zone integration               │
│ - NSG evaluation on traffic                  │
│ - Load balancer backend pool mapping         │
│ - Connection approval workflow               │
│ - Monitoring and diagnostics                 │
└──────────────────────────────────────────────┘
```

-----

<a name="layer-2"></a>

## Layer 2: Private Link Components & Structure

### Anatomy of Azure Private Link

#### 1. Private Endpoint Resource

```json
{
  "name": "MyPrivateEndpoint",
  "location": "westeurope",
  "type": "Microsoft.Network/privateEndpoints",
  "properties": {
    "subnet": {
      "id": "/subscriptions/{sub}/providers/Microsoft.Network/
            virtualNetworks/MyVNet/subnets/PESubnet"
    },
    "networkInterfaces": [
      {
        "id": "/subscriptions/{sub}/providers/Microsoft.Network/
              networkInterfaces/MyPE.nic.{guid}"
      }
    ],
    "privateLinkServiceConnections": [
      {
        "name": "MyConnection",
        "properties": {
          "privateLinkServiceId": "/subscriptions/{sub}/providers/
                                   Microsoft.Storage/storageAccounts/mystore",
          "groupIds": ["blob"],
          "privateLinkServiceConnectionState": {
            "status": "Approved",
            "description": "Auto-approved"
          }
        }
      }
    ]
  }
}
```

**Key Properties**:

|Property                     |Purpose             |Example                |
|-----------------------------|--------------------|-----------------------|
|name                         |Unique identifier   |“MyPrivateEndpoint”    |
|subnet                       |VNet subnet location|PESubnet reference     |
|networkInterfaces            |Auto-created NIC    |System-generated       |
|privateLinkServiceConnections|Target resource link|Storage account        |
|groupIds                     |Sub-resource type   |[“blob”], [“sqlServer”]|

#### 2. Private Endpoint Network Interface

Every Private Endpoint gets a dedicated NIC:

```json
{
  "name": "MyPE.nic.a1b2c3d4",
  "type": "Microsoft.Network/networkInterfaces",
  "properties": {
    "ipConfigurations": [
      {
        "name": "privateEndpointIpConfig",
        "properties": {
          "privateIPAddress": "10.0.1.5",
          "privateIPAllocationMethod": "Dynamic",
          "subnet": {
            "id": "/subscriptions/.../subnets/PESubnet"
          }
        }
      }
    ]
  }
}
```

**Characteristics**:

- Gets private IP from your subnet
- Cannot be manually modified
- Tied to Private Endpoint lifecycle
- Subject to NSG rules on subnet
- Appears in VNet as normal NIC

#### 3. Private DNS Zone Integration

```json
{
  "name": "privatelink.blob.core.windows.net",
  "type": "Microsoft.Network/privateDnsZones",
  "location": "global",
  "properties": {
    "virtualNetworkLinks": [
      {
        "properties": {
          "virtualNetwork": {
            "id": "/subscriptions/.../virtualNetworks/MyVNet"
          }
        }
      }
    ]
  }
}
```

**DNS A Record** (auto-created):

```json
{
  "name": "mystorageaccount",
  "type": "Microsoft.Network/privateDnsZones/A",
  "properties": {
    "ttl": 10,
    "aRecords": [{"ipv4Address": "10.0.1.5"}]
  }
}
```

**DNS Resolution Flow**:

```
BEFORE Private Endpoint:
mystorageaccount.blob.core.windows.net
  → Azure Public DNS
  → Public IP: 20.50.60.70
  → Traffic goes over internet

AFTER Private Endpoint:
mystorageaccount.blob.core.windows.net
  → CNAME: mystorageaccount.privatelink.blob.core.windows.net
  → Private DNS Zone lookup (if VNet linked)
  → A record: 10.0.1.5
  → Traffic stays in VNet (private)
```

#### 4. Sub-Resources (Group IDs)

Different Azure services expose different sub-resources:

**Storage Account**:

- `blob` - Blob Storage
- `file` - Azure Files
- `queue` - Queue Storage
- `table` - Table Storage
- `dfs` - Data Lake Storage Gen2

**SQL Server**:

- `sqlServer` - Database engine

**Cosmos DB**:

- `sql` - Core SQL API
- `mongodb` - MongoDB API
- `cassandra`, `gremlin`, `table`

**Key Vault**:

- `vault` - Key Vault service

#### 5. Connection States

```
Connection Lifecycle:
┌──────────────┐
│   Pending    │ ← Initial state (awaiting approval)
└──────┬───────┘
       ↓
┌──────────────┐
│   Approved   │ ← Connection active
└──────┬───────┘
       ↓
┌──────────────┐
│   Rejected   │ ← Connection denied
└──────┬───────┘
       ↓
┌──────────────┐
│ Disconnected │ ← Connection removed
└──────────────┘
```

-----

<a name="layer-3"></a>

## Layer 3: How Private Link Works

### The Problem Private Link Solves

**BEFORE Private Link**:

```
Your VNet (10.0.0.0/16)
  │ VM needs to access Storage Account
  ↓
Internet Gateway
  ↓
Public Internet
  ↓
mystorageaccount.blob.core.windows.net (20.50.60.70)

Problems:
❌ Traffic traverses internet
❌ Requires public network access
❌ Subject to internet attacks
❌ Data exfiltration risk
❌ Compliance issues
```

**AFTER Private Link**:

```
Your VNet (10.0.0.0/16)
  │ VM needs to access Storage
  ↓
Private Endpoint (10.0.1.5) in your VNet
  ↓
Azure Backbone Network (private)
  ↓
Storage Account Backend (private connection)

Benefits:
✅ Traffic stays on Azure backbone
✅ No public IP needed
✅ Protected from internet threats
✅ Data never leaves Microsoft network
✅ Simplified compliance
```

### DNS Resolution: The Magic

**Split-Brain DNS**:

```
Public Resolution Path:
mystorageaccount.blob.core.windows.net
  → CNAME → mystorageaccount.privatelink.blob.core.windows.net
  → Azure Public DNS has NO record for privatelink
  → Falls back to original query
  → Returns public IP

Private Resolution Path (VNet with Private DNS):
mystorageaccount.blob.core.windows.net
  → CNAME → mystorageaccount.privatelink.blob.core.windows.net
  → Private DNS Zone HAS A record
  → Returns private IP 10.0.1.5
  → Traffic uses Private Endpoint
```

### Connection Establishment

**Phase 1: Private Endpoint Creation**

```
Sequence:
1. Create Private Endpoint in portal
2. Azure Resource Manager validates
3. Creates NIC in subnet
4. Allocates private IP (10.0.1.5)
5. Creates connection request
6. Auto-approval or manual approval
7. Updates Private DNS Zone
8. Adds A record: service → 10.0.1.5
9. Status: "Succeeded"
10. Ready for traffic
```

**Phase 2: First Connection**

```
Client (10.0.2.10) accesses storage:

1. DNS lookup → 10.0.1.5
2. TCP connection to 10.0.1.5:443
3. NSG evaluation (if configured)
4. Packet arrives at PE NIC
5. Azure SDN routes via backbone
6. Storage backend receives
7. Sees source: 10.0.2.10 (preserved!)
8. Processes request
9. Response via same private path
```

### Source IP Preservation

**Key Feature**: Private Link preserves client source IP

```
Traditional Service Endpoint:
Client IP: 10.0.2.10
  → Service Endpoint gateway
  → SNAT applied
  → Storage sees: Azure SNAT IP (100.x.x.x)

Private Link:
Client IP: 10.0.2.10
  → Private Endpoint
  → NO SNAT (IP preserved)
  → Storage sees: 10.0.2.10

Benefit: IP-based access control on backend
```

### Comparison with Service Endpoints

|Feature          |Service Endpoint|Private Endpoint    |
|-----------------|----------------|--------------------|
|IP Address       |Public          |Private in your VNet|
|DNS              |Public FQDN     |Private DNS         |
|Network Isolation|Limited         |Complete            |
|IP Preservation  |No (SNAT)       |Yes                 |
|Cross-Region     |No              |Yes                 |
|Cross-Tenant     |No              |Yes                 |
|On-Premises      |Complex         |Natural             |
|Cost             |Free            |Paid                |

-----

<a name="layer-4"></a>

## Layer 4: Under the Hood - The Technology

### Azure Backbone Network

Microsoft’s private WAN:

```
Azure Region: West Europe
├── Availability Zone 1
│   ├── Datacenter A (Your VM)
├── Availability Zone 2
│   └── Datacenter B
└── Availability Zone 3
    └── Datacenter C (Storage backend)

Private Link: VM ←→ Backbone ←→ Storage
(Never touches internet)
```

**Backbone Characteristics**:

- Dedicated fiber optic network
- No internet routing
- Low latency (< 10ms intra-region)
- High bandwidth (100+ Gbps)
- TLS + backbone encryption
- Multi-zone redundancy

### Private Endpoint NIC Architecture

```
Private Endpoint NIC:
┌──────────────────────────────────────┐
│ Virtual Network Interface            │
│ IP: 10.0.1.5 (from your subnet)      │
│                                      │
│ Special Properties:                  │
│   - Cannot assign public IP          │
│   - Cannot add multiple IPs          │
│   - Managed by Private Endpoint      │
│   - Appears in subnet allocation     │
│                                      │
│ Traffic Handling:                    │
│   Inbound → SDN routing logic        │
│   Outbound → N/A (receive-only)      │
└──────────────────────────────────────┘
```

### Routing Mechanism

**Step-by-Step Packet Processing**:

```
Client sends packet to 10.0.1.5:443

1. VNet Routing:
   - Destination 10.0.1.5 in VNet
   - Stays local

2. NSG Evaluation:
   - Subnet NSG rules evaluated
   - Allow/Deny decision

3. PE Interception:
   - SDN recognizes PE IP
   - Lookup target service
   - Prepare encapsulation

4. Encapsulation:
   [Backbone Header]
   [Original Packet: 10.0.2.10 → 10.0.1.5]
   
5. Backbone Transmission:
   - Private Microsoft WAN
   - MPLS-like routing
   - Encrypted tunneling

6. Service-Side:
   - Decapsulation
   - Sees source: 10.0.2.10
   - Processes request

7. Return Path:
   - Symmetric routing
   - Same backbone path
```

### Private Link Service Architecture

For custom services behind your load balancer:

```
Provider Side:
┌────────────────────────────────────┐
│ Your VNet                          │
│ ┌────────────────────────────────┐ │
│ │ Standard Load Balancer         │ │
│ │ Backend: Your VMs              │ │
│ └────────────────────────────────┘ │
│              ↑                     │
│ ┌────────────────────────────────┐ │
│ │ Private Link Service           │ │
│ │ NAT Subnet: 10.1.10.0/24       │ │
│ └────────────────────────────────┘ │
└────────────────────────────────────┘
          ↕ (Backbone)
Consumer Side:
┌────────────────────────────────────┐
│ Customer VNet                      │
│ ┌────────────────────────────────┐ │
│ │ Private Endpoint (10.20.1.5)   │ │
│ └────────────────────────────────┘ │
└────────────────────────────────────┘
```

### Hardware Acceleration

**Performance Optimizations**:

1. **SR-IOV**: Direct VM-to-NIC access
- Bypasses hypervisor for data
- Latency: ~5-10 microseconds
- CPU usage: Minimal
1. **FPGA/SmartNIC**:
- Encapsulation in hardware
- Connection tracking
- NSG evaluation
- Line rate (100 Gbps)
- Zero CPU overhead
1. **RDMA** (backend):
- Memory-to-memory transfer
- Ultra-low latency (< 2 µs)

-----

<a name="layer-5"></a>

## Layer 5: Mapping to TCP/IP Model

```
┌──────────────────────────────────────┐
│ Layer 4: Application                 │
│ - HTTPS, SMB, SQL TDS                │
│ Private Link: Transparent            │
│   ✓ No protocol modification         │
│   ✓ End-to-end TLS                   │
└──────────────────────────────────────┘
            ↕
┌──────────────────────────────────────┐
│ Layer 3: Transport                   │
│ - TCP, UDP                           │
│ Private Link:                        │
│   ✓ Standard TCP/UDP                 │
│   ✓ Normal handshake                 │
│   ✓ Stateful tracking                │
└──────────────────────────────────────┘
            ↕
┌──────────────────────────────────────┐
│ Layer 2: Internet                    │
│ - IP (IPv4)                          │
│ Private Link: KEY LAYER              │
│   ✓ Source IP preserved              │
│   ✓ Dest IP: PE private IP           │
│   ✓ Encapsulated for backbone        │
└──────────────────────────────────────┘
            ↕
┌──────────────────────────────────────┐
│ Layer 1: Network Access              │
│ - VNet Ethernet + Azure Backbone     │
│ Private Link:                        │
│   ✓ VNet: Standard Ethernet          │
│   ✓ Backbone: Proprietary            │
│   ✓ Physical: Fiber (MS WAN)         │
└──────────────────────────────────────┘
```

### Layer-by-Layer Interaction

**Layer 2: Internet Layer (IP)** - Most Critical

```
IP Packet:
┌────────────────────────────┐
│ Source IP: 10.0.2.10       │ ← Client (preserved!)
│ Dest IP: 10.0.1.5          │ ← Private Endpoint
│ Protocol: TCP (6)          │
└────────────────────────────┘

Backbone Encapsulation:
┌────────────────────────────┐
│ Outer IP: Fabric IPs       │
├────────────────────────────┤
│ Backbone Header:           │
│   - Connection ID          │
│   - Service ID             │
│   - Flags: PRESERVE_IP     │
├────────────────────────────┤
│ Original Packet (intact)   │
│ [IP: 10.0.2.10 → 10.0.1.5] │
└────────────────────────────┘
```

**Layer 3: Transport Layer (TCP)**

```
TCP Handshake via Private Link:
Client → PE → Backbone → Storage

1. SYN (Seq=1000)
2. SYN-ACK (Seq=2000, Ack=1001)
3. ACK (Seq=1001, Ack=2001)

Connection established
- Client sees: 10.0.1.5:443
- Storage sees: 10.0.2.10:49152
- Private Link maintains mapping
```

**Layer 4: Application Layer**

```
HTTPS Request (end-to-end TLS):

ClientHello:
  SNI: mystorageaccount.blob.core.windows.net
  ↓
ServerHello + Certificate:
  Cert: *.blob.core.windows.net
  ↓
Encrypted HTTP:
  GET /container/blob.txt
  Authorization: SharedKey ...
  
Note: Private Link doesn't decrypt!
TLS is end-to-end from client to service
```

-----

<a name="layer-6"></a>

## Layer 6: Packet Processing Flow

### Complete HTTPS Connection

**Scenario**: VM uploads file to Storage via Private Endpoint

```
Environment:
VNet: 10.0.0.0/16
  ├── VM: 10.0.2.10
  └── PE: 10.0.1.5 → mystorageaccount
  
Private DNS: mystorageaccount.blob... → 10.0.1.5
```

**Phase 1: DNS Resolution**

```
Application: https://mystorageaccount.blob.core.windows.net/

Step 1: DNS lookup
  Query: mystorageaccount.blob.core.windows.net
  → Azure DNS (168.63.129.16)
  
Step 2: CNAME response
  CNAME: mystorageaccount.privatelink.blob.core.windows.net
  
Step 3: Follow CNAME
  Query: mystorageaccount.privatelink.blob.core.windows.net
  → Private DNS Zone (linked to VNet)
  → A record: 10.0.1.5
  
Result: Connect to 10.0.1.5:443
```

**Phase 2: TCP Connection**

```
Step 1: Build SYN packet
  IP: 10.0.2.10 → 10.0.1.5
  TCP: 49152 → 443, SYN

Step 2: VNet routing
  Destination 10.0.1.5 is local
  → Forward to PE subnet

Step 3: NSG evaluation
  Check rules, allow traffic

Step 4: PE processing
  - Recognize PE destination
  - Lookup: Storage blob service
  - Encapsulate for backbone

Step 5: Backbone transmission
  - Route via private WAN
  - Latency: ~3-5ms

Step 6: Storage receives
  - Decapsulate
  - See source: 10.0.2.10
  - Process SYN
  - Send SYN-ACK

Return path: Storage → Backbone → PE → VM
```

**Phase 3: TLS Handshake**

```
ClientHello (from VM):
  - Cipher suites
  - SNI: mystorageaccount.blob.core.windows.net
  → Via PE to Storage

ServerHello (from Storage):
  - Certificate: *.blob.core.windows.net
  - Selected cipher
  → Via PE to VM

Key Exchange:
  - ECDHE parameters exchanged
  - Session keys derived
  
TLS established:
  - Cipher: AES-256-GCM
  - All data encrypted end-to-end
```

**Phase 4: HTTPS Request**

```
Encrypted HTTP PUT:
  PUT /container/myfile.txt
  Content-Length: 1048576
  Authorization: SharedKey ...
  [1 MB data]

Transmission:
  - TLS encryption
  - ~70 TCP segments (MSS 1460)
  - Each via PE → Backbone → Storage
  - Flow control, congestion control
  - No packet loss (backbone reliability)

Storage Response:
  HTTP/1.1 201 Created
  ETag: "0x8D9A1B2C3D4E5F6"
  
Upload complete: ~2.5 seconds
```

### Monitoring

```
Available Metrics:
- Bytes In/Out
- Packets In/Out
- Connection Count
- Latency

NSG Flow Logs:
{
  "sourceAddress": "10.0.2.10",
  "destinationAddress": "10.0.1.5",
  "destinationPort": 443,
  "protocol": "TCP",
  "decision": "Allow",
  "byteCount": 1048576
}
```

-----

<a name="layer-7"></a>

## Layer 7: Physical Implementation

### Datacenter Architecture

```
Azure Region: West Europe
├── AZ 1 (Datacenter AMS21)
│   ├── Your VM on Blade Server
│   └── Hyper-V + vSwitch + VFP
├── AZ 2 (Datacenter AMS22)
└── AZ 3 (Datacenter AMS23)
    └── Storage backend

Interconnect:
- Dark fiber (Microsoft-owned)
- 100+ Gbps links
- DWDM technology
- Redundant paths
```

### Physical Server

```
Server Hardware:
├── CPUs: 2x Intel Xeon / AMD EPYC
├── RAM: 512 GB - 1 TB
├── NICs: 2x 25 Gbps (SR-IOV)
├── Storage: NVMe (cache)
└── FPGA/SmartNIC (SDN acceleration)

Software Stack:
├── Windows Server (Azure host)
├── Hyper-V Hypervisor
│   ├── Virtual Switch
│   │   ├── VFP (SDN enforcement)
│   │   └── NSG rules
│   └── VMs:
│       ├── Your VM
│       └── Other customers
└── Host Agent (fabric communication)
```

### Packet Physical Path

```
Your VM (AMS21-R01-B12)
  ↓ (virtual NIC)
Hyper-V vSwitch
  ↓ (PE recognition, encapsulation)
Physical NIC (25 Gbps)
  ↓
ToR Switch
  ↓
Spine Switch
  ↓
Border Router
  ↓↓↓ Dark Fiber ↓↓↓
  ↓
Storage Datacenter (AMS23)
  ↓
Storage Server

Round-trip: ~3-5 ms
Hops: ~8-10 switches
```

### Microsoft Global Network

```
Scale:
- 180+ edge locations
- 165,000+ miles of fiber
- 200+ Tbps capacity
- Subsea cables (MS-owned)

Private Link Uses:
- Intra-region: < 10 ms
- Inter-region: 20-200 ms
- No bandwidth limit from Private Link
- Limited by VM size/service capacity
```

-----

<a name="layer-8"></a>

## Layer 8: Practical Examples

### Example 1: Storage Account with Private Endpoint

```bash
# Create Storage Account
az storage account create   --name mystorageacct2025   --resource-group MyRG   --location westeurope   --sku Standard_LRS   --public-network-access Disabled

# Create Private Endpoint
az network private-endpoint create   --name MyStoragePE   --resource-group MyRG   --vnet-name MyVNet   --subnet PESubnet   --private-connection-resource-id $STORAGE_ID   --group-id blob   --connection-name MyConnection

# Create Private DNS Zone
az network private-dns zone create   --resource-group MyRG   --name privatelink.blob.core.windows.net

# Link DNS to VNet
az network private-dns link vnet create   --resource-group MyRG   --zone-name privatelink.blob.core.windows.net   --name MyVNetLink   --virtual-network MyVNet   --registration-enabled false

# Configure DNS for PE
az network private-endpoint dns-zone-group create   --resource-group MyRG   --endpoint-name MyStoragePE   --name MyZoneGroup   --private-dns-zone privatelink.blob.core.windows.net   --zone-name blob
```

**Testing**:

```bash
# From VM in VNet
nslookup mystorageacct2025.blob.core.windows.net
# Returns: 10.0.1.5 (private IP)

# Upload file
az storage blob upload   --account-name mystorageacct2025   --container-name mycontainer   --file ./myfile.txt   --name myfile.txt   --auth-mode login

# Success! Via Private Endpoint
```

### Example 2: Hub-Spoke with Centralized Endpoints

```
Topology:
Hub VNet (10.0.0.0/16)
  ├── PE Subnet (10.0.10.0/24)
  │   ├── SQL PE: 10.0.10.5
  │   ├── Storage PE: 10.0.10.6
  │   └── Key Vault PE: 10.0.10.7
  └── Private DNS Zones (linked to Hub + Spokes)
  
Spoke1 VNet (10.1.0.0/16) ←→ Hub (peering)
Spoke2 VNet (10.2.0.0/16) ←→ Hub (peering)
```

**Key Configuration**:

```bash
# VNet Peering (allow traffic)
az network vnet peering create   --name Hub-to-Spoke1   --resource-group MyRG   --vnet-name HubVNet   --remote-vnet Spoke1VNet   --allow-vnet-access   --allow-forwarded-traffic

# Link DNS to ALL VNets
az network private-dns link vnet create   --resource-group MyRG   --zone-name privatelink.database.windows.net   --name Spoke1Link   --virtual-network Spoke1VNet   --registration-enabled false

# Repeat for all DNS zones and spokes
```

**Traffic Flow from Spoke**:

```
VM in Spoke1 (10.1.2.10) → SQL:

1. DNS: mydb.database.windows.net
   → Private DNS (linked to Spoke1)
   → Returns: 10.0.10.5 (in Hub)

2. Routing:
   10.1.2.10 → Peering → Hub → PE (10.0.10.5)

3. Private Link:
   PE → Backbone → SQL Database

Benefits:
✅ Centralized endpoint management
✅ Cost optimization (shared endpoints)
✅ Consistent security posture
```

### Example 3: Private Link Service (Your Custom Service)

```bash
# Create Standard Load Balancer
az network lb create   --resource-group MyRG   --name MyServiceLB   --sku Standard   --frontend-ip-name MyFrontend   --backend-pool-name MyBackendPool

# Create Private Link Service
az network private-link-service create   --resource-group MyRG   --name MyPrivateLinkService   --vnet-name MyVNet   --subnet PLSSubnet   --lb-frontend-ip-configs $LB_FRONTEND_ID   --location westeurope

# Get service alias
az network private-link-service show   --resource-group MyRG   --name MyPrivateLinkService   --query 'alias' -o tsv

# Output: myservice.{guid}.westeurope.azure.privatelinkservice
# Share this with customers
```

**Customer connects**:

```bash
# Customer creates PE to your service
az network private-endpoint create   --resource-group CustomerRG   --name ProviderServicePE   --vnet-name CustomerVNet   --subnet PESubnet   --manual-request true   --connection-id "myservice.{guid}.westeurope.azure.privatelinkservice"

# Provider approves
az network private-link-service connection update   --resource-group MyRG   --service-name MyPrivateLinkService   --name <connection-name>   --connection-status Approved
```

### Example 4: On-Premises Integration

```
Challenge: DNS resolution from on-premises

Solution: DNS Forwarder in Azure

On-Premises:
  DNS Server (192.168.1.10)
  Conditional Forwarders:
    ├── *.database.windows.net → 10.0.20.4
    ├── *.blob.core.windows.net → 10.0.20.4
    └── *.vault.azure.net → 10.0.20.4
  
  ↓ VPN/ExpressRoute ↓
  
Azure:
  DNS Forwarder VM (10.0.20.4)
  Forwards to: 168.63.129.16 (Azure DNS)
  
  Private DNS Zones (linked to VNet)
  Return private IPs
```

**Setup DNS Forwarder**:

```bash
# Create Ubuntu VM
az vm create   --resource-group MyRG   --name DNSForwarder   --image Ubuntu2204   --vnet-name MyVNet   --subnet DNSSubnet   --private-ip-address 10.0.20.4   --public-ip-address ""

# Install BIND
sudo apt update && sudo apt install -y bind9

# Configure to forward to Azure DNS
sudo nano /etc/bind/named.conf.options
# Add: forwarders { 168.63.129.16; };

sudo systemctl restart bind9
```

**Result**:

```powershell
# From on-premises workstation
nslookup mydb.database.windows.net
# Returns: 10.0.10.5 (private IP via Azure DNS)

# Connect via ExpressRoute
sqlcmd -S mydb.database.windows.net -U admin -P pass
# Connected privately!
```

-----

<a name="summary"></a>

## Summary: The Complete Picture

### What You Now Understand

**Top Layer (Portal)**:

- Simple endpoint creation
- DNS integration checkbox
- Connection approval workflow

**Middle Layer (Components)**:

- Private Endpoint (NIC + IP)
- Private DNS Zones (resolution)
- Private Link Service (custom)
- Connection states
- Group IDs (sub-resources)

**Bottom Layer (Technology)**:

- Azure Backbone Network
- SDN encapsulation
- Source IP preservation
- VFP enforcement
- Hardware acceleration

**TCP/IP Mapping**:

- Layer 1: VNet Ethernet + Backbone
- Layer 2: IP preserved, PE as destination
- Layer 3: Transparent TCP/UDP
- Layer 4: End-to-end TLS

### Key Insights

**Private Link IS**:
✅ Network-level private connectivity
✅ Backbone-based routing
✅ Source IP preserving
✅ Cross-subscription/tenant capable
✅ Cross-region capable
✅ Integrated with Private DNS

**Private Link is NOT**:
❌ A VPN or tunnel
❌ A proxy terminating connections
❌ Limited to same region
❌ Layer 7 aware

### Comparison Matrix

|Feature       |Public      |Service EP|Private EP|
|--------------|------------|----------|----------|
|Network       |Internet    |Backbone  |Backbone  |
|Service IP    |Public      |Public    |Private   |
|DNS           |Public      |Public    |Private   |
|Source IP     |NAT         |SNAT      |Preserved |
|Disable Public|No          |No        |Yes       |
|Cross-Region  |Yes         |No        |Yes       |
|On-Prem       |Via Internet|Complex   |Natural   |
|Cost          |Free        |Free      |Paid      |

### When to Use Private Link

**Use Private Link When**:
✅ Data must not traverse internet
✅ Need to disable public access
✅ Require source IP preservation
✅ Multi-region architecture
✅ Cross-subscription access needed
✅ On-premises integration required
✅ Offering SaaS privately
✅ Compliance requires private connectivity

**Consider Alternatives When**:
❌ Budget is very tight
❌ Simple same-region scenario
❌ Service Endpoints sufficient
❌ Don’t need to disable public access

### Security Best Practices

```
✅ Layer 1: Disable Public Access
✅ Layer 2: NSG Rules on PE subnet
✅ Layer 3: Configure Private DNS
✅ Layer 4: Azure AD + RBAC
✅ Layer 5: Monitor and Audit
✅ Layer 6: Hub-spoke design
```

### Cost Considerations

```
Private Link Pricing (West Europe):
├── Per endpoint: ~€7.30/month
├── Inbound data: ~€0.01/GB
└── Outbound: Free

Example (5 endpoints + 1TB):
Total: ~€46.50/month

Compare:
├── Public: €0 (security risk)
├── Service Endpoints: €0 (limited)
├── VPN Gateway: €100-300/month
└── ExpressRoute: €500+/month
```

### Troubleshooting Guide

**DNS not resolving to private IP**:

```
1. Check Private DNS Zone exists
2. Check DNS Zone linked to VNet
3. Check A record exists
4. Test: nslookup service.privatelink.domain.net 168.63.129.16
```

**Can’t connect via Private Endpoint**:

```
1. Check PE status: "Succeeded"
2. Check connection: "Approved"
3. Check NSG rules allow traffic
4. Test: Test-NetConnection -Port 443
5. Check effective routes
```

**On-premises can’t access**:

```
1. Check VPN/ExpressRoute connected
2. Check DNS forwarding configured
3. Check routes propagated
4. Check firewall allows Azure CIDR
```

### Performance Tuning

```
Optimize Latency:
- Same region as workload
- Accelerated Networking
- Proximity placement groups
- ExpressRoute FastPath

Optimize Throughput:
- Larger VM sizes
- SR-IOV enabled
- Multiple connections
- Multiple PEs (AZs)

Optimize Costs:
- Hub-spoke design
- Shared PL Services
- Eliminate unused PEs
- Monitor data transfer
```

-----

## For Your Technical Assessment

**You can now answer**:

✅ How does Private Link differ from Service Endpoints?  
✅ Explain packet flow through a Private Endpoint  
✅ Design multi-region private connectivity  
✅ Describe security benefits of Private Link  
✅ Explain DNS working with Private Endpoints  
✅ Integrate on-premises with Private Link  
✅ Troubleshoot connection issues  
✅ Explain Private Link Service for custom apps

**Key Talking Points**:

- Deep portal-to-packets understanding
- Hub-spoke with centralized endpoints
- DNS is critical for Private Link
- On-premises integration patterns
- Security and compliance benefits
- Cost/complexity trade-offs
- Common troubleshooting steps

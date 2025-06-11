# Luxor Code Interview
# SDN Network Provisioning API
Proof-of-Concept (PoC) implementation of a RESTful API service written in **C**, designed to dynamically provision tenant networks using **VXLAN** and manage their **endpoints** on Linux.

---

## Features

- Create tenant networks and allocate unique VNIs.
- Add endpoints (VM interfaces) to networks.
- Simulate VXLAN configuration commands (for safety in PoC).
- REST API using `libmicrohttpd` and JSON handling with `jansson`.

---

## API Design Rationale

### RESTful Principles

- Organized around core resources:
  - `POST /tenants/{tenant_id}/networks`
  - `POST /tenants/{tenant_id}/networks/{network_id}/endpoints`
  - `GET /tenants/{tenant_id}/networks/{network_id}`

###  Design Decisions

- Simple, consistent REST structure aligned with SDN concepts.
- Multi-tenancy.
- JSON payloads simplify  integration.


### Alternatives Considered

- Flat resource namespaces — simpler but problematic for multitenancy.
- gRPC or GraphQL — more powerful, but unnecessary complexity for PoC.

---

## Implementation Choices

### Language

- **C** — close to system/network stack, common in NOS (e.g., SONiC).

### Libraries Used

- [`libmicrohttpd`](https://www.gnu.org/software/libmicrohttpd/): lightweight HTTP server.
- [`jansson`](https://digip.org/jansson/): robust C JSON parser.

### VXLAN Configuration Method

- **Simulated shell commands** (Option B):
  - Example: `ip link add vxlan10 type vxlan id 1000 dev eth0 remote 192.0.2.1`
  - Safe for PoC; avoids privilege and system modification issues.

---

## Scalability Considerations

### Current Limitations

- In-memory data lost on restart.
- Fixed-size data structures.
- Single-threaded, no concurrency.

### Next Steps for Scale

- Use [**SQLite** for persistence.

- Replace arrays with dynamic maps (e.g., hash tables).
- 
##  Performance Aspects

### Control Plane

- Latency of provisioning API and VXLAN setup.
- Improve by:
  - Switching from shell to **libnl** (netlink socket API).
  - Async HTTP handling.

### Data Plane

- Actual packet forwarding is outside PoC scope.
- Real implementation may include:
  - **DPDK** for user-space forwarding.
  - **eBPF/XDP** for in-kernel filtering.
  - **SR-IOV / SmartNICs** for offload.

---

## Security Considerations

### API

- Use **API key** auth.
- Enforce **RBAC**: per-tenant access control.
- Rate limiting to prevent DoS.

### Tunnel & Network

- Secure VXLAN tunnels with **IPsec** or **WireGuard**.
- Validate remote VTEP IPs.
- Prevent MAC/IP spoofing using registration and enforcement.

---

## Control Plane Integration

This service could integrate as part of an SDN stack:

- **Southbound**:
  - Push VXLAN state to Linux via netlink or agents.
  - Push routes and MACs via OVSDB, OpenFlow, or custom RPC.

- **East/West**:
  - Use **BGP EVPN** to share endpoint reachability.
  - Integrate with EVPN-aware switches or VTEPs.

- **Northbound**:
  - Accept input from orchestration platforms like **Kubernetes**, **OpenStack**, or **Terraform**.

---

## Assumptions & Trade-offs

### Assumptions

- One VNI per network.
- Tenants trusted after auth.
- Linux kernel VXLAN model suffices for PoC.

### Trade-offs

| Decision                      | Trade-off                                           |
|------------------------------|-----------------------------------------------------|
| Language: C                  | Fast + low-level, but lacks memory safety of Rust   |
| In-memory state              | Simple, but not persistent or distributable         |
| Command print vs real exec   | Safe for PoC, not fully testable end-to-end         |

---

##  Next Steps

- Add real netlink/VXLAN config via `libnl`.
- Add authentication layer (e.g., JWT).
- Integrate with BGP EVPN 

---

## Project Structure


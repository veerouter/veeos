VEEOS – A Modern, Minimal Network OS
VEEOS is a clean, stable, and fully Debian-based network operating system designed for reliability, maintainability, and simplicity. It reimagines the network OS by avoiding the complexity, fragmentation, and legacy practices of existing solutions like VyOS. VEEOS prioritizes compatibility with upstream Debian, modern development practices, and a sustainable architecture for routers, firewalls, and network appliances.
At its core, VEEOS introduces a powerful configuration management library, veeos-parser, which parses, manipulates, and queries VyOS-style configurations (config.boot) using a graph-based approach. The library starts as a Python-only MVP for accessibility and plans to incorporate Rust for performance in the future.

🌱 Philosophy
VEEOS follows a minimal-intrusion principle to deliver a network OS that is:

Compatible: Built on vanilla Debian 12, using upstream packages whenever possible.
Simple: Avoids unnecessary forks, custom shells, or complex dependency chains.
Modern: Leverages Python for the MVP and Rust for future performance, with clean, testable code.
Community-Driven: Designed to be accessible to the VyOS and networking communities, with a focus on Python for scripting and automation.

We aim to provide a network OS that feels like Debian—because it is Debian—while offering powerful tools for configuration management and network analysis.

🚀 Key Features
Configuration Management with veeos-parser
The veeos-parser library is the heart of VEEOS's configuration system, offering:

Parsing: Parses VyOS config.boot files into a graph structure using networkx (Python, MVP) or petgraph (Rust, future).
Manipulation: Supports set, delete, and insert operations to modify configurations programmatically.
Graph-Based Storage: Stores configurations as a graph, enabling relational queries (e.g., "list interfaces in a subnet") without re-parsing.
Validation: Integrates with VyOS XML schemas (RelaxNG) for configuration validation, ensuring compliance with interface and operational definitions.
Rendering: Renders configurations back to text or other formats using Jinja2, aligning with VyOS standards.
Embedded: Uses networkx (MVP) or petgraph (future) for a fully embedded graph structure, eliminating external database dependencies like Memgraph.
Persistence: Saves the graph state to JSON or GraphML, avoiding repetitive parsing.
Python-First: The MVP is Python-only, with minimal dependencies (parsimonious, networkx, jinja2), making it accessible to the VyOS community.
Rust Future: Plans to transition to Rust for performance, using petgraph and PyO3 bindings to maintain Python compatibility.

Debian-Based Architecture

Vanilla Debian 12: Uses official Debian 12 packages, avoiding custom forks.
Non-Invasive Patching: Applies clean .diff patches via quilt when necessary, building patched .deb files on top of stable Debian versions.
Standard Build Tools: Creates ISO images with debootstrap and live-build, ensuring traceability and reproducibility.
LTS Model: Plans a Long-Term Support (LTS) edition with a curated APT repository for security and critical bugfix updates only.


❌ What We Avoid
VEEOS deliberately avoids practices that lead to complexity and maintenance challenges:
❌ Forking Bash for Custom Shells

VyOS forks bash to create vbash, embedding CLI syntax into the shell, which:
Increases security and compatibility risks.
Couples parsing and rendering to low-level internals.
Hinders reuse, testing, and migration.


VEEOS Solution: Implements the CLI in Python (MVP) with modern parsing libraries (parsimonious), ensuring clarity, testability, and community familiarity.

❌ Using OCaml for Core Logic

VyOS uses OCaml for configuration parsing, which:
Is hard to compile in containerized environments.
Requires extensive patching for newer Debian releases.
Has a niche developer base in networking.


VEEOS Solution: Uses Python for the MVP (accessible, robust libraries) and plans Rust for future performance (memory-safe, fast, cross-compilable).

❌ Complex Dependency Chains

VyOS's custom forks and dependencies often break with upstream updates.
VEEOS Solution: Sticks to vanilla Debian packages, with minimal, well-documented patches.


✅ What We Do Differently



Feature
VyOS
VEEOS



CLI Implementation
Forked vbash
Python-based CLI (MVP), Rust planned


Configuration Parser
OCaml (complex)
Python (parsimonious, MVP), Rust (pest, future)


Configuration Storage
Text file (config.boot)
Embedded graph (networkx, MVP; petgraph, future)


Validation
XML (RelaxNG)
XML-based (simplified or full)


Rendering
Custom scripts, Jinja2
Jinja2 (aligned with VyOS)


Base OS
Custom Debian forks
Vanilla Debian 12


Patching Model
Full forks
Inline .deb patching


ISO Build
Custom scripts
debootstrap, live-build


Update Model
Rolling
Curated LTS



💡 Configuration Management with veeos-parser
The veeos-parser library is designed to simplify VyOS configuration management while introducing modern graph-based capabilities. Here's how it works:
Python MVP (Current)

Parsing: Uses parsimonious to parse config.boot into a graph structure with networkx.
Graph Storage: Stores configurations as nodes (Block, KeyValue) and edges (CONTAINS), persisted to JSON or GraphML.
Manipulation: Supports set, delete, and insert operations, updating the graph directly to avoid re-parsing.
Validation: Implements basic validation (e.g., IP address regex) based on VyOS XML schemas, with optional full validation using lxml and RelaxNG.
Rendering: Converts the graph to text using Jinja2 templates, compatible with VyOS workflows.
Queries: Enables relational queries (e.g., "find interfaces in a subnet") by traversing the graph.
Dependencies: Minimal (parsimonious, networkx, jinja2), installable via pip.

Example Usage:
from veeos_parser import VyosConfig

# Load and parse configuration
config = VyosConfig.from_file("config.boot")

# Modify configuration
config.set(["interfaces", "ethernet", "eth0", "address"], "192.168.1.2/24")
config.insert_key_value(["interfaces", "ethernet", "eth0"], "mtu", "1500")
config.insert_block(["interfaces"], "loopback lo")
config.delete(["interfaces", "ethernet", "eth0", "mtu"])

# Render updated configuration
print(config.render())
# Output:
# interfaces {
#     ethernet eth0 {
#         address 192.168.1.2/24
#     }
#     loopback lo {
#     }
# }

# Query interfaces in a subnet
results = config.query("interfaces_in_subnet", subnet="192.168.1.")
for result in results:
    print(f"Interface: {result['interface']}, IP: {result['ip']}")
# Output: Interface: eth0, IP: 192.168.1.2/24

# Save graph for persistence
config.to_json("config.json")

# Load graph without re-parsing
config = VyosConfig.from_json("config.json")

Rust Future (Post-MVP)

Parsing: Transition to pest for faster, memory-safe parsing.
Graph Storage: Use petgraph for an embedded graph structure, maintaining compatibility with networkx via shared serialization formats (JSON, GraphML).
Python Integration: Expose Rust functionality via PyO3 bindings, ensuring a seamless Python API.
Performance: Leverage Rust's speed for large configurations and complex queries.
Validation: Continue using VyOS XML schemas, with validation implemented in Rust or delegated to Python.

Why Start with Python?

Accessibility: Python is widely used in the VyOS community for scripting and automation (e.g., vyos/configsession.py).
Simplicity: No compilation or external dependencies like Memgraph, just pip install veeos-parser.
Community: Python-only MVP maximizes adoption, with Rust added later for performance-critical use cases.

Why Plan for Rust?

Performance: Rust's speed and memory safety are ideal for parsing large configurations and real-time operations.
Future-Proofing: Aligns with modern systems programming trends, ensuring long-term maintainability.
Compatibility: PyO3 bindings maintain the Python API, making the transition transparent to users.


🛠️ Getting Started
Installation (Python MVP)
pip install veeos-parser

Basic Usage
from veeos_parser import VyosConfig

# Parse a configuration
config = VyosConfig("""
interfaces {
    ethernet eth0 {
        address 192.168.1.1/24
        description "LAN"
    }
}
""")

# Modify configuration
config.set(["interfaces", "ethernet", "eth0", "address"], "192.168.1.2/24")
config.insert_key_value(["interfaces", "ethernet", "eth0"], "mtu", "1500")

# Render and save
print(config.render())
config.save("config.boot")

# Query relationships
results = config.query("duplicate_ips")
if results:
    print("Duplicate IPs:", results)
else:
    print("No duplicate IPs")

Persistence
# Save graph state
config.to_json("config.json")

# Load without re-parsing
config = VyosConfig.from_json("config.json")

Integration with VyOS

Validation: Uses VyOS XML schemas (e.g., interface_definition.rng) for basic or full validation.
Jinja2 Rendering: Aligns with VyOS's template-based rendering:config.render_jinja("template.j2")


Ansible: Automate configuration changes:- name: Update interface address
  ansible.builtin.command: python3 -c "from veeos_parser import VyosConfig; config = VyosConfig.from_file('config.boot'); config.set(['interfaces', 'ethernet', 'eth0', 'address'], '192.168.1.2/24'); config.save('config.boot')"
  delegate_to: vyos_router




📅 Roadmap
MVP (Python-Only)

 Parse config.boot with parsimonious.
 Store configurations as a networkx graph.
 Implement set, delete, insert operations.
 Support basic validation (e.g., IP regex) based on VyOS XML schemas.
 Add full XML validation with lxml and RelaxNG (optional).
 Enhance queries (e.g., VLAN dependencies, firewall rules).
 Publish on PyPI (pip install veeos-parser).
 Create GitHub repository with examples and documentation.

Post-MVP (Rust Integration)

 Implement parser with pest in Rust.
 Use petgraph for embedded graph storage.
 Expose Rust functionality via PyO3 bindings.
 Optimize performance for large configurations.
 Maintain JSON/GraphML compatibility with Python version.

VEEOS OS

 Build ISO image with debootstrap and live-build.
 Create curated APT repository for LTS updates.
 Integrate veeos-parser into the VEEOS CLI and configuration system.
 Release VEEOS LTS based on Debian 12.


🤝 Contributing
We welcome contributions from the VyOS and networking communities! To get started:

Clone the Repository (coming soon):git clone https://github.com/veeos/veeos-parser


Install Dependencies:pip install parsimonious networkx jinja2


Submit Issues or Pull Requests:
Suggest new queries (e.g., topology analysis).
Improve validation based on VyOS XML schemas.
Add integration with tools like Ansible or SaltStack.



Check the CONTRIBUTING.md (TBD) for guidelines.

💬 Community

VyOS Forum: Share feedback and use cases at forum.vyos.io.
Reddit: Join discussions on r/vyos or r/networking.
GitHub Issues: Report bugs or suggest features at github.com/veeos/veeos-parser (TBD).
Blog Posts: Follow our blog for tutorials and updates (TBD).


📜 License
VEEOS and veeos-parser are licensed under the MIT License, ensuring open collaboration and adoption.

💡 Vision
A modern, clean, and minimal network OS that feels like Debian, with a powerful configuration management library that simplifies parsing, manipulation, and analysis. VEEOS aims to be simple to use, easy to update, and safe to trust, empowering network administrators and DevOps professionals with a sustainable, community-driven platform.


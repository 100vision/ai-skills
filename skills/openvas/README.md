# openvas-cli Skill 

a skill for managing [Greenbone Community Edition (OpenVAS)](https://github.com/greenbone/openvas) vulnerability scanners remotely via CLI.

## Features

- Install and configure `openvas-cli` on any Linux workstation
- SSH-based remote access to OpenVAS instances (default transport)
- Full scan lifecycle: target creation, task management, scan execution, report retrieval
- Credential management (username/password, SSH key, SNMP)
- Built-in troubleshooting guide for common connectivity issues

## Installation

- for human, install by running:
```bash
npx skills add 100vision/openvas-cli-skill
```




## Skill Usage Examples

### CLI Install & Setup

> 帮我装一下 openvas-cli，我要连远程的 Greenbone 扫描器

> Install openvas-cli and connect to my OpenVAS instance at 192.168.1.100

### Run a Scan

> 帮我创建一个扫描任务，目标网段是 192.168.1.0/24

> Create a vulnerability scan for hosts 10.0.0.1-50 using the Full and Fast config

### Get Reports

> 把上次扫描的报告导成 PDF

> List all scan reports and export the latest one

### Troubleshoot

> openvas-cli doctor 报错了，帮我排查一下

> SSH connects but gvm-cli socket command fails, help me debug

## Skill Structure

```
openvas-cli/
├── SKILL.md                     # Main instructions
└── references/
    ├── commands.md              # Complete command reference
    └── troubleshooting.md       # Troubleshooting guide & Q&A
```

## Prerequisites

- A running OpenVAS / Greenbone Community Edition instance
- SSH access to the OpenVAS host (user must be in `_gvm` group)
- Debian/Ubuntu workstation with Python 3.10+
- `gvm-tools` installed locally (`pipx install gvm-tools`)

## Safety

This skill contains **no hardcoded credentials, IPs, or secrets**. All examples use placeholder values. The skill only provides instructions — it does not execute any commands on its own.

## License

MIT

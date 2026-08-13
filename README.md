# Linux Homelab

Hands-on Linux system administration homelab built on Hyper-V to develop
practical skills in Linux administration, networking, troubleshooting,
automation, and infrastructure management.

## Lab Environment

- Windows 11 host
- Hyper-V
- Ubuntu Server 24.04 LTS
- Dedicated `10.10.10.0/24` lab network
- Windows NAT
- SSH / VS Code Remote - SSH

## Architecture

TBA

## Labs

| Lab | Topic | Status |
|---|---|---|
| 01 | Linux Filesystem | Completed |
| 02 | Users and Groups | Completed |
| 03 | Permissions and Ownership | Next |
| 04 | Process Management | Planned |
| 05 | systemd and Services | Planned |
| 06 | Networking | Planned |
| 07 | Storage and LVM | Planned |
| 08 | Logging and Monitoring | Planned |
| 09 | Shell Scripting | Planned |
| 10 | Troubleshooting | Planned |

## Documentation

- Hyper-V Setup
- Remote Access and Networking

## AI Acknowledgement

AI tools are used throughout this project as a learning and guidance resource.

The purpose of using AI is not to replace hands-on practice, but to support the learning process by providing explanations, suggesting lab scenarios, reviewing configurations, and helping troubleshoot issues encountered while building the environment.

All administrative tasks documented in this repository are performed by me within my own homelab environment. This includes configuring virtual machines, executing commands, modifying configuration files, testing services, diagnosing failures, and verifying the results.

When problems occur, they are treated as part of the learning process rather than simply bypassed. For example, while configuring remote access, I encountered changing DHCP addresses, SSH configuration issues, Windows file permission problems, and connectivity troubleshooting. These issues were investigated and resolved within the lab environment.

### Why I Use AI

My goal is to use AI as an interactive learning tool rather than as a substitute for understanding.

Instead of only reading documentation or following static tutorials, AI allows me to:

- Ask questions when I do not understand a concept
- Build labs around the topics I am studying
- Receive guidance while troubleshooting
- Compare my understanding against explanations
- Explore why a configuration works or fails
- Turn theoretical knowledge into practical exercises

Where appropriate, official documentation and command-line manuals are also used to validate concepts and configurations.

### Intended Outcome

The intended outcome of this project is to develop the ability to administer and troubleshoot Linux systems independently.

As the homelab develops, I expect to become progressively less dependent on step-by-step guidance and more capable of:

- Planning configurations independently
- Selecting the appropriate commands and tools
- Diagnosing problems systematically
- Understanding the impact of configuration changes
- Recovering from mistakes
- Automating repetitive administration tasks
- Working across Linux and Windows infrastructure

The repository therefore represents not only the completed configurations, but also the progression of my practical system administration skills.
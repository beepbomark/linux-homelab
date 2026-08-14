# Linux Homelab

Hands-on Linux system administration homelab for developing practical
skills in Linux administration, networking, storage, security,
automation, and infrastructure management.

## Lab Environments

### Ubuntu

General-purpose Linux administration environment.

- Ubuntu Server 24.04 LTS
- Microsoft Hyper-V
- SSH / VS Code Remote
- Filesystem administration
- Users and groups
- Networking
- Shell scripting

[View Ubuntu Labs](ubuntu/)

### Red Hat Enterprise Linux

Two-server RHEL environment for enterprise Linux administration
practice.

| Host | IP |
|---|---|
| rhel-server01.example.com | 172.16.0.100 |
| rhel-tester01.example.com | 172.16.0.50 |

Topics include:

- RHEL system administration
- DNF and RPM
- systemd
- Storage and LVM
- SELinux
- Networking
- SSH
- Troubleshooting

[View RHEL Labs](rhel/)

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
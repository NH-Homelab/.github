# NH-Homelab
A collection of repositories for various self-hosted services. The purpose of the homelab is to have have an accessible and flexible development environment, host my projects at a lower cost than a hosting provider, and have fine-grained access control to my projects for other users. 

This project is ongoing, and much of the code I've written for it hasn't been published yet. I plan to organize the code and upload it to repositories in this organization. 

## Physical Server
I repurposed a desktop PC for this homelab. After migrating to proxmox, I decided to upgrade the memory with an additional 16 GB for the additional containers. The GPUs were added for future machine learning projects. 

### Hardware
| Device | Description |
| --- | --- |
| Processor |  Intel(R) Core(TM) i7-6700K CPU @ 4.00GHz |
| RAM | 4x 8GB |
| GPU1 | Geforce GTX 1050 TI 4GB |
| GPU2 | Geforce GTX 1060 6GB |
| HDD | 2x 3TB |
| SSD | 2x 500GB |

### Operating System
Initially, the server was running an Ubuntu server distribution. Eventually, I wanted to be able to spin up many containers on the fly, and interact with them using a web interface. Proxmox is now the OS running on the physical machine.


## Proxmox
Configuring proxmox has been a new challenge. Virtual networking was a huge struggle at first, since I was used to using UFW to manage open ports. While creating firewall rules was simple in the webUI, now I had to create rules to forward packets to the correct containers as well. Here is the work in progress plan for the network structure of the containers. I wanted to avoid a flat network, but I also don't want to overcomplicate it. I decided to separate containers that needed access to the NAS and those that didn't. The services on vmbr0 all have statically assigned IP's, while most of the other containers / machines on vmbr1 are dynamically assigned by the Pihole DHCP. 

![Network Diagram of the Services running on Proxmox](/profile/assets/NetworkDiagram.jpeg)


### Services

#### Reverse Proxy and User Authentication
A big motivation for this project was the ability to create as many web applications as I wanted on my server, while centrally authenticating users. Whenever I wish to start a new web project, I can run it on any arbitrary port and allow the Nginx reverse proxy to handle routing the traffic. This means that every service I create needs to have a configuration file for the nginx server. When a user tries to access a web application, their request is first sent to a nodejs backend for authentication. The backend is configured to use Google OAuth 2.0. The first time a user signs in to google on my site, their information is stored in the user database. From there, I have to manually assign them to user groups that dictate their access to services. This is how I protect all web applications running on this server.

| Group | Description |
| --- | --- |
| Admin | Only my accounts are in this group. Unrestricted access to the server's applications. |
| User | Approved users are in this group, like friends and family. They have access to a large number of applications, but no administrative rights. |
| Unapproved | Users that have logged in, but haven't been manually assigned a group. These users have limited access to the home page. |
| Banned | Users that have no access to the server applications. |

#### Jupyterlabs
I frequently use jupyterlabs for school assignments and side projects. I use different personal computers whenever I'm at home or out of the house, which made continuing work on projects challenging. I could use something like github to centrally host my code and keep that remote up to date, but I wanted my git history to be organized to actual progress and releases. my solution is a centrally hosted jupyter server which I can access from the web on any of my machines. It has a dedicated container that is ready to go for different python projects and has the most recent copies of my notebooks. Another advantage of this method is that collaboration in group projects is easier. Jupyterlabs has a collaborative flag which allows multiple users simultaneous access to the same notebook. When working on group projects, I can give people permission to this jupyter server where we can work together in real time. There's no worrying about IP's and port forwarding since it's already setup and ready to go on this centralized server. Of course, the web application is protected by the NGinx reverse proxy so that only authorized users have access.

#### OpenVPN + Pihole
In highschool, I wanted to circumvent content filtering on the school network. I was frusterated with free VPN's which were slow and made me watch ads. Eventually, the school admins even blocked port 1194 to crack down on VPN's. I decided to setup my own OpenVPN server at home for a free performant VPN. I learned to hide my traffic from the school admins by hosting this on port 443/80.

These days, my motivation for running my own VPN is privacy on public networks, and robust mobile ad blocking. With Pihole set as the primary DNS for my VPN, I can get ad blocking even when using an in-app browser that doesn't support plugins. 

#### Rustdesk 
Rustdesk allows me to remotely interact with various machines and see their desktops. When I'm using a windows VM for malware reverse engineering or want to connect to one of my personal computers, I have them connect to my rustdesk relay server. This allows me to securely interact with my machines over the internet with more performance than if I was using Rustdesk's public relay server. 

#### Samba NAS
A few of my containers needed shared access to data on my harddrives. However, I ran into a problem where I couldn't mount the same physical drive to multiple containers. I decided to create a samba NAS container for this shared data. Now, I can store my media and machine learning datasets in this centralized location, and give only the necessary containers access to connect over the vm bridge. I eventually want to create a file or picture sharing service for friends and family which will need to use this NAS as well.

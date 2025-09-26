## Enumeration Principals 
Often a lot of pentesters misunderstand this step and go right into looking for authentication services like ssh,rdp,winrm etc... our goal is not to get into the target system, but to find out all the possible ways to do that. There are some questions we should ask ourselfs to aid in the thinking process:
- What can we see? - what are the services available
- What reasons can we have for seeing it? - isn't there a protection system
- What image does what we see create for us? - what's the system infrastructure 
- What do we gain from it? - insight into the inner workings
- How can we use it? - is there any vulnerable version
- What can we not see? -  the services available internally (access via port forwarding)
- What reasons can there be that we do not see? - admin functions only (low security)
- What image results for us from what we do not see?
## Enumeration Methodology
There are three main areas of enumeration:
- Infrastructure-based enumeration
- Host-based enumeration
- OS-based enumeration
![[Pasted image 20250629172056.png]]
As can be seen in the picture above, we have six layers. We can summarize these layers in the following:
### Layer No.1: Internet Presence
As the name suggests, we want to know all the company assets that is available on the internet. **The goal of this layer is to identify all possible target systems and interfaces that can be tested.**
### Layer No.2: Gateway
In this layer, we want to know what are the defensive systems that will prevent us from interacting with the target, were is it located. **The goal is to understand what we are dealing with and what we have to watch out for.**
### Layer No.3: Accessible Services
Here we examine each service the server has, why is it on the server?, and is it connected with other services?. **This layer aims to understand the reason and functionality of the target system and gain the necessary knowledge to communicate with it and exploit it for our purposes effectively.**
### Layer No.4: Processes
After gaining access to the system, we want to know what is happening. We can check the process running on the server and see how the interact with each other. **The goal here is to understand these factors and identify the dependencies between them.**
### Layer No.5: Privileges
Every service is run by a user that have specific privileges. Our understanding of these privileges is a key factor to exploiting the system. **It is crucial to identify these and understand what is and is not possible with these privileges.**
### Layer No.6: OS Setup
After gaining access to the system, we can view the config files and view the internal security mechanisms. **The goal here is to see how the administrators manage the systems and what sensitive internal information we can glean from them.**

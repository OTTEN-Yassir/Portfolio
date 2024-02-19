---
title: Voice over IP (VoIP) Network Design and Implementation Project
publishDate: 2024-02-16 23:29:30
img: /assets/voip.png
img_alt: OTTEN Yassir
description: |
 The Voice over IP (VoIP) technology project: We design and implement a sophisticated call center system, integrating Elastix and Ekiga software for seamless communication. There is how i made this architecture step by step.
tags:
  - Network
  - Voice over IP
  - Elastix
  - Ekiga
  - Step by Step
---

# Table of Contents

1. [Introduction](#introduction)
2. [General Configuration](#general-configuration)
    - [Elastix Account](#elastix-account)
    - [Ekiga Account](#ekiga-account)
    - [Call](#call)
3. [Call Flow Diagram](#call-flow-diagram)
4. [Day/Night](#day/night)
    - [Follow Me](#follow-me)
    - [Day/Night Control](#day/night-control)
5. [IVR](#ivr)
    - [Upload Message](#upload-message)
    - [IVR Configuration](#ivr-configuration)
    - [Voicemail](#voicemail)
    - [Ring Group](#ring-group)
    - [Queue](#queue)
6. [Server Interconnection](#server-interrconection)
    - [IAX2 Extension](#iax2-extension)
    - [Trunk](#trunk)
    - [Route](#route)


## Introduction 


 We have implemented a call center system involving the configuration of 4 servers, each dedicated to a specific group of 2 or 3 persons. Each server operates with Elastix software for call management. Within each group, physical machines have been configured with Ekiga software, establishing a connection with the corresponding server. This client-server architecture enables efficient management of all communications within the call center.  

**Ekiga**:  
Ekiga is an open-source, real-time communication software application. It offers Internet telephony, instant messaging, and video conferencing functionalities. Compatible with SIP and H.323 protocols, Ekiga allows users to initiate voice and video calls over IP networks.  

**Elastix**:  
Elastix is a unified communication platform based on the Asterisk IP telephony management system. This open-source solution provides a user-friendly interface to configure and manage VoIP services, including call center features. Elastix servers play a central role in managing calls, users, and services within an integrated communication environment.  

## General Configuration  


The Elastix servers, once booted and assigned an IP address, are configured through a graphical interface accessible via a web browser at the server's IP address: 192.168.212.3. This interface will be used for further configuration.  

The clients are Ekiga software installed on Windows 7 PCs in the same network as the servers. These software applications require a reference server and accounts recognized by this server.  

#### Elastix Account  
On the Elastix server, under the **PBX** and **PBX Configuration** tabs, select **Extensions** on the left and **Add Extension** on the right. This page prompts the creation of a type of extension: a **Device**, **Generic SIP Device**.  
![elastix account](/assets/AllExtensions.PNG)  
Click on **Submit** to enter the configuration page; most options remain at their default values.  
![elastix account](/assets/DjaouadExtension_1.PNG)  
This account will be used by Djaouad. The SIP Alias is his identification number, which is also used as his phone number and email address.  
![elastix account](/assets/DjaouadExtension_2.PNG)  
Once these elements have been entered, click on **Submit** at the bottom of the page, then on the pop-up **Save Configuration**.  

#### Ekiga Account  
To add an account in the Ekiga application, once launched, select **Edit** then **Accounts**, which provides access to the following interface.  
![ekiga account](/assets/listAccount.png)  

The **Enable** and **Disable** options allow you to connect and disconnect the selected account, respectively. The **Edit** and **Delete** options open a form.  
![ekiga account](/assets/fieldAccount.png)   
**Name** is local, it describes the user.  
**Registrar** is the SIP server's IP address, 192.168.212.3.  
**User**, **ID**, and **Password** are the account characteristics defined on the server.  
We did not use the Timeout; it remained with its default value. 

To create an account, select **Accounts** then **Add a SIP account**, and the same form will open.  

#### Call  
Once a second account, under the name of Yassir, was created, we were able to make calls.  
On Ekiga's home screen, a keypad allows entering the recipient's number, i.e., Yassir's ID: 58102. This number is followed by the registrar's address.  
![call account](/assets/Clavier.PNG)  
  
![call account](/assets/AppelEnCours.PNG)   
  Yassir then receives a call notification through a ringtone and a pop-up from the application, which he can either accept or decline.  

## Call Flow Diagram  


Here is a representation of the exchanges when a call is made to a number recognized by our server.  
![call diagram](/assets/callDiagram.png)  

When a call is rejected, the **Day/Night** service activates, directing the call either directly to a voicemail or to an answering machine.  
The answering machine offers several services to contact; if the caller presses 3 or waits too long, the answering machine repeats its announcement.  
The configuration of the **Day/Night** service and the answering machine is detailed in the following sections.  

## Day/Night  

Le service **Day/Night** permet d’orienter différemment les appels selon la personne contactée et l’heure de l’appel.  
#### Follow Me  

The **Follow Me** feature associates one or more users with a destination. If the user does not answer an incoming call, the call is automatically redirected to the chosen destination.  
Here, the phone number 57444 (announced in the **Follow-Me** List) is associated with Pierre's **Day/Night** Mode.  

![Follow ME](/assets/FollowMe_Pierre.PNG)   

Other Follow Me settings have been created in a similar manner for various other accounts.  

#### Day/Night Control  

A **Day/Night** object is identified by a code and a description. The **Day** and **Night** options are used to define the behavior of the **Day/Night** based on the time of the call.  
![Day/Night](/assets/AllDayNight.PNG)  
In this example, if someone tries to contact Pierre during the day, the call is redirected to the IVR **VoicemailCenter**. During the night, the caller is sent directly to Pierre's voicemail.  
![Day/Night](/assets/DayNight_Pierre.PNG)  

## IVR  
IVR, or **Interactive Voice Response**, is an automated system of interactive voice responses used in telecommunications. It enables users to interact with a computer through keypad commands. The IVR guides callers with pre-recorded voice messages.  
 #### Upload Message  

 The first step in creating an IVR is to record a voice message to guide the caller when the recipient is unavailable.  
 To do this, go to **PBX Configuration > System Recordings** and click on **Add Recording**.  
 The audio file format must be wav mono 16 bits.  

![IVR](/assets/System_Recordings.PNG)  

My uploaded file is **VoicemailCenter**.  
#### IVR Configuration  

Next, go to **PBX Configuration > IVR**.  
Then, click on **Add IVR** to start its configuration.
**Change Name**: VoicemailCenter (the name of the IVR)  
**Announcemen**t: VoicemailCenter (the previously created audio file)  
At the bottom of the page, there are options associated with the keypad numbers.  
Redirects to other callers must be to users/extensions previously created.  
**key 1** = redirect to info account <301> (Ring Group)  
**key 2** = redirect to technical service <302> (Ring Group)  
**key 3**= listen to the IVR again  
These are the different options offered to the caller when the recipient does not answer.  
After redirecting to the info or technical service account, if they are not reachable, then a standard voicemail starts with the option to leave a message after the beep.  

![IVR](/assets/IVR.PNG)   

#### Voicemail  

The traditional Voicemail system, allowing users to leave a voice message after a beep, was implemented at certain times. This system was also linked to an email inbox associated with our SIP number (via the mailbox field in the extension).  

![Voicemail](/assets/AllWebMail.PNG)  

After a message has been left on the voicemail, an email containing the audio file is sent to the associated recipient.  
![Voicemail](/assets/58102Mail.PNG)  
  
![Voicemail](/assets/ContentMail_2.PNG)  

Subsequently, the operation was repeated by synchronizing our user extensions not with the mailbox on the server, but with our respective HE2B mailboxes.  
![Voicemail](/assets/mailAsterisk.png)  

#### Ring Group  

A Ring Group is a feature that allows multiple telephone extensions to be grouped together. When a call is received on the group number, all phones belonging to the group will ring simultaneously, enabling any of them to answer the call.  
This provides an efficient way to manage incoming calls and ensure they are promptly attended to by a member of the group.  
Here are screenshots of the two groups we created:  

![Ring Group](/assets/RingGroups_301.PNG)   
![Ring Group](/assets/RingGroups_301.PNG)   
  
For configuration, we had to fill in several fields, including "ringing strategy." The ringing strategy determines how group members will be alerted. Common strategies include:  

- **Ring All**: All group members ring simultaneously.  
- **Round Robin**: Calls are distributed sequentially to group members.  
- **Least Recent**: The call is directed to the group member who has not received a call for the longest time.  
The Ring Time is the duration for which member phones ring before the call is transferred elsewhere.

#### Queue  

Queue is a feature that efficiently manages incoming calls by placing them in a queue until an available member of the group can answer them. This is particularly useful in call centers and businesses where multiple employees can handle calls. The most common queue strategies include:  


- **Ringall**: All members of the queue ring simultaneously.  
- **Leastrecent**: The call is directed to the member who has not received a call for the longest time.  
- **Random**: The call is directed to a randomly chosen member.  
- **Roundrobin**: Calls are distributed in sequences to queue members.  

## Server Interconnection  
So far, we have established local communication, internal to our server and our network/company. Going forward, we would like to be able to communicate with the outside world. To do this, we need to interconnect the different servers with each other.  

#### IAX2 Extension 

To be able to configure trunks between the different servers, each of them must first create an IAX2 extension, which will allow the creation of the trunks.  

![IAX2](/assets/IAX2.png)   

The extensions must be unique for each link between the different servers.  

#### Trunk  


The second step in the server interconnection process is to create Trunks. This will allow the servers to properly route packets between each other.  
To do this, go to **PBX Configuration > Basic > Trunk**. Then, create the trunk by clicking on **Add IAX2 Trunk**.  

![Trunk](/assets/trunk.png)   


Here are my two Trunk links for the two servers I want to communicate with. To continue, these Trunks need to be configured with the correct information.  


![Trunk](/assets/TrunkVersJulien_1.PNG)  

**Trunk Name**: the name of the trunk link, here, TrunkToJulien.  

**+prefix**: a number that the user will have to enter with each number to indicate that the communication should be redirected to the Trunk link.  
**| match pattern**: the format of the numbers of the server to be reached. Here, all the numbers of Julien's server users consist of 3 digits so XXX.  
**In the Outgoing Settings section**:  
**host =**: the IP address of the server to contact (always Julien).  
**username =** : the extension number of the IAX2 (step a) created on the server to be reached.  

![Trunk](/assets/TrunkVersJulien_2.PNG)  

**In the Incoming Settings section**:   
**USER Context**: the extension number of the remote server's IAX2 extension.  
**user**: the extension number of the remote server's IAX2 extension.  
**Register String**: the extension number of the remote server's IAX2 extension and the server's IP address separated by an @.    
**secret**: same password everywhere.  
The same operation must be repeated according to the number of servers we want to interconnect.  However, it is essential to pay attention and adjust the above fields, which are specific to each server.  

#### Route  


The last step in the interconnection process and establishing inter-server communication is adding a route per server.  
This route will utilize the previously created trunk.  
To add a route, go to **Basic > Outbound Routes**.  

![Route](/assets/RouteVersJulien.PNG)  


**Route Name**: the name of the route, here, **routeToJulien**.  
In the central part, you need to enter the desired prefix and format (match pattern) of the server to be reached.  
In the bottom part (Trunk Sequence...), you need to expand the menu that will display all the created trunks. Select the corresponding trunk to establish the connection between them.  
Once this step is completed, users from both servers can communicate.  
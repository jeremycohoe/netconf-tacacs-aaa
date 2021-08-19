# netconf-tacacs-aaa
netconf-tacacs-aaa example

An example to allow local login on the console line, and NETCONF to authenticate and get authorization from TACACS via Cisco ISE

Thanks to Ioannis @mythryll and Joe Clarke @ Cisco, and the entire DEVNET 500 group for the example :)

For IOS-XE devices that need to be configured with netconf, the IOS-XE 16.x and later documentation includes a dedicated section on Programmability that details how to setup the IOS-XE device for netconf, for example:
https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg/configuring_yang_datamodel.html
The necessary commands are summarized in the respective section https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/173/b_173_programmability_cg/configuring_yang_datamodel.html#privilege-to-use-netconf , however no prevision is made for setups that are common in production networks, where device access is often based on AAA mechanisms like a group of TACACS+/ISE servers. This has been the same in the IOS-XE documentation ever since netconf support was included, going as back as v.16.6.
A similar section is also available in the documentation for netconf and programmability support in developer.cisco.com : https://developer.cisco.com/docs/ios-xe/#!enabling-netconf-on-ios-xe
However in this case there is a section dedicated to integration with AAA and TACACS+/ISE:
https://developer.cisco.com/docs/ios-xe/#!ios-xe-aaa-integration-with-netconf-and-restconf 
The example in the page does not include password protected or authorized access to the console. If only login authentication is needed then the change from none to local-if-authenticated should do the trick but for authorizing commands, it gets a little more complicated.
The purpose of this text is to define 2 separate example scenarios and display the necessary IOS-XE config snippets to deploy in order to satisfy the respective requirements, for anyone that needs such a reference.
The base for this text has been a discussion that evolved in the Devnet 500 webex space between Ioannis Theodoridis @mythryll from the Bank of Greece and Joe Clarke from Cisco, with the contribution of Stuart Clark and Jeremy Cohoe. The issue the led to the discussion was brought forward by Katerina Dardoufa also from Bank of Greece. Thanks again, Joe, Jeremy and Stuart.

Base Scenario:
In a production network, IOS-XE devices are already setup with TACACS+ (ISE) access for network administrators. A non default profile (OTHERGROUP) is defined for the AAA admin access.
It is decided that Netconf should be deployed in order to allow for MDT assisted performance monitoring.
As implied by the documentation in developer.cisco.com, it is necessary to define the default AAA profile for netconf authentication to work with ISE based user access.
It was discovered that unless specific actions are taken, console access is broken, meaning that login was allowed but no commands were allowed to be executed.
A separate profile (CON) is used to define console access in order to avoid the application of the default profile.
Do not forget that for local authentication to work, one user at least must have been already configured first, like in the documentation example:
username example-name privilege 15 password example_password
Also don't forget that providing user passwords in config in unencrypted form with be deprecated in the near future (type 7 passwords will soon be deprecated message comes up in the console when you reboot the machine)
The snippet below is a production based one, using accounting as well for ISE user profiles. Those commands are not necessary if you are not doing accounting.

Starting Config:
```
aaa authentication login default group TACACS-ISE local
aaa authentication login OTHERGROUP group TACACS-ISE local
aaa authentication login CON local
aaa authorization console
aaa authorization config-commands
aaa authorization exec default group TACACS-ISE if-authenticated 
aaa authorization exec OTHERGROUP group TACACS-ISE if-authenticated 
aaa authorization exec CON none 
aaa authorization commands 1 default group TACACS-ISE local if-authenticated 
aaa authorization commands 1 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 5 default group TACACS-ISE local if-authenticated 
aaa authorization commands 5 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 15 default group TACACS-ISE local if-authenticated 
aaa authorization commands 15 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa accounting exec default start-stop group TACACS-ISE
aaa accounting exec OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 5 default start-stop group TACACS-ISE
aaa accounting commands 5 OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 15 default start-stop group TACACS-ISE
aaa accounting commands 15 OTHERGROUP start-stop group TACACS-ISE

line con 0
 exec-timeout 60 0
 authorization exec CON
 logging synchronous
 login authentication CON
 stopbits 1
```

This of course did not allow for commands to be executed in the console, even if the aaa authorization exec CON none was changed from none to local-if-authenticated.
The use of command authorization in the default profile affects the console as well because of the aaa authorization console command. There are two ways to fix this situation:
1) Eliminate all authorization from the console by removing the aaa authorization console command and adjusting line console 0 config as well.
2) Provide command authorization configuration also for the CON profile both in the main section as well as the line console 0 section (thank you Joe!)
Here are the two config snippets:

Solution no 1:
No console authorization, privilege level 15 is added to enter directly in privileged exec mode after console authentication.

```
aaa authentication login default group TACACS-ISE local
aaa authentication login OTHERGROUP group TACACS-ISE local
aaa authentication login CON local
aaa authorization config-commands
aaa authorization exec default group TACACS-ISE if-authenticated 
aaa authorization exec OTHERGROUP group TACACS-ISE if-authenticated 
aaa authorization commands 1 default group TACACS-ISE local if-authenticated 
aaa authorization commands 1 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 5 default group TACACS-ISE local if-authenticated 
aaa authorization commands 5 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 15 default group TACACS-ISE local if-authenticated 
aaa authorization commands 15 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa accounting exec default start-stop group TACACS-ISE
aaa accounting exec OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 5 default start-stop group TACACS-ISE
aaa accounting commands 5 OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 15 default start-stop group TACACS-ISE
aaa accounting commands 15 OTHERGROUP start-stop group TACACS-ISE

line con 0
 exec-timeout 60 0
 privilege level 15
 logging synchronous
 login authentication CON
 stopbits 1
```

Solution no 2:
Keep console command authorization by adding all necessary commands both in the main and in the line console 0 sections:

```
aaa authentication login default group TACACS-ISE local
aaa authentication login OTHERGROUP group TACACS-ISE local
aaa authentication login CON local
aaa authorization console
aaa authorization config-commands
aaa authorization exec default group TACACS-ISE if-authenticated 
aaa authorization exec OTHERGROUP group TACACS-ISE if-authenticated 
aaa authorization exec CON local if-authenticated 
aaa authorization commands 1 default group TACACS-ISE local if-authenticated 
aaa authorization commands 1 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 5 default group TACACS-ISE local if-authenticated 
aaa authorization commands 5 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 15 default group TACACS-ISE local if-authenticated 
aaa authorization commands 15 OTHERGROUP group TACACS-ISE local if-authenticated 
aaa authorization commands 15 CON local if-authenticated 
aaa accounting exec default start-stop group TACACS-ISE
aaa accounting exec OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 5 default start-stop group TACACS-ISE
aaa accounting commands 5 OTHERGROUP start-stop group TACACS-ISE
aaa accounting commands 15 default start-stop group TACACS-ISE
aaa accounting commands 15 OTHERGROUP start-stop group TACACS-ISE

line con 0
 exec-timeout 60 0
 privilege level 15
 authorization commands 15 CON
 authorization exec CON
 logging synchronous
 login authentication CON
 stopbits 1
```

This should be enough to make it work. It has been tested with 17.3.1a and 17.3.3 on ISR-44k routers. It should work on other devices as well.

# netconf-tacacs-aaa
netconf-tacacs-aaa example

An example to allow local login on the console line, and NETCONF to authenticate and get authorization from TACACS via Cisco ISE

Thanks to Ioannis @mythryll and Joe Clarke @ Cisco, and the entire DEVNET 500 group for the example :)


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

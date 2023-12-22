# TACACS Case Config
## Configure Local AAA

1. Create user
```
Rtr-1(config)# username admin privilege 15 algorithm-type scrypt secret Cisco123
```
2. Enable AAA on Rtr-1 with the aaa new-model command.
```
Rtr-1(config)# aaa new-model
```
3. Configure the Rtr-1 router to use the local database as the default authentication method for administrative logins.
```
Rtr-1(config)# aaa authentication login default local
```
4. On the Rtr-1 router, configure the use of the local database as the default authorization method to determine if a user has permission to access the device command line.
```
Rtr-1(config)# aaa authorization exec default local
```
By default, as a safeguard, AAA authorization does not affect line CON 0.
5. test the user
```
login as: admin
Using keyboard-interactive authentication.
Password: Cisco123
Rtr-1#
```
Note the command prompt. The username admin is not only authorized to access the CLI, they are authorized for privilege level 15. This privilege level was authorized upon login.
6. verify the user
```
Rtr-1# show privilege
Current privilege level is 15
Rtr-1# show users
    Line       User       Host(s)              Idle       Location
   0 con 0                idle                 00:03:38   
*  1 vty 0     admin      idle                 00:00:00 10.10.10.15

  Interface    User               Mode         Idle     Peer Address
```

## Configure Server-Based AAA

Local authentication and authorization are not very scalable, because the configuration must be repeated on all network devices. Any changes to AAA policy must then be updated individually on all network devices. Using a centralized AAA server is much more scalable. Policy is defined centrally on the server. Updates that are made to the centralized policy are available to all network devices. Also, using centralized AAA servers allows the third "A," accounting. There is not enough persistent storage on network devices to store accounting records locally. Using a centralized server can provide plenty of storage.

TACACS+ has been configured on the Ubuntu server. A username, admin, has been defined with full privileges along with a password (Cisco123).

The first step in using centralized AAA is to define the AAA server with which the router will communicate.

1. use the tacacs server name command to access tacacs server subconfiguration mode. At a minimum, you must define the address and shared secret key (Cisco123!) for the server.
```
Rtr-1(config)# tacacs server Ubuntu-Server
Rtr-1(config-server-tacacs)# address ipv4 10.10.10.41
Rtr-1(config-server-tacacs)# key Cisco123!
```
note : https://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/46420-pre-sh-keys-ios-rtr-cfg.html

2.  verify that the server is correctly defined and operational
```
Rtr-1# test aaa group tacacs+ admin Cisco123 new-code 
Sending password
User successfully authenticated

USER ATTRIBUTES

username             0   "admin"
reply-message        0   "Password: "
```
3. define AAA login authentication and EXEC authorization to use group tacacs+ as the primary method and the local database as the secondary method.
```
Rtr-1(config)# aaa authentication login default group tacacs+ local
Rtr-1(config)# aaa authorization exec default group tacacs+ local
```
4.  configure the use of per-command authorization using the TACACS+ servers, for privilege level 15 commands and for configuration mode commands.
```
Rtr-1(config)# aaa authorization commands 15 default group tacacs+ local      
Rtr-1(config)# aaa authorization config-commands
```
5. enable accounting for EXEC process access and privilege level 15 commands.
```
Rtr-1(config)# aaa accounting exec default start-stop group tacacs+
Rtr-1(config)# aaa accounting commands 15 default stop-only group tacacs+
```
6. Verify that the accounting records are stored in the TACACS+ server logs.

On the Ubuntu-server, enter the sudo tail /var/log/tac_plus.acct command to display the last few accounting entries sent to the TACACS+ server. The AAA server username is aaa and password is cisco.

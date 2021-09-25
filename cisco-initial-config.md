<h2>Cisco initial configuration </h2>


<h3>1. User/Password</h3>

```
username test privilege 15 password 0 Test1234

enable secret Test1234
```

<h3>2. Enable telnet</h3>

```
line vty 0 4
 login local
 transport input telnet
!
````

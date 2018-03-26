# Tuneling Tutorial 
For accessing your Jupyter notebooks running on a remote server such as client network
---

This is courtesy of [AugustCouncil](http://www.augustcouncil.com/~tgibson/tutorial/tunneling_tutorial.html)

### An illustrated guide, tutorial, how-to, on ssh tunneling

There are two situations which typify the need for ssh tunneling to a computer which is accessible on the Internet. Let's call this remote server "```remote.server.com```":

**Problem**: A server/service on remote.server.com is listening to port 11000. 
You want to use your own personal, local computer to connect to it. However, port 11000 is hidden behind remote.server.com's firewall.

**Tunneling solution**: You can connect to port 22000 of your own personal, local computer and the tunnel will push it to port 11000 on remote.server.com

**Problem**: A server/service on your own personal, local computer is listening on port 11000. You want remote.server.com to connect to your server/service. However, your personal, local computer does not have a publicly accessible address on the internet.

**Tunneling solution**: remote.server.com can connect to itself on port 22000 and the tunnel will push it to port 11000 on your personal, local computer.
Here's what the command looks like for #1 when typed on your personal, local computer:
```ssh -N -L 22000:localhost:11000 remote.server.com```

```-N``` After you connect just hang there (you won't get a shell prompt)

```-L 22000``` The connection will originate on port ```22000``` of your local machine 

```localhost:11000 remote.server.com``` will make sure that the other end of the tunnel is ```localhost, port 11000```

![](http://www.augustcouncil.com/~tgibson/tutorial/images/sshtunnel1.jpg)

Here's what the command looks like for #2 when typed on your personal, local computer:

```ssh -N -R 22000:localhost:11000 remote.server.com```

```-N``` After you connect just hang there (you won't get a shell prompt)

```-R 22000``` The connection will originate on port ```22000``` of the Remote computer (in this case, remote.server.com)
localhost:11000 your personal, local computer will make sure that the other end of the tunnel is localhost, port 11000

 ![](http://www.augustcouncil.com/~tgibson/tutorial/images/sshtunnel2.jpg)

# BONUS USAGE!
You've read this far, so one more useful usage we'll pretend to use
```ssh -N -L 22000:192.168.1.2:11000 remote.server.com```

-N After you connect just hang there (you won't get a shell prompt)
-L 22000 The connection will originate on port 22000 of your personal, Local machine
```192.168.1.2:11000``` remote.server.com will make sure that the other end of the tunnel is ```192.168.1.2, port 11000```

![](http://www.augustcouncil.com/~tgibson/tutorial/images/sshtunnel3.jpg)


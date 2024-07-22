
# Nmap scan techniques 

-sS/sT/sA/sW/sM


Lets start 

![[Pasted image 20240612214331.png]]
`nmap -sn 10.4.27.83`

we found out that the host is up .


![[Pasted image 20240612214404.png]]

`nmap -Pn -sS -F 10.4.27.83`


based on the ports we can tell this is a windows server

when we take a closer look it says

![[Pasted image 20240612214442.png]]

(closed) not filtered 

if its filtered thats likely a firewall there

now thats one way of telling if there is a firewall or filtering between you and the host .

taking the fact that this is a windows system we are not sure if the windows firewall is off 

to be sure we try this

`nmap -Pn -sA -p443,3389 10.4.27.83`

![[Pasted image 20240612214638.png]]

if there is a firewall it would say that the state is filtered .

This is how to detect if there is a firewall or filtering .

----------------------------------------


![[Pasted image 20240612214725.png]]

if we use nmap -h we find this section


one of the cool techniques to evade th eids

is to use fragmented packets fragmenting it to smaller packets so they cant tell if this is a packet that should be flagged or not .

While running this 

![[Pasted image 20240612214850.png]]

we go to wireshark to see whats happening behind the scenes

![[Pasted image 20240612214958.png]]

all packets are being sent as a full packet and not fragmented 

output has come !!
![[Pasted image 20240612215041.png]]


now lets try fragmenting it 

`nmap -Pn -sS -sV -p80,445,3389 -f 10.4.27.83`

lets limit the ports to better understanding and now lets go back to wireshark !

![[Pasted image 20240612215140.png]]

we can see that the packets are fragmented they are ip because fragmentation happens at the network layer of the osi model .


![[Pasted image 20240612215227.png]]

here we sent 2 packets that made a full packet sent to port 445 .


![[Pasted image 20240612215258.png]]

the first had fragment offset set to 0 but this one has it set to 8 now if we go back to  btw we can see the syn being sent to port 445 and we get a syn back from 445 .

![[Pasted image 20240612215358.png]]

we can see the scan is complete .

now if we go to the nmap help page we can see a 

![[Pasted image 20240612215425.png]]

--mtu 

for example we use the same scan as we did but we add --mtu

![[Pasted image 20240612215509.png]]


now we go to wireshark 



![[Pasted image 20240612215603.png]]

we cant see any fragmented packets cuz we set the mtu to 32 mtu stands for the minimum transmission units size to each packet

because we set it to 32 that its not fragmenting any of the packets 


![[Pasted image 20240612215756.png]]

we try lowering it

![[Pasted image 20240612215804.png]]

now we can see the fragments 

when we go to the network layers we see the length is 8 

![[Pasted image 20240612215830.png]]

now lets use decoy ip addresses to try evading the firewall 

we do 
`ifconfig `

![[Pasted image 20240612215901.png]]

as we know the first ip in the network is served for the gateway so if a network admin or anyone going through the traffic are being sent from the gateway ip the only thing we need is to be connected to the same network .

before we go with that lets highlight this nmap option becauase its important 

its the --ttl that is always very very useful

and the --data-length that allows you to appened random data 

now lets combine all of them include the use of decoy ips

`nmap -Pn -sS -sV -p445,3389 -f --data-length 200 -D 10.10.23.1,10.10.23.2  (target) `

the ip i provided i got via ifconfig and changeing the inet ip to the gateway which is the first in the subnet for example i did ifconfig and found 192.168.1.24 what i do is 192.168.1.1


when we go to wireshark we see this 


![[Pasted image 20240612220431.png]]

1 - First we have fragmentation 

because the packets are much larger they got fragmented into very smaller pieces

2 - ![[Pasted image 20240612220548.png]]

when we pay attention to the source we can see its being sent from the gateway ip or decoy ip we used how cool is that !



we can change the source port to make it even less sus 

`nmap -Pn -sS -sV -p445,3389 -f --data-length 200 -g 53 -D 10.10.23.1,10.10.23.2  (target) `

![[Pasted image 20240612220736.png]]

we can see the source port is 53 !!!!!!! 









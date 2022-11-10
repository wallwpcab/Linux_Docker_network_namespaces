# Linux_Docker_network_namespaces
It's a simple network namespace diagram

Our mission is to create two network namespaces,two virtual ehternat cables and a bridge and connect them according to the diagram 
given above.Finally we will communicate between the network namespaces using ping command.

Procedure:

1.Create two namespaces using following command:
   sudo ip netns add red (It will create a namespace called red in the linux namespace)
   sudo ip netns add green(It will create a namespace called green in the linux namespace)

2.Now create a bridge:
   sudo ip link add br0 type bridge(It will create a bridge in the namespace)

3.Create 2 virtual ehternet cables:
   sudo ip link add rveth type veth peer name rbveth
   sudo ip link add gveth type veth peer name gbveth

4.Attach the Veth Cables to their respective namespaces:
   sudo ip link set rveth netns red
   sudo ip link set gveth netns green

5.Now up the bridge interface:
   sudo ip link set br0 up 


6.Now up the network interfaces in the namespaces above:
   sudo ip netns exec red ip link set lo up
   sudo ip netns exec red ip link set rveth up
   sudo ip netns exec green ip link set lo up
   sudo ip netns exec green ip link set gveth up

7.Assign ip addresses to Each Namespaces
   sudo ip netns exec red ip addrress add 10.0.1.2/24 dev rveth
   sudo ip netns exec green ip address add 10.0.1.3/24 dev gveth
  
8.Now we can set the ip of the bridge:
   sudo ip addr add 10.0.1.1/16 dev br0

9.We can add the bridge veth interfaces to the bridge by setting the bridge device as their master:
   sudo ip link set rbrveth master br0
   sudo ip link set gbrveth master br0

10.With the bridge device set, it's time to connect the bridge side of the veth cable to the bridge:
    sudo ip link set rbrveth up
    sudo ip link set gbrveth up
11.Now we use ping command to communicate between the two network namespace:
    sudo ip netns exec red ping 10.0.1.3
    sudo ip netns exec green ping 10.0.1.2

12.To delete the namespaces use the following commands:
    sudo ip netns del red
    sudo ip netns del green

13.To delete bridge use following:
    sudo ip link delete br0 type bridge

14.To delete unattached veth interfaces:
    sudo ip link delete <interface_name>
    
Using Dockerfile to test

1.Run the following commands to build and run the container:
   docker build -t docker_bridge_test .
   docker run --privileged docker_bridge_test:latest
   
2.After running the container using:
   docker ps

3.Now get inside the container using:
   docker exec -it <container_id> bash

4.After getting inside docker container use above mentioned commands sequentially without using the sudo keyword

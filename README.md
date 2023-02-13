# Big-Data-Project
Introduction to Hadoop and Spark with a simple cluster setup and a demo notebook

## Descriptions of the files:
-  Vagrantfile: configuration of the virtual machines 
-  bootstrap.sh: provision file, used at the first launch of the machines
-  conf: configuration files for hadoop and spark
-  UK-traffic-accidents: contains the dataset
-  UK-traffic-accidents.ipynb: demo notebook on the dataset


## Prerequisites: 
- VirtualBox
- Vagrant

## Description of the setup
The cluster is formed by three separate virtual machines and has the following structure: 

- node-master: Namenode and ResourceManager
- node1 and node2: Datanodes and NodeManager

The machine are connected by a host-only network (subnet 192.168.33.0/24). Addresses are the following: 
- node-master: 192.168.33.2
- node1: 192.168.33.10
- node2: 192.168.33.11
 
## Configuring the environment 
The configuration of the machines is almost fully automated, still a few steps are necessary. 

- Boot the machines. The first time, vagrant will download the box and create a master machine from which deriving three linked clones, and for each machine it will run the provision.
  ```
  vagrant up
  ```
- Connect to the machines via ssh. In three separate windows run the following commands: 
  ```
  vagrant ssh node-master
  ```
  ```
  vagrant ssh node1
  ```
  ```
  vagrant ssh node2
  ```
- Check if the private network is working properly. On one of the machines (or all, if you want), try to ping the other ones: 
  ```
  ping node-master # equivalent to ping 192.168.33.2, aliases are contained in the file /etc/hosts.
	```
- All the following commands are to be run by the user ubuntu, both in the master and the slave machines. If you are not logged in as ubuntu, do the following:
  ```
  sudo -i -u ubuntu
  ```
- Hadoop needs ssh to operate, so it is necessary to exchange the ssh keys between the master and the slaves On node-master run the following: 
  ```
  ssh-keygen -b 4096
  ```
  this will guide you to the generation of a couple of RSA keys. Skip all the prompts by pressing enter until the end.

- Once you have generated a key pair, you need to copy them into the file /home/ubuntu/.ssh/authorized_keys. To do so, type
  ```
  cat /home/ubuntu/.ssh/id_rsa.pub
  ```
- Copy the key

- In the slave machines,create a file /home/ubuntu/.ssh/master.pub and copy there the key. Then add it to the list of authorized keys.
  ```
  echo <KEY_HERE> >/home/ubuntu/.ssh/master.pub
  cat /home/ubuntu/.ssh/master.pub >> /home/ubuntu/.ssh/authorized_keys
  ```
- Also copy the key in the node-master: 
  ```
  cat /home/ubuntu/.ssh/id_rsa.pub >> /home/ubuntu/.ssh/authorized_keys
  ```

- Test the connection from node-master to localhost, node1 and node2
  ```
  ssh localhost
  ssh node1
  ssh node2
  ```
- If this works, you're ready to set up the hdfs and yarn. On node-master, run the following commands: 
  ```
  hdfs dfs -format # this will create the necessary directories in namenode and datanodes 
  hdfs dfs -mkdir /spark-logs # this is necessary when running spark to access the logs using the GUI
  start-all.sh # this will launch both the hadoop demon and yarn 
  $SPARK_HOME/sbin/start_history_server.sh
  ```
- Now, if all is good, you should have all the services working and available on the web interface. To access them, just open a browser and insert the urls: 
  > - 192.168.33.2:8088   => yarn dashboard
  > - 192.168.33.2:9870   => hadoop hdfs dashboard
  > -  192.168.33.2:18080 => spark historyserver
 
- To check if all the services are up, you can run the following:
  ```
  jps 
  ```
  you should see Namenode,SecondaryNameNode and ResourceManager in node-master, and DataNode and NodeManager in the slaves. 
  
- To work on the dataset, you'll need to put it in the hdfs: 
  ```
  hdfs dfs -put /vagrant/UK-traffic-accidents /user/ubuntu
  ```
  NOTE: sometimes, for reasons unknown to me, the user folder doesn't get created, so it is necessary to create it by hand
  ```
  hdfs dfs -mkdir /user && hdfs dfs -mkdir /user/ubuntu
  ```
- Since pyspark launches a jupyter server which is listening on the loopback interface only, it is necessary to forward the port 8888 to the host in order to access the notebooks.
  Close the connection with node-master and reopen it with the option -L 8888:localhost:8888
  ```
  vagrant ssh node-master -- L 8888:localhost:8888
  ```
- To open the notebook, copy it in the home folder and launch pyspark
  ```
  cp /vagrant/UK-traffic-accidents.ipynb /home/ubuntu
  cd
  pyspark
  ```
- Now, on the host machine, you should be able to access the jupyter server at the address which is displayed on the terminal.
- Spark is already configured to use yarn as master. When running the notebook, on the yarn dashboard you should see the active job along with all the informations about nodes status and whatsoever. Enjoy!

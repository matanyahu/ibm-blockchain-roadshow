# IBM Blockchain Roadshow Lab 1

## This tutorial will help you to create a Marbles application on your computer

### **1. Prerequisites**

- Initial setup of Vagrant box and environment complete
- Access to the Internet

### **2. Now let's modify Vagrant file in order to forward a Marbles website to our host:**

- Go to the folder where you hold your Vagrant configuration (it should be something like fabric/devenv) and open **Vagrantfile** with your favourite editor. Then, search for the following line:

	```
Vagrant.require_version ">= 1.7.4"
Vagrant.configure('2') do |config|
  config.vm.box = "ubuntu/xenial64"
  ...
  ```

- Add the following line of code and save the file:

  ```
  config.vm.network :forwarded_port, guest: 3001, host: 3001, id: "marbles", host_ip: "localhost", auto_correct: true
    ```

- run Vagrant box:

	```
  vagrant up
	```

- The setup should be complete very fast as your VM is already in place. You should be able to ssh into the Vagrant VM just created:

	```
  vagrant ssh
	```

- This should log you into the box to the following folder: **/opt/gopath/src/github.com/hyperledger/fabric**
- Verify if you can use these commands:

```
cryptogen
configtxgen
configtxlator
peer
```

- If they are not recognized, export your binary folder again and try afterwards:

```
export PATH=/opt/gopath/src/github.com/hyperledger/fabric/bin:$PATH
```

- We will be using remaining commands inside the box.

### **3. Let's now clone Marbles repository**

- Clone the current Marbles (v4.0) git repository:

```
git clone https://github.com/IBM-Blockchain/marbles.git --depth 1
```

### **4. Let's run a test sample environment to host Marbles instance**

- Go to fabric-samples folder and run Hyperledger Fabric 1.0 test instance:

```
cd fabric-samples/fabcar/
./startFabric.sh
```

- This process will take around 2 minutes to complete with the following message:

```
...
2017-10-27 05:30:41.264 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00a Chaincode invoke successful. result: status:200
2017-10-27 05:30:41.265 UTC [main] main -> INFO 00b Exiting.....

Total execution time : 50 secs ...
```

### **4. Just a few clicks away from your Marbles instance...**

- Go back to marbles folder and install necessary nodejs modules:

```
cd ../../marbles/
npm install
npm install gulp -g
```

- If you receive an error message while installing gulp, try executing it with **sudo**:

```
sudo npm install gulp -g
```

- Now install and instantiate Marbles chaincode onto your Hyperledger Fabric test network:

```
cd scripts
node install_chaincode.js
node instantiate_chaincode.js
```

- **install_chaincode.js** will read in our marbles config file and the blockchain creds file. You can change the marbles chaincode ID or version by editing the install_chaincode.js file. **instantiate_chaincode.js** will have the peer spin up the marbles chaincode for your channel mychannel. Once this is complete we are ready to use the blockchain network to record our marble activities.

### **5. We are now ready to run Marbles:**

- Go back to Marbles root folder and execute the nodejs server:

```
cd ..
gulp marbles_local
```
- If all goes fine you will see the following log:

```
[05:31:36] Using gulpfile /opt/gopath/src/github.com/hyperledger/fabric/marbles/gulpfile.js
[05:31:36] Starting 'env_local'...
[05:31:36] Finished 'env_local' after 108 μs
[05:31:36] Starting 'build-sass'...
[05:31:36] Finished 'build-sass' after 16 ms
[05:31:36] Starting 'watch-sass'...
[05:31:36] Finished 'watch-sass' after 72 ms
[05:31:36] Starting 'watch-server'...
[05:31:36] Finished 'watch-server' after 91 ms
[05:31:36] Starting 'server'...
info: Checking credentials file is done
info: Loaded config file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/marbles_local.json
info: Loaded creds file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/blockchain_creds_local.json
info: Loaded config file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/marbles_local.json
info: Loaded creds file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/blockchain_creds_local.json
debug: cache busting hash js 1509082299946 css 1509082299946
Loaded config file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/marbles_local.json
Loaded creds file /opt/gopath/src/github.com/hyperledger/fabric/marbles/config/blockchain_creds_local.json


----------------------------------- Server Up - localhost:3001 -----------------------------------
------------------------------------------ Websocket Up ------------------------------------------
```

- Now you may switch back to your host's web browser and type **http://localhost:3001**

### **5. We are now ready to run Marbles:**

- In your browser you do not need to enter a password or change the prefilled username of `admin`.

![](/images/localhost2.png)


- Next the set up panel should pop up. Ideally it will walk itself through the 3 stages of initial setup.

- Enroll Admin - this step is communicating with your network's CA to verify the admin user credentials (enrollID/enrollSecret)
		- If it fails double check the enrollID and enrollSecret fields in your blockchain credentials file
- Finding Chaincode - this step is looking for the marbles chaincode on your peer. It is using the chaincode ID found in your blockchain credentials file. If this is a brand new network it will not exist yet.
		- If the chaincode was instantiated but it was unable to find it try the "Retry" button.
- Register Marble Owners - this step will create the marble owners you specified in the marbles configuration JSON file
		- This can take awhile 1-2minutes. Check your console logs for progress.

![](/images/localhost3.png)

- Once you see this message you are good to go:

![](/images/localhost4.png)

- Open up your favorite browser and browse to [http://localhost:3001](http://localhost:3001) or your Bluemix www route.
    - If the site does not load, check your node console logs for the hostname/ip and port marbles is using.

- Finally we can test the application. Click the "+" icon on one of your users in the "United Marbles" section

	![](/images/use_marbles1.png)

- Fill out all the fields, then click the "CREATE" button
- After a few seconds your new marble should have appeared.
    - If not refresh the page by hitting the refresh button in your browser, or by pressing F5
- Next let’s trade a marble.  Drag and drop a marble from one owner to another. Only trade it to owners within "United Marbles" if you have multiple marble companies. It should temporary disappear and then redraw the marble within its new owner. That means it worked!
    - If not refresh the page
- Now let’s delete a marble by dragging and dropping it into the trash can. It should disappear after a few seconds.

	![](/images/use_marbles2.png)

- efresh the page to double check that your actions "stuck".
- Use the search box to filter on marble owners or marble company names.  This is helpful when there are many companies/owners.
    - The pin icon will prevent that user from being filtered out by the search box.
- Now lets turn on the special walk through. Click the "Settings" button near the top of the page.
	- A dialog box will open.
	- Click the "Enabled" button to enabled Story Mode
	- Click the "x" in the top right to close the menu.
	- Now pick another marble and drag it to another user.  You should see a breakdown of the transaction process. Hopefully it gives you a better idea of how Fabric works.
	- Remember you can disable story mode when it becomes frustratingly repetitive and you are a cold husk of your former self.
- Congratulations you have a working marbles application :)!

### **6. Shutting down and cleaning up the environment for the next excercise:**

- Press ctrl+c in your Vagrant VM to cancel nodejs application
- Go to the fabric-samples folder to shut down the network:

```
cd ../../fabric-samples/basic-network/
./teardown.sh
```

- Some artifacts are still left running so you need to delete them manually:

```
docker rm $(docker ps -a -q)
docker rmi $(docker images -q "dev-*")
```

## Congratulations! You have now finished Lab 1!

Setup process of velociraptor in Linux Ubuntu AMD64 variant:
1. Visit the [official Velociraptor Downloads](https://docs.velociraptor.app/downloads/) page and download the Linux AMD64 binary.
2. Open a terminal and navigate to the download directory.
3. Make the binary executable:
	```bash
	chmod +x velociraptor-vx.x.x-linux-amd64
	```
4. (Optional but recommended) Install it system-wide:
	```bash
	sudo mv velociraptor-v0.77.1-linux-amd64 /usr/local/bin/velociraptor
	```
	saving it into `/usr/local/bin` allows to access `velociraptor` throughout the Linux system.
5. Verify that the binary is a valid Linux executable:
	```bash
	file /usr/local/bin/velociraptor
	```
	The output should begin with:
	```bash
	ELF 64-bit LSB executable
	```
	Anything other than this such as `Not Found` or `ASCII text` will mean failure.
6. Verify the installation:
	`velociraptor version`
	`which velociraptor`
7. Velociraptor is now installed and ready to use.

With this 7 step the velociraptor is installed in the laptop.

On let's look at **windows** server setup, which is also pretty simple as the Linux setup:
1. Open CMD as an administrator and make a directory to your desired path.
2. Download the executable file for windows from [official website](https://docs.velociraptor.app/downloads/)
3. Copy the exe file in the directory.
4. Then open powershell, go to the download directory and write: 
	`velociraptor-v0.77.1-windows-amd64.exe config generate -i`, the v0.77.1 is the version I downloaded, you may have later version downloaded.
5. Answer the question accordingly (make sure to choose Windows).
6. If prompted about using experimental WebSocket, enter **"No"**
7. If prompted about using the registry to store writeback files, please enter **"N"**
8. When asked about which **DynDNS** provider is used, select **None** and press enter.
9. For the GUI username, put username and password (multiple users can be set, if you are done press `ctrl + c`)
10. If it asks if you would to **"restrict VQL"** functionality on the server, please enter **"N"**
11. When it asks where to write the server and client configs, **just hit enter** on both prompts to accept the defaults. (basically they are the same path you choose to put the path of the datastore directory). with this we will have a `server.config.yaml` file.
12. Let's start server: `velociraptor-v0.72.3-windows-amd64.exe --config server.config.yaml frontend -v`
13. visit `<your_server_url>:<gui_port_address>/app/index.html`.
14. Some browser's may give a caution alert. Simply Go to advanced and Select Continue.

 Now lets get to know some basics of Velociraptor.

> [!note] Take a note!!
>From here onwards you will see **velociraptor** in the command lines example. For example in the above Linux command I wrote `velociraptor version`. Now what you need to understand is it may not work on your machine. In my case I stored the velociraptor's binary files in `usr/local/bin` which makes velociraptor a part of the system of Linux. If you used any other path, you have to write `./velociraptor`. If you are on a windows machine you have to write `./velociraptor.exe`. For simplicity I will be writing just `velociraptor` for the rest of the article.
## Structure of Velociraptor
Velociraptor consists of:
- **Server**: central management system.
- **Clients**: an instance of Velociraptor running on the endpoint, that is it’s our endpoint “agent”.
- **Frontend**: is the server component that communicates with the client. 
- **GUI**: is the web application server that provides the administrative interface.
- **API**: is the gRPC-based API server.
<div align="center">  
<img src="Velociraptor client setup.png" width="1000">  
</div>
# The Purpose of Each Component
## 1. Server
The **server** is the central controller of the entire deployment. According to the documentation, it is responsible for:
- maintaining client registrations
- storing collected forensic evidence
- scheduling hunts
- distributing artifacts
- managing users and organizations
- authenticating clients
- storing notebooks and timelines
- coordinating all communications
Think of the server as the **brain** of Velociraptor. Without the server:
- clients don't know what to collect,
- analysts cannot issue investigations,
- evidence cannot be centrally stored.

## 2. Client
The client is the endpoint agent installed on every monitored machine. The documentation emphasizes an important point:
>[!note] **There is not a separate client binary and server binary.** Velociraptor provides one binary per operating system, and command-line options determine whether it behaves as a client or a server. On the next section when we will setup server and client it will be easier to understand

A client's responsibilities are:
- enroll with the server
- periodically contact the server
- receive tasks
- execute VQL queries
- collect forensic artifacts
- upload results
The client never decides what investigation to perform. It simply executes instructions sent by the server.

## 3. Frontend
Many beginners confuse the **Server** and the **Frontend**. They are related but not identical. The **Frontend** is the server component responsible for network communication with clients.
It:
- listens for HTTPS connections
- authenticates clients
- receives uploaded evidence
- sends queued tasks
- manages client enrollment
Every client communicates with the Frontend. Think of it as the **communication gateway** of the server.
Client |> sends HTTPS request |>Frontend |> Server

## 4. GUI (Graphical User Interface)
The GUI is **only for investigators**. It is a web application accessed through a browser.
It allows analysts to:
- search clients
- launch artifact collections
- create hunts
- inspect collected evidence
- build notebooks
- review timelines
- manage users
- configure organizations
The GUI never communicates directly with clients. Instead:
Investigator's Browser |> GUI |> API |> Server

## 5. API
The API sits between the GUI and the server. Its purpose is to expose server functionality programmatically. The GUI itself uses the API internally. Other software can also use it for automation.
Examples include:
- creating hunts
- downloading results
- listing clients
- triggering artifact collections
- integrating with SIEM or SOAR platforms
So the API is essentially the **programmatic interface** to the server.

A more realistic way to look at the whole velociraptor is to see this below illustration:
![[VRDiagram.png]]
# How They Work Together
Imagine a SOC analyst wants to determine whether malware is running on 500 computers. Here is the step he follows:
1. The analyst opens the GUI. Browser |> GUI. GUI displays all enrolled clients.
2. The analyst launches an artifact. Example: `Generic.Clients.Info` or `Windows.Process`
3. The GUI sends the request to the API. GUI |> API
4. API stores the request on the server. API |> Server.
	The server records:
	- target clients
	- artifact parameters
	- execution status
5. Clients periodically "check in" with the Frontend.  Client |> HTTPS |> Frontend. 
	Rather than the server pushing commands, clients **poll** the server for work. This outbound communication model helps clients operate behind NAT and firewalls.
6. The Frontend responds: `RUN this VQL query`. The client downloads the task.
7. The client executes the VQL locally. For example: `SELECT Name, CommandLine FROM pslist()` or `Collect Browser History`.
8. The client compresses the results.
	`Running Processes`
	`Chrome History`
	`Event Logs`
	`Registry Keys`
9. The client uploads the results back. Client |> HTTPS |> Frontend |> Server
10. The server stores the result. The API retrieves them. The GUI displays them. The investigator now has the collected evidence without ever logging into the endpoint directly.

# Responsibilities at a glance

| Component    | Primary Responsibility                                                                                                | Communicates With |
| ------------ | --------------------------------------------------------------------------------------------------------------------- | ----------------- |
| **GUI**      | Web interface for investigators to manage hunts, artifacts, and results                                               | API               |
| **API**      | Programmatic interface that processes requests from the GUI or external tools                                         | GUI, Server       |
| **Server**   | Central coordinator that stores data, schedules work, and manages users, artifacts, and client state                  | API, Frontend     |
| **Frontend** | Network-facing communication layer that receives client check-ins, distributes tasks, and accepts uploaded evidence   | Server, Clients   |
| **Client**   | Endpoint agent that enrolls, polls the server, executes VQL queries, collects forensic artifacts, and returns results | Frontend          |
Velociraptor follows a **centralized command-and-collect architecture**. The **server** decides _what_ needs to be collected, the **frontend** handles secure communication, the **clients** perform the actual forensic work on endpoints, the **API** exposes the server's capabilities, and the **GUI** provides investigators with an intuitive interface to launch investigations and analyze the collected evidence. Together, these components enable scalable remote digital forensics and threat hunting across many systems.

# Velociraptor Server Setup
1. **Generate the configuration file:** Just like a Docker that has Daemon (background server) and Docker client, Velociraptor has similar server and client and just like docker uses YAML files to set the configuration parameter, Velociraptor uses it too. It stores how your server and client operate as well as how they communicate. To generate a config file we need to run:
	`velociraptor config generate -i`
	The `config generate` will generate a YAML configuration file and `-i` flag stands for interactive mode. It will ask some questions to the user about configuration and store them in YAML file. it is very important to use `-i` flag here. *If anything goes wrong in answering the question, it can be corrected by editing the YAML file*.
2.  **YAML file:** in the directory a new file named `server.config.yaml` has been created. Use VS Code or nano editor to see and edit configurations. *No need to touch the certificates.*
3. **Create Installation Package:** According to the YAML file, velociraptor can either create a Debian or RPM based server's installation file. here is the Debian based installation package creation command: 
	`velociraptor --config server.config.yaml debian server`; for windows you have to write `./server.config.yaml`.
	Now you will see a .deb file in the directory. mostly like this: `velociraptor-server-0.77.1.amd64.deb` 
4. **Install the server:** Now just install the server and it will automatically start the service at designated URL. Write this command in the CLI: `sudo dpkg -i velociraptor-server-0.77.1.amd64.deb`, change the 0.77.1 with the version you are using or have installed. After installation the server will automatically run and you can see the status by:
	`systemctl status velociraptor_server.service`.

	To stop the server: `systemctl stop velociraptor_server.service`
	To start the server: `systemctl start velociraptor_server.service`

5. **Visit the server:** server's GUI is running at `<given_url>:<gui_port_address>/app/index.html`. for example: `127.0.0.1:8889/app/index.html`.
6. **Optional Task:** some Velociraptor Artifacts are more complex than others and needs extra attention. Velociraptor has developed some side projects to strengthen the management of such artifacts. One can collect (add) `Server.Import.Extra` Artifact and use them.
	Let's see how can we collect artifact from the server. In the following example we will collect the Extras we just talked about.
	- On the left side bar, look for `server artifact` you will see a hard disk (stack) - like icon, 5th icon
	- On the upper part of left side, click on the add icon (New collection)
	- It will open a new dialog, search `Server.Import.Extras`
	- Look below the dialog and select Configure Parameters
	- you will see the list of all extras. select bin icon if you want to remove one.
	- If everything is set, look again to the bottom of dialog and select launch
	- This will start the process of collection the artifacts 
	- When the state (at the left of most) shows a (tic ✔️) sign it means the collection is completed.
	We have just collected out first artifact !!
	Feel free to explore the server
# Velociraptor Client Setup
Now it's time to setup the client side of velociraptor. Remember that, what makes velociraptor unique is that it is the responsibility of client to run the query according to the instruction of server that the admin wrote there. Client sends back to the server for the admin to inspect. Unlike autopsy app security analyst does not require the physical device to inspect security issues.
We will look into setups in three different kinds:
- Into the server itself
- Into a remote/client's Linux system
- Into a remote/client's Windows system

> [!tip] **What makes a system client or server?**
> We already mentioned that what we download from the official website is just a binary file (for Linux) and it does not segregate whether it is a server or client. The existence of `server.config.yaml` segregates this fact. If it exists and currently running then it is a server, if it is not then it is simply a potential client. The existence of `client.config.yaml` makes it a proper client. We will look into it now.

**Variant 1: Setting up client in the server itself**
If you have looked carefully in the `server.config.yaml` there is **Client** part. Why is it client there? Velociraptor made the .yaml file to  setup client too. We will fetch the client part and write it into a separate yaml file named `client.config.yaml` this will serve as the configuration file for all clients under this server we just set up.
To write a new `client.config.yaml` within the same directory as `server.config.yaml` we will just:
```bash
velociraptor --config server.config.yaml config client > client.config.yaml
```
It's easy to understand what is happening here. We are just taking everything under the client config part and writing them down in a new file called `client.config.yaml`.
Make sure to use `sudo` if you get permission related errors.
Now our `client.config.yaml` is ready. What comes next is to initialize a client within the server itself.
To do so we write: `velociraptor --config client.config.yaml client`; use `sudo` if error shows.
This command is quite similar to `velociraptor --config server.config.yaml debian server`, but instead of creating any Deb installer package it just initializes a client that always communicates with the server. 
The `client.config.yaml`'s server URL and ca certificate helps to connect with the correct server automatically.

After successfully doing everything you will see something like this:
![[init client velociraptor.png]]
It will be stuck here like this. No need to panic here. This is a good sign and signifies that the client is always communicating with the server. It is waiting for server's instruction...

In the server. At the top left corner's search box select the Show all
![[Show all client.png]]

After this you will be seeing a list of clients.
![[client list.png]]
Click on the client ID and you will get to see all the basic information regarding your client.
![[Client info picture.png | 600]]

Now let's stop exploring what we can do with client and move on to next type of installation.
**This setup of both server and client on same machine is useful for testing purposes.**

**Variant-2: Setting up Client in a Linux Machine**
This one is quite simple and we have already most of the work beforehand (in the variant 1). Follow every step in the server PC up until you get the `client.config.yaml`. This yaml file will be distributed in each and every client machine.
Make a separate folder (suggestion is to do in `usr/local/bin/` path) for velociraptor, download the binary file from the download page and make it executable, the path will be like this `usr/local/bin/velociraptor/<downloaded_binary>` and then in cli type `chmod +x <downloaded_binary>`
check if everything is ok by typing `velociraptor version`
Take the `client.config.yaml` file and copy it in the `usr/local/bin/velociraptor/` directory.
After that just type `sudo velociraptor --config client.config.yaml client` and your client is up and running.

> [!warning] **Do your client need to run the command every time system starts?**
> Yes, because the velociraptor client is not a system service and will not startup automatically as the system loads.
> To make it start with the system boot we need to make it a service. To do so we simply write 
> `sudo velociraptor --config client.config.yaml service install`, this will create a **systemed** service, named as **velociraptor.service**.
> Now enable the service by typing `sudo systemctl enable velociraptor`, this tells Linux to start this service automatically every time the system boots. Third step is to start the service before rebooting:
> `sudo systemctl start velociraptor`. Now the client will not require to write any kind of command to start the service. Client will start his Linux machine, the admin can see the client's machine connected to the server.

**Variant 3: Setting up Client in a Windows Machine**
According to the documentation the easiest way to set up velociraptor on client's windows machine is to generate msi package installer from the server artifact and install it in the client's machine. This msi package will already contain the `client.config.yaml` file so no need to copy it like previously done. Let's see how to do it step by step.

- Go to server artifacts (from the left bar) and select new collection (the add icon on the top left)
- Search for `Server.Utils.CreateMSI` and launch it.
- It will generate msi package installer for clients
- After it is completed, move to **Uploaded Files** tab, inside it you will find the MSI installer.![[msi installer in uploaded files.png]]
- Download the file.
- You are required to put the username and password before you proceed the download.
- After the download is finished install it on client's windows machine (not the server machine).
- After installation you can verify if the velociraptor service is running or not.
- Open Powershell as admin and type:  `Get-Service Velociraptor` and you will see something like this
	Status         Name           DisplayName
	------            ----               -----------
	Running  Velociraptor    Velociraptor

- In the Services of windows you can see it as running.

> [!tip] 
> For testing purposes in case you are considering to run Velociraptor Server on WSL and Client on the same actual windows machine. Don't! It will not work. Took a lot of time to figure it out


With this we have a decent idea to install and setup velociraptor on different systems. and How to establish connection between sever and client.

Next Article: [[Velociraptor Artifact]]
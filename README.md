# PolyEdge Docs

## Occupancy Tracking - How to get started - Tutorial
If using Edge device offered by Tiami Networks:

Switching device on:
Power the device. Please note that the voltage range is on devices provided by Tiami Networks follow voltage range of 100-240V AC for global compatibility. If PolyEdge is run on user devices, note the power compliances for B2x0 devices from Ettus Research. 

Enter device encryption password:
Enter the provided encryption password for the SDD.

Connect to WiFi &/ Ethernet:
Use the ethernet port on the device to connect to the AWS endpoint. 
Note: Wifi Sensing paradigms will disable wifi ports for transferring data, thus require Ethernet to access AWS endpoint benefits.

### Running PolyEdge:
PolyEdge leverages docker services for deployment, if you have the edge version of PolyEdge, all necessary services should already exist, please run 

sudo apt update && sudo apt upgrade

Starting Docker containers:
Utility scripts to provide user arguments for both NR and WiFi can be utilized for setting:
- Channel, bw for wifi or 
- dl arfcn/freq bw and scs for NR.

provided by the args


users can also simply run with docker compose up if utilizing the default args and/ modify docker-compose.yml for providing the args directly.

The ISAC commands and their usage for NR and WiFi through docker entrypoints are:

For 802.11:
./stream_wifi 
./recv_wifi
./visualize_wifi

For NR:
./stream_NR
./nr-uesoftmodem
./visualize_NR

### Login Page
Both methods provide communicate with AWS GreenGrass-IoT for user-endpoints. To get started, enter your username and password in the following login page. Note that the username is also case sensitive.
http://react-frontend.eba-ecjcuphd.us-west-2.elasticbeanstalk.com/


And to Log out, simply click on the “Logout” button located on the bottom of the entire dashboard

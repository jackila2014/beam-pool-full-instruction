# BEAM OPEN SOURCE POOL GUIDE 
(All credit to Greg of https://icemining.ca for his excellent efforts)

Open Source Beam Pool Software w/ Guide
DISCLAIMER: The Beam open source pool comes with no warranty and it is the sole responsibility of the user to ensure the functions and design of their own deployment, meet the required standards which a crypto-miner would expect to see in a mining pool. Where the code is complete in-as-much as the pool stratum, API and a barebones deployment of the GUI is functional and working, it is also the responsibility of the user to ensure that you have adept knowledge in pool systems.

The pool software is offered without a coin distribution function and this part is the sole responsibility of the user to add to the existing code. This part is deliberately omitted from the code to allow the user to determine which method payout they wish to utilise (PROP, PPS, PPLNS etc etc). By installing the pool software in this guide, you fully agree to these terms, and assure your prospective miners that you are able to create your own Beam distribution method.

My testing pool is at http://94.130.104.164:8010 - you can see what you will get by navigating to this link.

If you want to make any kind of donation to me for my ongoing guides, please point a GPU to 94.130.104.164:1690

All OK with that? then read on...

To begin compiling the Beam open source pool software, you will need to find a suitable VPS (Virtual Private Server) from a respectable hosting company. On the BEAM ACCEPTED HERE pages you can find a few who accept Beam for their services.

Once you have found a VPS, please log in as ROOT and you can start to set up the pool.

Firstly update your server.

    sudo apt update

    sudo apt upgrade

Next, install the required dependencies to build and run the pool

    sudo apt install python-dev build-essential libsodium-dev npm libboost-all-dev libboost-dev

You will also need to install NODEJS 10.x and MYSQL, separately from the above dependencies

    sudo apt install curl

    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -

    sudo apt-get install -y nodejs

    sudo apt install -y mysql-server

Update & Upgrade again just to make sure all these are current releases.

    sudo apt-get update

    sudo apt-get upgrade

Next, install REDIS SERVER

    sudo apt-get install redis-server

and enable the REDIS service

    sudo systemctl enable redis-server.service

This will then require a server reboot...

    sudo reboot now

Once reboot is complete, log into your server again as root and install php-redis

    sudo apt-get install php-redis

You are now ready to clone the repository which holds all the pool code!

    git clone https://github.com/r45ku1/beam-pool.git

Go into the beam-pool folder and make sure it's up to date.

    cd beam-pool

    npm update
    
    npm install

Now you need to configure your MYSQL Db's

    cd /etc/mysql/mysql.conf.d/

    nano /etc/mysql/mysql.conf.d/mysqld.cnf

and edit the line as shown:

    bind-address            = 0.0.0.0

save the file.

You will now want to build the Beam node and Wallet which will link to the pool stratum, and amend your config files.

    cd

    mkdir beam-node

    cd beam-node

    wget https://beam.mw/downloads/mainnet-linux (and use the node link )

    tar -xvf beam-node-VERSION.tar.gz

    wget https://beam.mw/downloads/mainnet-linux (and use the wallet link )

    tar -xvf beam-wallet-VERSION.tar.gz

Make a subfolder named stratum_secrets

    mkdir stratum_secrets

    cd stratum_secrets

You will need to save into this folder, one stratum.crt (certificate) and one stratum.key (key). To do this navigate to

  https://raw.githubusercontent.com/BeamMW/beam/master/utility/unittest/test.crt

make a new file in your current directory

    nano stratum.crt

Now copy & paste the contents of the page in your browser from the above link into that document you just created.

And same process with the navigation to

  https://raw.githubusercontent.com/BeamMW/beam/master/utility/unittest/test.key

making the file in your current directory

    nano stratum.key

And copy & paste the contents of this page in your browser from the above link into that document you just created.

You will now have one file stratum.key and one file stratum.crt in this folder. Go back up a level

    cd ..

Next step will be to choose a password for the wallet and add it to your beam-wallet.cfg file. Open the beam-wallet.cfg file using a text editor

    nano beam-wallet.cfg

and put your password at the command line:

    pass=YOURPASSWORD

Uncomment the line by removing the single # sign.

Also amend there:

    node_addr=127.0.0.1:10127

Again, uncomment this line by removing the single # sign.Those are the only 2 configurations that must be done on beam-wallet.cfg
Save the file and close it.

You can now initiate your Beam Pool wallet

    ./beam-wallet init

Save the output info in a safe place. This is your seed phrase and wallet default address, into which all mined coins from the pool, will be stored prior to distribution to your miners

You will also want to set that default address to never expire

    ./beam-wallet change_address_expiration --address=YOURDEFAULTWALLETADDRESS --expiration_time=never

Next you will need two keys to configure them the node. Please run the following command

    ./beam-wallet export_miner_key --subkey=1

    ./beam-wallet export_owner_key

Take the keys and put in the beam-node.cfg on the lines

    key_mine=

&

    key_owner=

Uncomment the line by removing the single # signs on each line. Now edit #password for keys in beam-node.cfg so that it matches #password for the wallet in beam-wallet.cfg - this is the password that you created previously.

    pass=YOURPASSWORD

Whilst still within a text editor in beam-node.cfg, locate each of the below lines and amend;

    port=10127
    log_level=verbose
    file_log_level=verbose
    peer=94.130.104.164:10127,eu-nodes.mainnet.beam.mw:8100,us-nodes.mainnet.beam.mw:8100,ap-nodes.mainnet.beam.mw:8100,chinabeam.omg10086.com:8100,chinabeam1.omg10086.com:8100,chinabeam2.omg10086.com:8100,ap-hk-nodes.mainnet.beam.mw:8100,beam.omg10086.com:8100,beamseed.21mil.com:8100,beamseed.mmitech.info:8100
    stratum_port=3333
    stratum_secrets_path=./stratum_secrets/

Open a new command line window which you are prepared to leave open (or screen -dmS it) all the time - this command window will contain the perpetual node connection. It is imperative that this process is never stopped. Navigate into your beam-node folder and start the node by typing;

    ./beam-node

Whilst allowing the node to synchronise and depending on your device firewall settings, you may need to “allow” it to open all the ports required to run the pool. Have a separate terminal window open for extra commands (these are all the hard-coded ports which the pool will utilise)

    sudo ufw allow 10127

    sudo ufw allow 8010

    sudo ufw allow 80

    sudo ufw allow 1690

    sudo ufw allow 3333

    sudo ufw allow 6379

    sudo ufw allow 6543

    sudo ufw allow 3306
    
    sudo ufw allow 666

Let the node synchronise with the network by downloading the Beam blockchain. This may take a while, depending on your connection speed. Once fully synchronised, you will see your Beam node collect the most up to date block height. Compare the node height with https://explorer.beam.mw/ and ensure it matches before continuing. The % will also show 100% in the node printout.

Once the node is fully synchronised, use a command line window which you are prepared to leave open all the time (or screen -dmS it). Navigate within the command line to your beam-node folder and type:

    ./beam-wallet listen

This enables the Beam pool wallet to perpetually listen at the Beam node. Now let's move onto your MYSQL database setup. Use an third-party software (such as sequelpro) to manage your schema tables. You will also need to create a MYSQL USER outside of ROOT to manage your Beam database (which we will create as follows...)

    mysql -u root

    CREATE USER 'newMYSQLuser'@'localhost' IDENTIFIED BY 'MYSQLpassword';

    GRANT ALL PRIVILEGES ON *.* TO 'newMYSQLuser'@'localhost';

    FLUSH PRIVILEGES;

And confirm your new user has privileges

    SHOW GRANTS FOR 'database_user'@'localhost';

Now in your MYSQL third party software (sequelpro or similar) create your tables within a new database 'beam'

    CREATE TABLE `accounts` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
    `coinid` int(11) DEFAULT NULL,
    `username` varchar(96) DEFAULT NULL,
    `ip` varchar(96) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4;

    CREATE TABLE `blocks` (
     `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
     `coinid` int(11) DEFAULT NULL,
     `reward` float DEFAULT NULL,
     `time` int(18) DEFAULT NULL,
     `userid` int(11) DEFAULT NULL,
     `workerid` int(11) DEFAULT NULL,
     `height` int(11) DEFAULT NULL,
     `sharediff` float DEFAULT NULL,
     `blockdiff` float DEFAULT NULL,
     `confirmations` float DEFAULT NULL,
     `difficulty` float DEFAULT NULL,
     `blockhash` varchar(64) DEFAULT NULL,
     `category` varchar(32) DEFAULT NULL,
     `jobid` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=24 DEFAULT CHARSET=utf8mb4;


    CREATE TABLE `workers` (
        `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
        `userid` int(11) DEFAULT NULL,
        `ip` varchar(96) DEFAULT NULL,
        `name` varchar(96) DEFAULT NULL,
        `difficulty` int(11) DEFAULT NULL,
        `rigname` varchar(32) DEFAULT NULL,
        `time` int(18) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=19 DEFAULT CHARSET=utf8mb4;

Create a new folder for the API

    mkdir beam-api

Navigate into that folder and install the Beam Explorer API. From the link at https://github.com/BeamMW/beam/releases find and identify the most recent version of the Beam Wallet API

    wget https://github.com/BeamMW/beam/releases/download/APIVERSION.tar.gz

    tar -xvf https://github.com/BeamMW/beam/releases/download/APIVERSION.tar.gz

edit the file wallet-api.cfg using nano and add the following flags

password for the wallet
        pass=YOURPASSWORD
address of node
        node_addr=127.0.0.1:10127
path to wallet file
        wallet_path=../beam-node/wallet.db

EDIT YOUR CONFIGURATION FILES
LOCATION: beam-pool/config.json
should be amended in the REDIS sections

        "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "password": ""

and in the website section, to reflect the IP of the VPS upon which you are installing the pool (the below example is Raskul Beam Pool, please use your own IP, and port remains 8010).

        "website": {
        "enabled": true,
        "host": "94.130.104.164",
        "port": 8010,
        "stratumHost": "94.130.104.164",
        "stats": {

2. LOCATION beam-pool/pool_configs/beam.json
amend to specify the path to your stratum.key and stratum.crt

        },
        "tlsOptions": {
        "enabled": true,
        "serverKey": "../stratum_secrets/stratum.key",
        "serverCert": "../stratum_secrets/stratum.crt",
        "ca": ""

and in the MYSQL section (mposMode section) this should reflect the newMYSQLuser and MYSQLpassword you created earlier

        },
        "mposMode": {
        "enabled": true,
        "host": "127.0.0.1",
        "port": 3306,
        "user": "newMYSQLuser",
        "password": "MYSQLpassword",
        "database": "beam",
        "checkPassword": false,
        "autoCreateWorker": true
        }

3. LOCATION beam-pool/libs/beam-blockconf.js
amend const=api and the MYSQL pieces

        const coinid = 2423;
        const coin_symbol = 'BEAM';
        const confirmations = 240;
        const api = 'http://127.0.0.1:666';
        module.exports = async function() {
        const connection = await mysql.createConnection({
        host: '127.0.0.1',
        user: 'newMYSQLuser',
        password: 'MYSQLpassword',
        database: 'beam',

4. LOCATION beam-pool/libs/stats.js
amend the MYSQLconnection here;

        async function setupMysqlConnection() {
        this.connection = await mysql.createConnection({
        host: '127.0.0.1',
        user: 'newMYSQLuser',
        password: 'MYSQLpassword',
        database: 'beam',

INSTALL THE EXPLORER-NODE (API)

From the web page at https://github.com/BeamMW/beam/releases find the most recent version of the Linux Explorer Node

    wget https://github.com/BeamMW/beam/releases/download/EXPLORER-NODE.tar.gz

    tar -xvf EXPLORER-NODE.tar.gz

And amend the configuration file within this folder as follows

    nano explorer-node.cfg 

Peer address to point the local api server to

peer address

    peer=YOUR SERVER IP:PORT

port to start the local api server on

    api_port=666

owner key from wallet steps

    key_owner=YOUR_OWNER_KEY_FROM_NODE_CONFIGURATION

password for owner key

    pass=YOUR_PASSWORD_FROM_NODE_CONFIGURATION

TIME TO GET IT ALL RUNNING!

Give your server a quick reboot

    sudo reboot now

Once powered up and logged in, make sure you have your node 100% synch and screen it, buy going into beam-node folder and using the following command:

    screen -dmS NODE ./beam-node

Navigate into your beam-explorer folder and run the API using. the following command:

    screen -dmS API ./explorer-node

Set up your wallet listener from within the beam-node folder:

    screen -dmS LISTEN ./beam-wallet listen -n YOURSERVERIP:10127

RUN THE POOL

From within the beam-pool folder, run the command as follows;

    screen -dmS POOL node init.js

Your (very basic) pool GUI will then be publicly accessible from

http://YOUR_POOL_IP:8010

It is now your own responsibility to add in a payment distribution mechanism, and tidy up your GUI to report all the sections you want to show, we have done most of the work, so as previously mentioned, you need to do the rest.

Again, with massive thanks to Greg of https://www.icemining.ca for his brilliant work in making all of this possible.
Released with no warranty and no support assistance. 

Raskul.

armor-nodejs-pool
======================

High performance Node.js (with native C addons) mining pool for **Armor Network**.

**Attention! This pool only works with LEGACY ADDRESSES**
===
A legacy address is created as follows:
```
./walletd --create-wallet --wallet-type=legacy --wallet-file=/PATH/TO/Save/MyArmorLegacy.wallet
```

#### Table of Contents
* [**Attention! This pool only works with LEGACY ADDRESSES**](#attention-this-pool-only-works-with-legacy-addresses)
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Starting the Pool](#3-start-the-pool)
  * [Host the front-end](#4-host-the-front-end)
  * [Customizing your website](#5-customize-your-website)
  * [SSL](#ssl)
  * [Upgrading](#upgrading)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)


Features
===

#### Optimized pool server
* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Ability to configure multiple ports - each with their own difficulty
* Miner login (wallet address) validation
* Workers identification (specify worker name as the password)
* Variable difficulty / share limiter
* Set fixed difficulty on miner client by passing "address" param with "+[difficulty]" postfix
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* SSL support for both pool and API servers
* RBPPS (PROP) payment system

#### Live statistics API
* Currency network/block difficulty
* Current block height
* Network hashrate
* Pool hashrate
* Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, payout estimate, etc)
* Blocks found (pending, confirmed, and orphaned)
* Historic charts of pool's hashrate, miners count and coin difficulty
* Historic charts of users's hashrate and payments

#### Mined blocks explorer
* Mined blocks table with block status (pending, confirmed, and orphaned)
* Blocks luck (shares/difficulty) statistics

#### Smart payment processing
* Splintered transactions to deal with max transaction size
* Minimum payment threshold before balance will be paid out
* Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option
* Option to enable dynamic transfer fee based on number of payees per transaction and option to have miner pay transfer fee instead of pool owner (applied to dynamic fee only)
* Control transactions priority with config.payments.priority (default: 0).
* Set payment ID on miner client when using "[address].[paymentID]" login
* Integrated payment ID addresses support for Exchanges

#### Admin panel
* Aggregated pool statistics
* Coin daemon & wallet RPC services stability monitoring
* Log files data access
* Users list with detailed statistics

#### Pool stability monitoring
* Detailed logging in process console & log files
* Coin daemon & wallet RPC services stability monitoring
* See logs data from admin panel

#### Extra features
* An easily extendable, responsive, light-weight front-end using API to display data
* Onishin's [keepalive function](https://github.com/perl5577/cpuminer-multi/commit/0c8aedb)
* Support for slush mining system (disabled by default)
* E-Mail Notifications on worker connected, disconnected (timeout) or banned (support MailGun, SMTP and Sendmail)
* Telegram channel notifications when a block is unlocked
* Top 10 miners report
* Multilingual user interface


Community / Support
===

* [GitHub Wiki](https://github.com/armornetworkdev/armor-nodejs-pool/wiki)
* [GitHub Issues](https://github.com/armornetworkdev/armor-nodejs-pool/issues)
* [Telegram Group](https://t.me/ARMORCURRENCY)

Pools Using This Software
===
https://miningpoolstats.stream/armor

Usage
===
#### Requirements
+ [Node.js](http://nodejs.org/) v12.0+
  + For Ubuntu:
    ```
    curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash
    sudo apt-get install -y nodejs
    ```
    + Check nodejs version:
     ```
     node --version
     ```
      + Must return v12.0+.
        + Else, you can install "n", to run `sudo n [version to install]`:
       ```
        sudo npm install -g n
        n --help
        sudo n 12
       ```
  + Or use NVM(https://github.com/creationix/nvm) for debian/ubuntu.

+ [Redis](http://redis.io/) key-value store v2.6+
  + For Ubuntu:
  ```
  sudo add-apt-repository ppa:chris-lea/redis-server
  sudo apt-get update
  sudo apt-get install redis-server
  ```
    + If you have errors, like this:
      ```
      Reading package lists... Done 
      W: Skipping acquire of configured file '6.0/binary-amd64/Packages' as repository 'https://apt.llvm.org/bionic llvm-toolchain-bionic InRelease' doesn't have the component '6.0' (component misspelt in sources.list?) 
      W: Skipping acquire of configured file '6.0/i18n/Translation-en' as repository 'https://apt.llvm.org/bionic llvm-toolchain-bionic InRelease' doesn't have the component '6.0' (component misspelt in sources.list?) 
      W: Skipping acquire of configured file '6.0/cnf/Commands-amd64' as repository 'https://apt.llvm.org/bionic llvm-toolchain-bionic InRelease' doesn't have the component '6.0' (component misspelt in sources.list?)
      ```
      + Try to see packages: `sudo grep -rhE ^deb /etc/apt/sources.list*`
      + Then just open this file `sudo nano /etc/apt/sources.list`
      + And comment both lines with `#`
      ```
      #deb https://apt.llvm.org/bionic/ llvm-toolchain-bionic 6.0 main 
      #deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-6.0 main
      ```
        + then `run sudo apt update`
 
 Dont forget to tune redis-server:
 ```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo 1024 > /proc/sys/net/core/somaxconn
 ```
 Add this lines to your /etc/rc.local and make it executable
 ```
 chmod +x /etc/rc.local
 ```

* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`

* Boost is required for the cryptoforknote-util module
  * For Ubuntu: `sudo apt-get install libboost-all-dev`

* libsodium  
  * For Ubuntu: `sudo apt-get install libsodium-dev`

##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.

[**Redis warning**](http://redis.io/topics/security): It'sa good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).

**Do not run the pool as root** : create a new user without ssh access to avoid security issues :
```bash
sudo adduser --disabled-password --disabled-login your-user
```
To login with this user :
```
sudo su - your-user
```

#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/armornetworkdev/armor-nodejs-pool.git pool
cd pool

npm update
```

#### 2) Configuration

Copy the `config_examples/COIN.json` file of your choice to `config.json` then overview each options and change any to match your preferred setup.

Explanation for each field:
```javascript
/* Pool host displayed in notifications and front-end */
"poolHost": "your.pool.host",

/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "graft", // Must match the parentCoin variable in config.js

/* Used for front-end display */
"symbol": "GRFT",

/* Minimum units in a single coin, see COIN constant in DAEMON_CODE/src/cryptonote_config.h */
"coinUnits": 10000000000,

/* Number of coin decimals places for notifications and front-end */
"coinDecimalPlaces": 4,

/* Coin network time to mine one block, see DIFFICULTY_TARGET constant in DAEMON_CODE/src/cryptonote_config.h */
"coinDifficultyTarget": 120,

"blockchainExplorer": "http://blockexplorer.arqma.com/block/{id}",  //used on blocks page to generate hyperlinks.
"transactionExplorer": "http://blockexplorer.arqma.com/tx/{id}",    //used on the payments page to generate hyperlinks

/* Set daemon type. Supported values: default, forknote (Fix block height + 1), bytecoin (ByteCoin Wallet RPC API) */
"daemonType": "default",

/* Set Cryptonight algorithm settings.
   Supported algorithms: cryptonight (default). cryptonight_light and cryptonight_heavy
   Supported variants for "cryptonight": 0 (Original), 1 (Monero v7), 3 (Stellite / XTL)
   Supported variants for "cryptonight_light": 0 (Original), 1 (Aeon v7), 2 (IPBC)
   Supported blob types: 0 (Cryptonote), 1 (Forknote v1), 2 (Forknote v2), 3 (Cryptonote v2 / Masari) */
"cnAlgorithm": "cryptonight",
"cnVariant": 1,
"cnBlobType": 0,
"includeHeight":false, /*true to include block.height in job to miner*/
"includeAlgo":"cn/wow", /*wownero specific change to include algo in job to miner*/
"isRandomX": true,
/* Logging */
"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},
"childPools":[ {"poolAddress":"your wallet",
                    "intAddressPrefix": null,
                    "coin": "MCN",  	//must match COIN name in the child pools config.json
                    "childDaemon": {
                        "host": "127.0.0.1",
                        "port": 26081
                    },
                    "pattern": "^Vdu",  //regex to identify which childcoin the miner specified in password. eg) Vdu is first 3 chars of a MCN wallet address.
                    "blockchainExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_block",
                    "transactionExplorer": "https://explorer.mcn.green/?hash={id}#blockchain_transaction",
                    "api": "https://multi-miner.smartcoinpool.net/apiMerged1",
                    "enabled": true
                    }
]
/* Modular Pool Server */
"poolServer": {
    "enabled": true,
    "mergedMining":false,
    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "your wallet",

    /* This is the Public address prefix used for miner login validation. */
    "pubAddressPrefix": 343,

    /* This is the Integrated address prefix used for miner login validation. */
    "intAddressPrefix": 340,

    /* This is the Subaddress prefix used for miner login validation. */
    "subAddressPrefix": 439,

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem" // The SSL certificate authority chain

    "ports": [
        {
            "port": 3333, // Port for mining apps to connect to
            "difficulty": 2000, // Initial difficulty miners are set to
            "desc": "Low end hardware" // Description of port
        },
        {
            "port": 4444,
            "difficulty": 15000,
            "desc": "Mid range hardware"
        },
        {
            "port": 5555,
            "difficulty": 25000,
            "desc": "High end hardware"
        },
        {
            "port": 7777,
            "difficulty": 500000,
            "desc": "Cloud-mining / NiceHash"
        },
        {
            "port": 8888,
            "difficulty": 25000,
            "desc": "Hidden port",
            "hidden": true // Hide this port in the front-end
        },
        {
            "port": 9999,
            "difficulty": 20000,
            "desc": "SSL connection",
            "ssl": true // Enable SSL
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 100, // Minimum difficulty
        "maxDiff": 100000000,
        "targetTime": 60, // Try to get 1 share per this many seconds
        "retargetTime": 30, // Check to see if we should retarget every this many seconds
        "variancePercent": 30, // Allow time to vary this % from target without retargeting
        "maxJump": 100 // Limit diff percent increase/decrease in a single retargeting
    },

    /* Set difficulty on miner client side by passing <address> param with +<difficulty> postfix */
    "fixedDiff": {
        "enabled": true,
        "separator": "+", // Character separator between <address> and <difficulty>
    },

    /* Set payment ID on miner client side by passing <address>.<paymentID> */
    "paymentId": {
        "addressSeparator": ".", // Character separator between <address> and <paymentID>
        "validation": true // Refuse login if non alphanumeric characters in <paymentID>
        "validations": ["1,16", "64"], //regex quantity. range 1-16 characters OR exactly 64 character
        "ban": true  // ban the miner for invalid paymentid
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
        "enabled": true,
        "min": 10, // Minimum percent probability for share hashing
        "stepDown": 3, // Increase trust probability % this much with each valid share
        "threshold": 10, // Amount of valid shares required before trusting begins
        "penalty": 30 // Upon breaking trust require this many valid share before trusting
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, // How many seconds to ban worker for
        "invalidPercent": 25, // What percent of invalid shares triggers ban
        "checkThreshold": 30 // Perform check when this many shares have been submitted
    },

    /* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards 'loyal' miners by valuing younger shares higher than older shares. Remember adjusting the weight!
    More about it here: https://mining.bitcoin.cz/help/#!/manual/rewards */
    "slushMining": {
        "enabled": false, // Enables slush mining. Recommended for pools catering to professional miners
        "weight": 300 // Defines how fast the score assigned to a share declines in time. The value should roughly be equivalent to the average round duration in seconds divided by 8. When deviating by too much numbers may get too high for JS.
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 300, // How often to run in seconds
    "maxAddresses": 50, // Split up payments if sending to more than this many addresses
    "mixin": 5, // Number of transactions yours is indistinguishable from
    "priority": 0, // The transaction priority    
    "dynamicTransferFee": true, // Enable dynamic transfer fee (fee is multiplied by number of miners)
    "minerPayFee" : true, // Miner pays the transfer fee instead of pool owner when using dynamic transfer fee
    "minPayment": 100000000000, // Miner balance required before sending payment
    "maxPayment": null, // Maximum miner balance allowed in miner settings
    "maxTransactionAmount": 0, // Split transactions by this amount (to prevent "too big transaction" error)
    "denomination": 10000000000 // Truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, // How often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 60,
    "poolFee": 0.8, // 0.8% pool fee (1% total fee total including donations)
    "soloFee": 0, // solo fee
    "finderReward": 0.2, // 0.2 finder reward
    "devDonation": 0.2, // 0.2% donation to send to pool dev
    "networkFee": 0.0, // Network/Governance fee (used by some coins like Loki)

    /* Some forknote coins have an issue with block height in RPC request, to fix you can enable this option.
       See: https://github.com/forknote/forknote-pool/issues/48 */
    "fixBlockHeightRPC": false
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, // How many second worth of shares used to estimate hash rate
    "updateInterval": 3, // Gather stats and broadcast every this many seconds
    "bindIp": "0.0.0.0", // Bind API to a specific IP (set to 0.0.0.0 for all)
    "port": 8117, // The API port
    "blocks": 30, // Amount of blocks to send at a time
    "payments": 30, // Amount of payments to send at a time
    "password": "your_password", // Password required for admin stats
    "ssl": false, // Enable SSL API
    "sslPort": 8119, // The SSL port
    "sslCert": "./cert.pem", // The SSL certificate
    "sslKey": "./privkey.pem", // The SSL private key
    "sslCA": "./chain.pem", // The SSL certificate authority chain
    "trustProxyIP": false // Proxy X-Forwarded-For support
},

/* Coin daemon connection details (default port is 18981) */
"daemon": {
    "host": "127.0.0.1",
    "port": 18981
},

/* Wallet daemon connection details (default port is 18980) */
"wallet": {
    "host": "127.0.0.1",
    "port": 58082,
    "password": "user:pass"
},

/* Redis connection info (default port is 6379) */
"redis": {
    "host": "127.0.0.1",
    "port": 6379,
    "auth": null, // If set, client will run redis auth command on connect. Use for remote db
    "db": 0, // Set the REDIS database to use (default to 0)
    "cleanupInterval": 15 // Set the REDIS database cleanup interval (in days)
}

/* Pool Notifications */
"notifications": {
    "emailTemplate": "email_templates/default.txt",
    "emailSubject": {
        "emailAdded": "Your email was registered",
        "workerConnected": "Worker %WORKER_NAME% connected",
        "workerTimeout": "Worker %WORKER_NAME% stopped hashing",
        "workerBanned": "Worker %WORKER_NAME% banned",
        "blockFound": "Block %HEIGHT% found !",
        "blockUnlocked": "Block %HEIGHT% unlocked !",
        "blockOrphaned": "Block %HEIGHT% orphaned !",
        "payment": "We sent you a payment !"
    },
    "emailMessage": {
        "emailAdded": "Your email has been registered to receive pool notifications.",
        "workerConnected": "Your worker %WORKER_NAME% for address %MINER% is now connected from ip %IP%.",
        "workerTimeout": "Your worker %WORKER_NAME% for address %MINER% has stopped submitting hashes on %LAST_HASH%.",
        "workerBanned": "Your worker %WORKER_NAME% for address %MINER% has been banned.",
        "blockFound": "Block found at height %HEIGHT% by miner %MINER% on %TIME%. Waiting maturity.",
        "blockUnlocked": "Block mined at height %HEIGHT% with %REWARD% and %EFFORT% effort on %TIME%.",
        "blockOrphaned": "Block orphaned at height %HEIGHT% :(",
        "payment": "A payment of %AMOUNT% has been sent to %ADDRESS% wallet."
    },
    "telegramMessage": {
        "workerConnected": "Your worker _%WORKER_NAME%_ for address _%MINER%_ is now connected from ip _%IP%_.",
        "workerTimeout": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has stopped submitting hashes on _%LAST_HASH%_.",
        "workerBanned": "Your worker _%WORKER_NAME%_ for address _%MINER%_ has been banned.",
        "blockFound": "*Block found at height* _%HEIGHT%_ *by miner* _%MINER%_*! Waiting maturity.*",
        "blockUnlocked": "*Block mined at height* _%HEIGHT%_ *with* _%REWARD%_ *and* _%EFFORT%_ *effort on* _%TIME%_*.*",
        "blockOrphaned": "*Block orphaned at height* _%HEIGHT%_ *:(*",
        "payment": "A payment of _%AMOUNT%_ has been sent."
    }
},

/* Email Notifications */
"email": {
    "enabled": false,
    "fromAddress": "your@email.com", // Your sender email
    "transport": "sendmail", // The transport mode (sendmail, smtp or mailgun)

    // Configuration for sendmail transport
    // Documentation: http://nodemailer.com/transports/sendmail/
    "sendmail": {
        "path": "/usr/sbin/sendmail" // The path to sendmail command
    },

    // Configuration for SMTP transport
    // Documentation: http://nodemailer.com/smtp/
    "smtp": {
        "host": "smtp.example.com", // SMTP server
        "port": 587, // SMTP port (25, 587 or 465)
        "secure": false, // TLS (if false will upgrade with STARTTLS)
        "auth": {
            "user": "username", // SMTP username
            "pass": "password" // SMTP password
        },
        "tls": {
            "rejectUnauthorized": false // Reject unauthorized TLS/SSL certificate
        }
    },

    // Configuration for MailGun transport
    "mailgun": {
        "key": "your-private-key", // Your MailGun Private API key
        "domain": "mg.yourdomain" // Your MailGun domain
    }
},

/* Telegram channel notifications.
   See Telegram documentation to setup your bot: https://core.telegram.org/bots#3-how-do-i-create-a-bot */
"telegram": {
    "enabled": false,
    "botName": "", // The bot user name.
    "token": "", // The bot unique authorization token
    "channel": "", // The telegram channel id (ex: BlockHashMining)
    "channelStats": {
        "enabled": false, // Enable periodical updater of pool statistics in telegram channel
        "interval": 5 // Periodical update interval (in minutes)
    },
    "botCommands": { // Set the telegram bot commands
        "stats": "/stats", // Pool statistics
         "enable": "/enable", // Enable telegram notifications
        "disable": "/disable" // Disable telegram notifications
    }    
},

/* Monitoring RPC services. Statistics will be displayed in Admin panel */
"monitoring": {
    "daemon": {
        "checkInterval": 60, // Interval of sending rpcMethod request
        "rpcMethod": "getblockcount" // RPC method name
    },
    "wallet": {
        "checkInterval": 60,
        "rpcMethod": "getbalance"
    }
},

/* Prices settings for market and price charts */
"prices": {
    "source": "cryptonator", // Exchange (supported values: cryptonator, altex, crex24, cryptopia, stocks.exchange, tradeogre, maplechange)
    "currency": "USD" // Default currency
},

/* Collect pool statistics to display in frontend charts  */
"charts": {
    "pool": {
        "hashrate": {
            "enabled": true, // Enable data collection and chart displaying in frontend
            "updateInterval": 60, // How often to get current value
            "stepInterval": 1800, // Chart step interval calculated as average of all updated values
            "maximumPeriod": 86400 // Chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
        },
        "miners": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "workers": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "difficulty": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "price": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "profit": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        }

    },
    "user": { // Chart data displayed in user stats block
        "hashrate": {
            "enabled": true,
            "updateInterval": 180,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "worker_hashrate": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 60,
            "maximumPeriod": 86400
        },
        "payments": { // Payment chart uses all user payments data stored in DB
            "enabled": true
        }
    },
    "blocks": {
        "enabled": true,
        "days": 30 // Number of days displayed in chart (if value is 1, display last 24 hours)
    }
}
```

#### 3) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis
* `chartsDataCollector` - Processes miners and workers hashrate stats and charts
* `telegramBot`	- Processes telegram bot commands


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.

To keep your pool up, on operating system with systemd, you can create add your pool software as a service.  
Use this [example](https://github.com/armornetworkdev/armor-nodejs-pool.git/blob/master/deployment/cryptonote-nodejs-pool.service) to create the systemd service `/lib/systemd/system/cryptonote-nodejs-pool.service`
Then enable and start the service with the following commands :

```
sudo systemctl enable cryptonote-nodejs-pool.service
sudo systemctl start cryptonote-nodejs-pool.service
```

#### 4) Host the front-end

+ Simply host the contents of the `website_example` directory on file server capable of serving simple static files.
  + For example:
   ```
   sudo npm install http-server -g
   ```
    + then
      ```
      http-server pathwayTo/website_example -a 192.168.0.156 -p 8080
      ```
      + then go to http://192.168.0.156:8080
    + or
      ```
      http-server pathwayTo/website_example -a 192.168.0.156 -p 8080 -S --cert pathwayTo/localhost.crt --key pathwayTo/localhost.key
      ```
      + then go to https://192.168.0.156:8080

Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:4009";

/* Must match the "coin"-value in coin config.json, else this will be displayed twice... */
var parentCoin = "default"; //when "coin": "default"

/* Pool server host to instruct your miners to point to (override daemon setting if set) */
var poolHost = "poolhost.com";

/* Number of coin decimals places (override daemon setting if set) */
var coinDecimalPlaces = 8;

/* Contact email address. */
var email = "support@poolhost.com";

/* Pool Telegram URL. */
var telegram = "https://t.me/YourPool";

/* Pool Discord URL */
var discord = "https://discordapp.com/invite/YourPool";

/*Pool Facebook URL */
var facebook = "https://www.facebook.com/<YourPoolFacebook";

/* Market stat display params from https://www.cryptonator.com/widget */
var marketCurrencies = ["{symbol}-BTC", "{symbol}-USD", "{symbol}-EUR", "{symbol}-CAD"];

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/default.css";

/* Default language */
var defaultLang = 'en';

```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.

#### SSL
  + If you have no any SSL Certificate, or private key, you can generate this, as self-assigned certificate, [by the following guide](https://gist.github.com/username1565/7321d63b44e1241f992c00d76b833c9a).

  + You can configure the API to be accessible via SSL using various methods.
    + Using SSL api in `config.json`:
      + Need to switch `(config.json).api.ssl = true`, and set it true, to run API via HTTPS too, on start pool-backend.
        + By using this you will need to update your `api` variable in the `website_example/config.js`.
        + For example: `var api = "https://poolhost:4009";` (just, change API from HTTP to HTTPS), there, and see comments.

    + Find an example for nginx below. Inside your SSL Listener, add the following:
      ``` javascript
      location ~ ^/api/(.*) {
          proxy_pass http://127.0.0.1:4009/$1$is_args$args;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
      ```
      + By adding this you will need to update your `api` variable in the `website_example/config.js` to include the /api.
        + For example: `var api = "http://poolhost/api";`

      + You no longer need to include the port in the variable because of the proxy connection.

    + Using his own subdomain, for example `api.poolhost.com`:
    ```bash
    server {
        server_name api.poolhost.com
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl_certificate /your/ssl/certificate;
        ssl_certificate_key /your/ssl/certificate_key;

        location / {
            more_set_headers 'Access-Control-Allow-Origin: *';
            proxy_pass http://127.0.01:4010;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_cache_bypass $http_upgrade;
        }
    }
    ```
      + By adding this you will need to update your `api` variable in the `website_example/config.js`.
        + For example: `var api = "//api.poolhost.com";`
    
      + You no longer need to include the port in the variable because of the proxy connection.

#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### JSON-RPC Commands from Armor daemon
### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here: https://github.com/Armor-Network/armor/tree/master/docs
* Daemon https://github.com/Armor-Network/armor/blob/master/docs/Armor-Node-Daemon-JSON-RPC-API.md
* Wallet https://github.com/Armor-Network/armor/blob/master/docs/Armor-Wallet-Daemon-JSON-RPC-API.md

Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:58081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```

Documentation for JSON-RPC commands can be found here:
* Armor RPC https://github.com/armornetworkdev/armor/wiki

Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [Netdata](https://github.com/firehol/netdata)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever) or [PM2](https://github.com/Unitech/pm2)


Donations
===
https://discord.com/channels/801139879982399509/807983301519212617


Credits
---------

* [fancoder](//github.com/fancoder) - Developper on cryptonote-universal-pool project from which current project is forked.
* [dvandal](//github.com/dvandal) - Developer of cryptonote-nodejs-pool software
* [armornetworkdev](//github.com/armornetworkdev) - Armor Network Developers Team.

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html

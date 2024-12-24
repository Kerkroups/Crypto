#### ТОКЕНЫ:
- ERC-20
- ERC-721  

#### ТЕСТОВЫЕ СЕТИ:  
- Rinkbey.
- Ropsten.
- Goerli.  

#### ИНТСРУМЕНТЫ РАЗРАБОТКИ:  
- Truffle.
- Hardhat.
- Remix.
- Ganache.  


#### TRUFFLE + GANACHE:  
1. Install Truffle: https://archive.trufflesuite.com/docs/truffle/how-to/install/, ```npm install truffle -g```  
2. Install Ganache: https://archive.trufflesuite.com/ganache/
3. Initialixe new project: ```truffle init```
4. In Ganache select "Quickstart".
5. Edit truffle-config.js: uncomment "development" in network code block. Add from Ganache "network id" and "RPC port" (Settings->Server).
6. Prepare migrations file: ```cd migrations/```, create file e.g. 2_deploy_contract.js
7. Deploy contract: truffle migrate --network \[network name\]  

Содержимое файла 2_deploy_contract.js:  

```
const MyContract = artifacts.require("MyContract");

module.exports = function(deployer){
  deployer.deploy("MyContract");
}

```

After project initialization truffre create some folders and config files:
```
├── contracts // это место где Truffle ожидает найти все наши контракты.
    ├── Migrations.sol
├── migrations //  это место где создается файл JavaScript который сообщает Truffle как развернуть смарт-контракт.
    ├── 1_initial_migration.js
└── test // директория где буду храниться unit тесты.
truffle-config.js // конфигурационные файлы содержащие настройки сети.
truffle.js // конфигурационные файлы содержащие настройки сети.
```

#### HARDHAT:  
https://hardhat.org/tutorial  


#### REMIX IDE + METAMASK BROWSER EXTENSION:  
















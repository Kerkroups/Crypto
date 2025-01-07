#### ТОКЕНЫ:
- ERC-20: стандарт токенов, включает три основных функции **transfer**, **approve**, **transferFrom**. **transfer** - передает токен от msg.sender другому аккаунту. **approve** - подтвержение на использование другим аккаунтом токенов контракта. **transferFrom** - подтвержение на разрешение пересылки токена контракта другому акккаунту. **approve** и **transferFrom** зачастую используются в таком сценарии: пользователь собирается перечислить деньги в контракт, мы ообряем контракт, далее контракт вызывает **transferFrom** чтобы пеевести токены от нас в контракт. Эти шаги позволяю избежать случайного перечисления средств на другой адрес.

###### Отличия transfer() от transferFrom():  

**transfer()** — это прямой перевод токенов от одного пользователя другому. Пользователь сам решает, сколько и кому передать токенов.  
**transferFrom()** - используется в случае, когда пользователь (например, Alice) заранее разрешил смарт-контракту или другому пользователю тратить её токены. Это позволяет третьей стороне переводить средства с её аккаунта, но только в пределах заранее установленного лимита.  


```
interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(
        address owner,
        address spender
    ) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Approval(address indexed owner, address indexed spender, uint256 amount);
}
```

Сценарий 1:  

Пользователи: Alice, Bob.  
Smart contract (game): SomeGame.  
ERC-20 Token: GC ("gameCoin") - используется для ставок в игре.  

ШАГ 1: Alice разрешает смарт-контракту SomeGame использовать её токены.  

```
// Alice вызывает approve на своём токене
token.approve(address(SomeGame), 100); // Разрешает SomeGame потратить до 100 токенов с её счёта
```

Результат: Alice даёт разрешение контракту SomeGame тратить максимум 100 её токенов.  

ШАГ 2: Bob делает ставку в смарт контракте SomeGage.  

Теперь, когда Alice разрешила смарт-контракту SomeGame потратить её токены, Bob может делать ставку, даже если у него нет своих токенов. Для этого он вызывает функцию placeBet в смарт-контракте SomeGame, который использует transferFrom для перевода 100 токенов с аккаунта Alice на свой счёт (или на счёт смарт-контракта).  

```
// Функция для обработки ставки в смарт-контракте
function placeBet(uint amount) public {
    require(amount <= token.balanceOf(msg.sender), "Insufficient balance");

    // Переводим токены с Alice на контракт
    token.transferFrom(msg.sender, address(this), amount);

    // Логика игры (например, обработка выигрыша или завершение ставки)
    bets[msg.sender] = amount;
}
```  

```
// Bob вызывает функцию placeBet в смарт-контракте SomeGame
SomeGame.placeBet(100); // Bob делает ставку на 100 токенов
```

Смарт-контракт SomeGame будет вызывать transferFrom для перевода токенов с аккаунта Alice.  

```
// Смарт-контракт SomeGame вызывает transferFrom
token.transferFrom(aliceAddress, address(SomeGame), 100); // Переводит 100 токенов с Alice на контракт SomeGame
```

Поверка баланса в контракте:  

```
uint aliceBalance = token.balanceOf(aliceAddress); // 500 - 100 = 400
uint gameBalance = token.balanceOf(address(SomeGame)); // 100
```  

Отправка токенов непосредственно адрессату:  

```
// Alice переводит 100 токенов в смарт-контракт SomeGame
token.transfer(address(SomeGame), 100); // Переводит 100 токенов напрямую

// Alice переводит 100 токенов Bob
token.transfer(bobAddress, 100);
```



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

#### ДЛЯ НАЧАЛА:  
- **Contracts can be created “from outside” via Ethereum transactions or from within Solidity contracts.**
- https://docs.soliditylang.org/en/v0.8.9/contracts.html#creating-contracts  

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


#### MYTHRIL (SAST):  

1. Analyze file: ```myth analyze <solidity-file>```
2. Analyze contract by address: ```myth analyze -a <contract-address>```

Create report in html: ```cd docs && make html```  

Create report in pdf: ```cd docs && make latexpdf```  

#### SOLHINT:   

Rules: https://github.com/protofire/solhint/blob/develop/docs/rules.md


#### SOLC (SOLIDITY COMPILER):  

https://github.com/ethereum/solidity/releases  













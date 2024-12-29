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


#### VULNERABILITI IN ABI.ECNCODEPACKED():  

This function vulnerable to collision: https://docs.soliditylang.org/en/latest/abi-spec.html#non-standard-packed-mode  

#### VULNERABILITY IN SELFDESTRUCT():  

В смарт-контрактах функция **selfdestruct()** - это специальная функция, используемая в основном для прекращения работы контракта при определенных обстоятельствах и освобождения связанных с ним ресурсов.  

Основная цель функции **selfdestruct()** - позволить создателям контрактов уничтожить развернутый смарт-контракт.  

При срабатывании функции **selfdestruct()** смарт-контракт будет окончательно удален, а оставшиеся Ether, хранящиеся в контракте, будут отправлены на указанный целевой адрес. В то же время все данные и функции, связанные с контрактом (хранилище и код удалены из состояния), больше не будут доступны. Функция **selfdestruct()** может использоваться для отказа от контрактов, которые больше не используются, обновления версий контрактов или решения потенциальных проблем безопасности.  


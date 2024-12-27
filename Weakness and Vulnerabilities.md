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

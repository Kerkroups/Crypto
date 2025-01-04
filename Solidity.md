# СИНТАКСИС И КЛЮЧЕВЫЕ СЛОВА:  

```
// Зададим версию компилятора;
pragma solidity [версия компилятоа];

// Создадим пустой контракт;
contract ContractName {
  // Код контракта;
}
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ВИДЫ ХРАНИЛИЩ В SOLIDITY:  
- memory: временное хранилизе данных (хранятся на период выполнения функции), аналогичен RAM.
- storage: постоянное хранилище дынных (state переменные, хранятся в блокчейн), аналог HARD DRIVE.
- calldata: схож с memory, но доступен только для external функций (хранит аргументы функции). Помогает сэкономить gas.

По умолчанию Solidity интерпретирует переменные вне функции хранимые как в STORAGE, а внутри функции интерпретирует как хранимые в MEMORY которые исчезают по завершению вызова функции.  

Иногда эти ключевые слова необходимо использовать напрямую, например когда работаем со структурами и массивами внутри функции.  

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ПЕРЕМЕННЫЕ И ТИПЫ ДАННЫХ:  

В Solidity переменные бывают трех типов: local, state и global.  

**state** переменные хранятся в блокчейн. Определяются в контракте вне функции.  
**local** переменные определяются внутри функции и существуют только во время выполнения кода функции.  
**global** переменные предоставляют информацию о блокчейн.  
**constant** значение этих переменных устанавливается перед компиляцией кода и не может быть моджифицированно после. Расходуют меньше gas. Должны определяться в верхнем регистре.  ```uint256 public constant UINT_CONSTANT = 123;```  
**immutable** значение этих переменных пожоже на использование констант но отличается тем, что их можно использовать в конструкторе.  

 - uint256: беззнаковое целочисленное, число не может быть отрицательным.
 - int: знаковое целочисленное, число может быть положительным и отрицательным.
 - uint: пседвдоним для uint256, также существуют другие варианты uint: uint32, uint16, uint8.
 - string: строчный тип данных, может содержать данные произвольной длинны в кодировке UTF-8. Должен храниться в **memory**.

**Ссылочные типы данных**: 
Существует два способа момещения данных в аргументы функции:
  1. По значению, это означает, что компилятор Solidity  создает копию  значение переменной и передает в аргумент функции. Это позволяет функции модифицировать значение не оппасаясь, что изначальное значение  переменой было модифицированно.
  2. По ссылке, это означает, что функция работает с ссылкой на оригинальное значение ( не копией). В этом случае, функция модифицирует оригинальное значение переменной.

  Значения аргументов функции желательно записывать с “_" перед именем аргумента.

```
// [тип] [имя] = [значение];
uint number = 123;
```

Значение переменных по-умолчанию:  
int256 = 0;
uint256 = 0;  
bool = false;  
address = 0x0000000000000000000000000000000000000000;  
byre32 = 0x0000000000000000000000000000000000000000000000000000000000000000;  

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### СТРУКТУРЫ:  
Структуры это сложные, комбинированные типы данных которые имеют множество значений:  

```
// создадим структуру
struct testStructure {
  // Инициалилировали переменую с типом uint именем "a";
  uint a;
  // Инициалилировали переменую с типом "string" именем "b";
  string b;
}
```
Структуры должны храниться в **memory**.  

```
testStructure memory test1 = testStructure(1, "Hello World");
testStructure memory test2 = testStructure({a: 1, b: "String value"});
testStructure memory test3;

testStructure[] memory tests; // Массив структур.
tests.push(testStructure(1, "string"));

testStructure[] memory tests;
testStructure storage test = tests[0];  // ССылка на массив структур.  
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### МАССИВЫ:  
 - ```uint[3] fixedArr;```: массив фиксированной длины.
 - ```uint[] dynamicArr;```: динамический массив.
 -  ```testStructure[] public structArr;```: массив с типом данных структура. Такой тип эквивалентен сохранению данных в БД.

Если объявить массив как **public**, solidity автоматически создаст для него метод **getter**. Другие контракты могут читать данные из него, но не будут иметь право записи в этот массив. Массивы должны храниться в **memory**.  
```uint256[] memory a = new uint256[](3);```  

Массивы имеют несколько встроенных функций:  
- push: добавляет значение в конец массива.
- pop: выталкивает значения с конца массива, уменьшая длинну массива на 1 
- length: текущая длинна массива.
- delete: не меняет длину массива, задает значение переменной по индексу i на 0. ```delete arr[i]```.  

Обращение к элементу массива: ```arr[i];``` где arr это имя массива, i индекс элемента массива.  

Работа с массивами структур:  
```
struct Todo { // Создаем структуру;
        string text;
        bool completed;
    }
    
    Todo[] public todos; // Создаем массив структур;
    
    function create(string calldata text) external { // Функция добавляет значение text в массиы структур.
        todos.push(Todo({text: text, completed: false}));
    }
    
    function updateText(uint256 index, string calldata text) external {
        Todo storage todo = todos[index];
        todo.text = text;
    }
    
   function toggleCompleted(uint256 index) external {
        Todo storage todo = todos[index];
        todo.completed = !todo.completed;
    }

    function get(uint256 index) external returns (string memory, bool) {
        Todo storage todo = todos[index];
        return (todo.text, todo.completed);
    }
    
}
```

Сдвиг данных в массиве:  
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract ArrayShift {
    uint256[] public arr = [1, 2, 3, 4, 5];

    function remove(uint256 index) external {
        require(index < arr.length, "Index out of bounds"); // Проверка индекса на валидность

        // Сдвигаем элементы на одну позицию влево, начиная с индекса
        for (uint256 i = index; i < arr.length - 1; i++) {
            arr[i] = arr[i + 1];
        }

        // Убираем последний элемент массива
        arr.pop();
    }
}
```


-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ENUM:  

Solidity поддерживает перечисления, и они полезны для моделирования выбора и отслеживания состояния.  

```
enum Status {
  None, // 0
  Pending, // 1
  Shipped // 2
}

Status public status;

function get() external view returns (Status) {
        return status;
    }

function cancel() external {
        status = Status.Canceled; // Set variable with direct request to enum;
    }

    // Reset enum to it's default value, 0
function reset() public {
        delete status; // Reset enum to default value;
    }
```  


------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ДОБАВЛЕНИЕ ДАННЫХ В СТРУКТУРУ И МАССИВ:  
```
// Создаем объект структуры.
testStructure qa = testStructure(1, "test");

// Добавим qa в массив structArr.
structArr.push(qa);

// Еще один вариант создания и добавления переменной в массив.
structArr.push(testStructure(1, "tester"));
```  

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ОБЪЯВЛЕНИЕ ФУНКЦИЙ:  
```
function functionName([тип] [arg1], [тип] [arg2], [тип] [тип хранилища] [argN]) {
  // Код функции;
}
```
Например:  
```
function func1 (string memory _name, uint _number) public {
        // Код функции.
    }
```
- public: любой может обратиться к функции и выполнить её код. В Solidity по умолчанию все функции public.
- private: к этой функции есть доступ только внутри контракта, внешний доступ к этой функции недоступен. Имена **private** функций следдует с символа "_".

Если функция что-то возвращает, то мы должны это явно указать, например:  
```
// Функция вернет результат сложения целочисленных переменных.
function _sum(uint a, uint b) private returns (uint) {
  return a + b;
```
```
// Функция вернет строчный тип данных.
function greetings() public returns (string memory) {
  string greet = "Greeting from public function!";
  return greet;
```

**МОДИФИКАТОРЫ**:  
- view: если функция не меняет ничего и не добавляет, то её добавляется модификатор **view**. Может обращаться к state и global переменным.  
- pure: функция работает только с передаваемыми в неё аргументами. Может обращаться только к локальным переменным.  
- public:
- private:  
- internal: функция доступна только внутри контракта и наследуемых контрактов.
- external: функция доступна только для внешних вызовов.  
- custom modifiers.
- payable: тип функции который может получать ETH.
```
contract Payable {
    // Payable address can receive Ether
    address payable public owner;

    // Payable constructor can receive Ether
    constructor() payable {
        owner = payable(msg.sender);
    }

    function deposit() external payable {}
}
```  

Пример **view** функции:  
```
function greeting() public view returns (string memory){
}
```

Пример **pure** функции:  
```
function add(uint a, uint b) public pure returns (uint) {
  return a + b;
}
```
Пример **internal** функции:
```
contract OtherContract {
  function internalFuncName(uint a) internal returns (uint) {
    return a++;
  }
}
```

Пример **external** функции: 

```
contract ContractName is OtherContract { // Наследование от OtherContract.
  fucntion externalFuncName() external returns (uint) { // Внешняя функция.
    return internalFuncName(1); // Вызываем функцию internalFuncName из контракта OtherContract.
  }
}
```  

Пример **custom modifiers**:  

```
modifier myModifier() {
  require(msg.sender == owner);
  _;
}
```

Пример **custom modifiers** с аргументами:  

```
modifier myModifier(uint a, uint b) {
  require(a % b == 0);
  _;
}

function c(uint a, uint b) public myModifier(4,2) returns (uint) {
  retturn a * b;
}
```

```
public bool = true;
public int256 = -123;
public uint256 = 123;
public address addr = 0x...;
public byte32 byteAddr = 0x...;
```  

**Модификаторы видимости**: private, internal, external, public.
**Модификаторы состояния**: view, pure.
**Пользовательские модификаторы**.

Модификаторы могут быть объеденены следующим образом: ```function test() external view onlyOwner anotherModifier { /* ... */ }```  

Если функция не payable, а на нее пытаются отправить ETH, то функция отклонит транзакцию.  

После отправки ETH он сохранится в контракте в аккаунте Ethereum. Он не исчезнет оттуда пока не добавим функцию снятия ETH с адреса контракта.

**Функция снятия ETH с адреса контракта**:  

```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```

**transfer** - переводит средства на любой адрес в Ethereum.
**this.balance** - вернет общий баланс контракта.

Функция может возвращать несколько значенией:  
```
function myFunc() external view returns (address, bool) {
        return (msg.sender, false);
    }
```  

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ПРЕОБРАЗОВАНИЕ ТИПОВ:  

Преобразование целочисленных:  
```
uint8 a = 5;
uint b = 6;

uint8 c = a * b; // Вызовет ошибку, потому что a * b вернет uint, не uint8.

uint8 c = a * uint8(b); // Мы примели b к типу uint8;

uint(keccak256(abi.encodePacked("aaaab"))); // Преобразование значения HEX в целочисленное.
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### KECCAC256:  

В Etherreum есть встроенная хеш функция keccak256, которая является версией SHA3. Хеш функция преобразует входящие данные в рандомное значение в HEX размером 256 бит. Важно отметить, что keccak256 принимает на вход один параметр типа bytes.Это означает, что мы должны "упаковать" параметр до передачи его на вход keccak256.

Пример:
```
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256(abi.encodePacked("aaaab"));
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256(abi.encodePacked("aaaac"));
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### EVENTS (СОБЫТИЯ):  
События это способ коммуникации контракта с front-end приложение о произощедшем в blockchain. Front-end приложение прослушивает соответствующие события и выполняет действия когда они произошли.  
События позволяют смарт-контрактам регистрировать данные в блокчейне без использования переменных состояния.  
Чаще всего Events используются для дебага, мониторинга и как дешевое хранилище для хранения переменных состояния.  

```
event Message(address indexed from, address indexed to, string message);
```

```
event Multiplication(uint a, uint b, uint number); // Создали событие.

function mul(uint _a, uint _b) public returns (uint) {
  uint number = _a * _b;
  emit Multiplication(_a, _b, number ); // Вызываем event;
  return number;
}
```

Код на front-end приложении:  
```
contractName.Multiplication(function(error, result) {
  // Обрабатываем ответ.
})
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### MAPPINGS:  
**Mapping** это key/value хранилище данных.

```
mapping (address => uint) public addressToId;

mapping (uint => string) idToUser;
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ, ВСТРОЕННЫЕ ФУНКЦИИ:  

- ```msg.sender```: адресс пользователя/контракта вызвавшего функцию.
- ```msg.value```: кличество wei/eth.
- ```msg.value``` - способ увидеть, сколько ETH было отправлено на адрес контракта.
- ```ether``` - встроенный блок.
- ```require```: проверяет условие и если условие не выполнено, выполнение кода будет приостановленно и произойдет откат всех изменени1 состояния сделанные до require. 
- ```revert```: похож на require но более удобен когда проверяемое значение вложенно в несколько операторов if.  
- ```assert``` - проверяет условие и выбрасывает ошибку если false. Разница между assert и require - require вернет остаток газа если функция не сработает, assert - не вернет остаток газа. assert используется в случаях если в коде что-то пошло не так (например, переполнение uint).
- ```tx.origin``` - адресс изначального инициатора транзации.  
- ```transfer``` - ф-ция перевода средств, при возникновении ошибки выбрасывается исключение, после чего код не будет выполняться. Нет отката транзакции. ```address payable recipient.transfer(uint256 amount)```  
- ```send``` - ф-ция перевода средств, ошибка передачи не вызывает исключения и возвращает true/false это означает, что программисты должны вручную проверять успешность перевода и принимать соответствующие меры. Код продолжит выполняться. ```bool success = recipient.send(uint256 amount)```
- ```call``` - низкоуровневая функция для взаимодействия с другими контрактами. Также может вызывать функции в дргуих контрактах
- ```call.value().gas()()``` - ф-ция перевода стредств, ошибка передачи не выбрасывает исключение и возвращает true/false. Позволяет контролировать сумму перевода и количество газа. Код будет выполнен, но функции вызова для передачи подвержены атакам реентерабельности. Нет отката транзакции. ```(bool success, ) = recipient.call{value: amount, gas: gasLimit}("");```  
- ```payable(msg.sender)``` - это указывает, что адрес отправителя (msg.sender) будет получателем средств при уничтожении контракта. Адрес msg.sender должен быть преобразован в тип payable, чтобы получить эфир.
- ```selfdestruct()``` - после самоуничтожения контракта все ссылки на него будут указывать на байткод 0x, как если бы это был обычный счет. Однако важно отметить, что поскольку блокчейн неизменяем, все прошлые транзакции и вызовы контрактов все равно будут храниться в истории предыдущих блоков и не могут быть удалены, даже если контракт будет уничтожен. Так что на самом деле код контракта в некотором смысле все еще хранится в предыдущих блоках.
- ```block.timestamp``` - временная метка текущего блока.
- ```block.number``` - текущи   номер блока.
- ```blockhash(block.number)``` - хеш текущего блока. Тип данных byte32. Работате только для последных 256 блоков.  
- ```constructor()``` - функция будет вызвана один раз при развертывании контракта. Основная цель это задать начально значение переменным.
- ```indexed``` - индексированные параметры для зарегистрированных событий позволят вам искать эти события, используя индексированные параметры в качестве фильтра. Используется для логирования.  

#### LOW LEVEL FUNCTIONS:  

```call```: низкоуровневая функция для взаимодействия с другими контрактами. Возвращает логическое значение и результат работы вызванной функции.

Пример вызова функции call:  
```
contract SendEther {
function sendViaCall(address payable _to) public payable {
  // Call returns a boolean value indicating success or failure.
    (bool sent, bytes memory data) = _to.call{value: msg.value}("");
    require(sent, "Failed to send Ether");
   }

function sendViaCall(address payable addr) public {
  // 
    (bool sent, bytes memory data) = addr.call(abi.encodeWithSignature("bar(uint256,bool)", 1, true));
    require(sent, "Failed to send Ether");
   }

}
```
```
contract ReceiveEther {
// Function to receive Ether. msg.data must be empty
  receive() external payable {}‍
  
  // Fallback function is called when msg.data is not empty
  fallback() external payable {}‍
  
  function getBalance() public view returns (uint) {
  return address(this).balance;
   }

  fucntion bar(uint256 a, bool b) external returns(uint256, bool) {
    return (a,b);
  }
}
```  

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ЦИКЛЫ:  

- if, else if, else: условие "если".
```
uint a = 1;
uint b = 2;

function(uint a, uint b) public {
  // Function code;
  if (a == b) {
    // Function code;
  }
  else if (a != b) {
    // Block code;
  }
  else {
    // Block code;
  }
```

Использование тернартного оператора для проверки условия:  
```
function exercise_1(uint256 x) external pure returns (uint256) {
        return x > 0 ? 1 : 0;
    }
```

- for: цыкл "for".
```
uint[] numbers;
function c() public {
  for (uint i = 0; i <= 10; i++) {
    if (i % 2 == 0) {
      numbers.push(i);
    }
  rerutn numbers;
  }
}
```

- while: цыкл "while".
```
uint a = 10;
uint b;
while(b < a) {
  b++;
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### НАСЛЕДОВАНИЕ:  

Контракты могут наследовать контракты. У контрактов, которые наследуют другие контракты есть доступ к публичным функциям наследуемого контракта.

```
contract Animal {

}

contract Dog is Animal{

}
```

Перегрузка функций:  

Функции, которые могут быть перегруженны потомком должны быть помеченны **virtual**.  
Функции, которые перегружают родительские функции должны быть помеченны **override**.  

```
contract A {
    // foo() can be overridden by child contract
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

// Inherit other contracts by using the keyword 'is'.
contract B is A {
    // Перегружаем функцию foo() контракта A.
    function foo() public pure override returns (string memory) {
        return "Inheritance";
    }
}
```

В Solidity можно наследовать множество контрактов:  

```
contract X {
    function foo() public pure virtual returns (string memory) {
        return "X";
    }
}

contract Y is X {
    // Перегружаем функцию foo() контракта X.
    // Перегружаемую ф-цию также помечаем как virtual, для того, чтобы она также могла быть перегруженна потомом.
    function foo() public pure virtual override returns (string memory) {
        return "Y";
    }
}

// Полследовательность наследования - most base-like to derived
contract Z is X, Y {
    // Перегружаем ф-цию foo() из контрактов X и Y.
    function foo() public pure override(X, Y) returns (string memory) {
        return "Z";
    }
}
```  

Вызов родительского конструктора:

```
contract A {
    string public name;

    constructor(string memory _name) {
        name = _name;
    }
}

contract B {
    string public text;

    constructor(string memory _text) {
        text = _text;
    }
}

// 2 варианта вызвать родительский конструктор.
contract C is A("A"), B("B") {}

contract D is A, B {
    // Pass the parameters here in the constructor,
    constructor(string memory _name, string memory _text) A(_name) B(_text) {}
}
```

Вызов родительских функций:  

Вызвать функцию родительского контракта можно напрямую или использовать слово **super**.  

```
contract E {
    function bar() public virtual {
        emit Log("E.bar");
    }
}

contract F is E {
  function bar() public override(F) {
        super.bar();
    }

  E.bar();
}

```


----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### IMPORTS:  

В Ethereum можно импортировать другие контракты.

**import** позволялет подключить в наш контракт другие контракты и библиотеки.

Можно импортировать контракты из других файлов, библиотек, внешних репозиториев (через URL или npm пакеты).  

Важно следить за версиями Solidity в других контрактах, так как если версии Solidity отличаются, код может не скомпилироваться.  

```
import "./SolidityContract.sol"

contract anotherContract {

}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ВЗАИМОДЕЙСТВИЕ С ДРУГИМИ КОНТРАКТАМИ:  

Для того, чтобы наш контракт мог общаться с другим контрактом (владельцем которошго являемся не мы), сперва необходимо определить interface. **Interfeces**: Interfaces enable a contract to call other contracts without having its code.

```
interface TestInterface {
  function foo() external;
  function bar() external returns (uint256);
}

contract TestContract {
  function example(address addr) external {
    TestInterface(addr).foo();
  }
}
```  

```
contract Random {
  function getNumber(address _myAddress) public view returns (uint); // Код функции не определяется и определение функции завершается точкой с запятой, так компилятор узнает, что эьто интерфейс.
}

contract GetNumber {
  address randomAddress = [АДРЕС КОНТРАКТА Random];
  Random randomContract = Random(randomAddress);

  function result() public {
    uint number = randomContract.getNumber(msg.sender);
  }
}
```
Таким образом контракт будет взаимодействовать с другим контрактом в блокчейн, если они задают функции как public или external.  



----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### TOKEN ON ETHEREUM:  

Токен - в целом это контракт который следует определенным правилам и реализуем стандартный набор функций которые есть у всех других токен-контрактах: 
- transferFrom(address _from, address _to, uint256 _amount).
- balanceOf(address _owner).

Внутри смарт-контракта есть mapping (key = value), mapping(address => uint256) balances , который отслеживает состояние баланса каждого адресса.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### БИБЛИОТЕКИ:  

- OpenZeppelin: SafeMath.


----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### ВЛОЖЕННЫЕ КОНТРАКТЫ:  

Создание контрактов внутри контрактов в блокчейне, или "вложенные контракты", может быть полезным для различных целей, включая:  

- Модульность и переиспользуемость.
- Повышение безопасности.
- Гибкость и расширяемость.
- Управление состоянием.
- Сложные схемы взаимодействия.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MainContract {
    // Вложенный контракт для управления правами доступа
    contract AccessControl {
        address public owner;
        
        constructor() {
            owner = msg.sender;
        }

        modifier onlyOwner() {
            require(msg.sender == owner, "Not the owner");
            _;
        }
    }
    
    // Вложенный контракт для управления токенами
    contract TokenManager {
        mapping(address => uint256) public balances;

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw(uint256 amount) public {
            require(balances[msg.sender] >= amount, "Insufficient balance");
            balances[msg.sender] -= amount;
            payable(msg.sender).transfer(amount);
        }
    }
    
    // Используем вложенные контракты
    AccessControl private accessControl;
    TokenManager private tokenManager;

    constructor() {
        accessControl = new AccessControl();
        tokenManager = new TokenManager();
    }

    // Функция для депозита в токенах
    function depositTokens() public payable {
        tokenManager.deposit{value: msg.value}();
    }

    // Функция для вывода токенов
    function withdrawTokens(uint256 amount) public {
        tokenManager.withdraw(amount);
    }

    // Функция для обновления владельца (только для владельца)
    function changeOwner(address newOwner) public {
        accessControl.onlyOwner();
        accessControl.owner = newOwner;
    }

    // Получить баланс пользователя
    function getBalance() public view returns (uint256) {
        return tokenManager.balances(msg.sender);
    }
}

```  

#### FALLBACK(), RECEIVE():  
```
Which function is called, fallback() or receive()?

    Ether is sent to contract
               |
        is msg.data empty?
              / \
            yes  no
            /     \
receive() exists?  fallback()
         /   \
        yes   no
        /      \
    receive()   fallback()
```

В Solidity функция fallback() используется для обработки вызовов, которые не совпадают с сигнатурой существующих функций контракта, а также для получения эфира на контракт. Она выполняется в следующих случаях:

1. При вызове несуществующей функции: Если вызывается функция, которая не была определена в контракте, то выполняется fallback(). Например, если кто-то пытается вызвать функцию с ошибочной сигнатурой, эта функция будет срабатывать.
2. Для получения эфира: Если контракт получает эфир, но не указывает конкретной функции для обработки этого перевода, то вызывается fallback(). Эта функция может быть использована для выполнения логики, связанной с принятием средств.
3. Когда контракт получает данные: Если данные (например, при вызове с помощью call или delegatecall) не соответствуют никакой функции контракта, также будет вызвана fallback().

Enable contract to receive Ether:  
```
receive() external payable {}
```

Send Ether:  
```
function sendViaTransfer(address payable to) external payable {
        // This function is no longer recommended for sending Ether.
        to.transfer(msg.value);
    }

    function sendViaSend(address payable to) external payable {
        // Send returns a boolean value indicating success or failure.
        // This function is not recommended for sending Ether.
        bool sent = to.send(msg.value);
        require(sent, "Failed to send Ether");
    }

    function sendViaCall(address payable to) external payable {
        // Call returns a boolean value indicating success or failure.
        // This is the current recommended method to use.
        (bool sent, bytes memory data) = to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
    
    function sendEth(address payable to, uint256 amount) external payable {
        (bool sent, ) = to.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
```






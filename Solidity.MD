## Solidity - это язык программирования для написания смарт-контрактов на блокчейне Ethereum (и EVM-совместимых сетях).

Смарт-контракт - это программа, которая:

* Выполняется на блокчейне
* Не может быть изменена после деплоя
* Выполняется автоматически при определенных условиях
* Хранит данные в блокчейне

### Первый контракт

````solidity
// SPDX-License-Identifier: MIT
// Лицензия - обязательная строка

// Версия компилятора Solidity
pragma solidity ^0.8.0;

// Объявление контракта (как класс в других языках)
contract HelloWorld {
    
    // Переменная состояния (хранится в блокчейне)
    string public message;
    
    // Конструктор - выполняется один раз при деплое
    constructor() {
        message = "Hello, Blockchain!";
    }
    
    // Функция для чтения (не изменяет состояние)
    function getMessage() public view returns (string memory) {
        return message;
    }
    
    // Функция для записи (изменяет состояние, требует газ)
    function setMessage(string memory newMessage) public {
        message = newMessage;
    }
}

/*
ОСНОВНЫЕ КОНЦЕПЦИИ:

1. SPDX-License-Identifier - указывает лицензию кода
2. pragma - версия компилятора
3. contract - основная структура (аналог класса)
4. public - видимость переменной/функции
5. view - функция только читает данные
6. memory - временное хранение данных
7. constructor - инициализация при деплое
*/
````
### Типы данных
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DataTypes {
    
    // ========== ЧИСЛОВЫЕ ТИПЫ ==========
    
    // Целые числа без знака (только положительные)
    uint8 public smallNumber = 255;        // 0 до 255 (1 байт)
    uint256 public largeNumber = 1000000;  // самый популярный (32 байта)
    uint public defaultNumber = 42;        // uint = uint256
    
    // Целые числа со знаком (+ и -)
    int8 public smallInt = -128;           // -128 до 127
    int256 public largeInt = -1000000;
    int public defaultInt = -42;           // int = int256
    
    // ========== АДРЕСА ==========
    
    address public myAddress = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;
    address payable public payableAddress; // может принимать ETH
    
    // ========== БУЛЕВЫЕ ==========
    
    bool public isActive = true;
    bool public isComplete = false;
    
    // ========== СТРОКИ И БАЙТЫ ==========
    
    string public name = "Alice";
    bytes32 public data = "Hello";         // фиксированный размер (дешевле)
    bytes public dynamicData = "World";    // динамический размер
    
    // ========== МАССИВЫ ==========
    
    // Фиксированный массив
    uint[5] public fixedArray = [1, 2, 3, 4, 5];
    
    // Динамический массив
    uint[] public dynamicArray;
    address[] public users;
    
    // ========== MAPPING (словарь/хеш-таблица) ==========
    
    mapping(address => uint) public balances;
    mapping(address => bool) public whitelist;
    mapping(uint => string) public idToName;
    
    // Вложенный mapping
    mapping(address => mapping(address => uint)) public allowances;
    
    // ========== СТРУКТУРЫ ==========
    
    struct User {
        string name;
        uint age;
        bool isActive;
        address wallet;
    }
    
    User public admin;
    User[] public allUsers;
    mapping(address => User) public userRegistry;
    
    // ========== ENUM ==========
    
    enum Status {
        Pending,    // 0
        Active,     // 1
        Completed,  // 2
        Cancelled   // 3
    }
    
    Status public currentStatus = Status.Pending;
    
    // ========== ФУНКЦИИ ДЛЯ РАБОТЫ С ТИПАМИ ==========
    
    constructor() {
        // Инициализация payable адреса
        payableAddress = payable(msg.sender);
        
        // Инициализация структуры
        admin = User({
            name: "Admin",
            age: 30,
            isActive: true,
            wallet: msg.sender
        });
    }
    
    // Работа с массивами
    function addToArray(uint value) public {
        dynamicArray.push(value);
    }
    
    function getArrayLength() public view returns (uint) {
        return dynamicArray.length;
    }
    
    function removeLastElement() public {
        dynamicArray.pop();
    }
    
    function getElement(uint index) public view returns (uint) {
        require(index < dynamicArray.length, "Index out of bounds");
        return dynamicArray[index];
    }
    
    // Работа с mapping
    function setBalance(address user, uint amount) public {
        balances[user] = amount;
    }
    
    function getBalance(address user) public view returns (uint) {
        return balances[user]; // если ключа нет, вернет 0
    }
    
    // Работа со структурами
    function createUser(string memory _name, uint _age) public {
        User memory newUser = User({
            name: _name,
            age: _age,
            isActive: true,
            wallet: msg.sender
        });
        
        allUsers.push(newUser);
        userRegistry[msg.sender] = newUser;
    }
    
    function getUserInfo(address userAddress) public view returns (
        string memory,
        uint,
        bool,
        address
    ) {
        User memory user = userRegistry[userAddress];
        return (user.name, user.age, user.isActive, user.wallet);
    }
    
    // Работа с enum
    function setStatus(Status newStatus) public {
        currentStatus = newStatus;
    }
    
    function getStatus() public view returns (Status) {
        return currentStatus;
    }
    
    function isPending() public view returns (bool) {
        return currentStatus == Status.Pending;
    }
    
    // Преобразования типов
    function convertTypes() public pure returns (
        uint,
        address,
        bytes32
    ) {
        // uint в address
        address addr = address(uint160(123456));
        
        // address в uint
        uint addrAsUint = uint160(addr);
        
        // string в bytes32
        bytes32 b = bytes32(bytes("Hello"));
        
        return (addrAsUint, addr, b);
    }
}

/*
ВАЖНЫЕ МОМЕНТЫ:

1. uint = uint256 (всегда используйте uint256 для ясности)
2. mapping не имеет length и не итерируется
3. string хранится как bytes, дороже в газе
4. memory - временные данные в функции
5. storage - постоянные данные в блокчейне
6. Массивы можно итерировать, но это дорого по газу
7. struct можно использовать в mapping и массивах
8. enum экономит газ по сравнению с uint
*/
````
### Функции и модификаторы видимости
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Functions {
    
    uint public counter;
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    // ========== МОДИФИКАТОРЫ ВИДИМОСТИ ==========
    
    // PUBLIC - доступна всем (извне и внутри)
    function publicFunction() public pure returns (string memory) {
        return "Anyone can call this";
    }
    
    // EXTERNAL - только извне контракта (дешевле чем public)
    function externalFunction() external pure returns (string memory) {
        return "Only external calls";
    }
    
    // INTERNAL - только внутри контракта и наследников
    function internalFunction() internal pure returns (string memory) {
        return "Only inside contract";
    }
    
    // PRIVATE - только внутри текущего контракта
    function privateFunction() private pure returns (string memory) {
        return "Only in this contract";
    }
    
    // Функция для тестирования internal/private
    function callInternalFunctions() public pure returns (string memory, string memory) {
        return (internalFunction(), privateFunction());
    }
    
    // ========== МОДИФИКАТОРЫ СОСТОЯНИЯ ==========
    
    // VIEW - читает состояние, но не изменяет (бесплатно вне транзакций)
    function getCounter() public view returns (uint) {
        return counter;
    }
    
    // PURE - не читает и не изменяет состояние (чистая функция)
    function calculate(uint a, uint b) public pure returns (uint) {
        return a + b;
    }
    
    // БЕЗ МОДИФИКАТОРА - изменяет состояние (требует газ)
    function increment() public {
        counter += 1;
    }
    
    // ========== МОДИФИКАТОРЫ (MODIFIERS) ==========
    
    // Кастомный модификатор для проверки владельца
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _; // Место, где выполнится код функции
    }
    
    // Модификатор с параметром
    modifier minValue(uint amount) {
        require(amount >= 100, "Value too small");
        _;
    }
    
    // Модификатор для проверки до и после
    modifier checkInvariants() {
        uint oldCounter = counter;
        _;
        require(counter >= oldCounter, "Counter decreased!");
    }
    
    // Использование модификаторов
    function resetCounter() public onlyOwner {
        counter = 0;
    }
    
    function addValue(uint value) public minValue(value) {
        counter += value;
    }
    
    function safeIncrement() public checkInvariants {
        counter += 1;
    }
    
    // Несколько модификаторов
    function ownerAddValue(uint value) public onlyOwner minValue(value) {
        counter += value;
    }
    
    // ========== ПАРАМЕТРЫ ФУНКЦИЙ ==========
    
    // Простые параметры
    function add(uint a, uint b) public pure returns (uint) {
        return a + b;
    }
    
    // Memory vs Calldata vs Storage
    function processString(string memory str) public pure returns (uint) {
        return bytes(str).length;
    }
    
    function processStringCalldata(string calldata str) external pure returns (uint) {
        // calldata дешевле, но только для external функций
        return bytes(str).length;
    }
    
    uint[] private myArray;
    
    function processArray(uint[] memory arr) public pure returns (uint) {
        // memory - копия массива
        return arr.length;
    }
    
    function modifyStorageArray() public {
        // storage - ссылка на оригинал
        uint[] storage arr = myArray;
        arr.push(1);
    }
    
    // ========== ВОЗВРАЩАЕМЫЕ ЗНАЧЕНИЯ ==========
    
    // Один результат
    function getOne() public pure returns (uint) {
        return 1;
    }
    
    // Несколько результатов
    function getMultiple() public pure returns (uint, bool, string memory) {
        return (42, true, "Hello");
    }
    
    // Именованные возвращаемые значения
    function getNamedReturns() public pure returns (
        uint number,
        bool flag,
        string memory text
    ) {
        number = 42;
        flag = true;
        text = "Hello";
        // return не нужен, если присвоили значения
    }
    
    // Деструктуризация при вызове
    function useMultipleReturns() public pure returns (uint) {
        (uint num, , string memory txt) = getMultiple();
        // Пропускаем второе значение
        return num;
    }
    
    // ========== СПЕЦИАЛЬНЫЕ ФУНКЦИИ ==========
    
    // PAYABLE - может принимать ETH
    function deposit() public payable {
        // msg.value - количество отправленного ETH
        require(msg.value > 0, "Send some ETH");
    }
    
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
    
    // FALLBACK - вызывается при неизвестной функции
    fallback() external payable {
        counter += 1;
    }
    
    // RECEIVE - вызывается при получении ETH без данных
    receive() external payable {
        // Получили ETH
    }
    
    // ========== ВЫЗОВЫ ДРУГИХ ФУНКЦИЙ ==========
    
    function callAnotherFunction() public view returns (uint) {
        // Внутренний вызов
        return getCounter();
    }
    
    // Вызов функции другого контракта
    function callExternalContract(address contractAddr) public view returns (uint) {
        Functions other = Functions(contractAddr);
        return other.getCounter();
    }
    
    // ========== ПЕРЕГРУЗКА ФУНКЦИЙ ==========
    
    function setValue(uint value) public {
        counter = value;
    }
    
    function setValue(uint value, bool increment) public {
        if (increment) {
            counter = value + 1;
        } else {
            counter = value;
        }
    }
    
    // ========== СОБЫТИЯ (EVENTS) ==========
    
    event CounterChanged(uint oldValue, uint newValue, address changer);
    event ValueAdded(address indexed user, uint indexed amount, uint timestamp);
    
    function incrementWithEvent() public {
        uint oldValue = counter;
        counter += 1;
        emit CounterChanged(oldValue, counter, msg.sender);
    }
    
    function addWithEvent(uint amount) public {
        counter += amount;
        emit ValueAdded(msg.sender, amount, block.timestamp);
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ:

ВИДИМОСТЬ:
- public: доступна всем
- external: только извне (экономит газ)
- internal: внутри контракта и наследников
- private: только в текущем контракте

МОДИФИКАТОРЫ СОСТОЯНИЯ:
- view: читает, но не изменяет
- pure: не читает и не изменяет
- payable: может принимать ETH
- без модификатора: изменяет состояние

МОДИФИКАТОРЫ (modifier):
- Переиспользуемые проверки
- Выполняются перед функцией
- _ указывает место выполнения функции

ХРАНЕНИЕ ДАННЫХ:
- storage: постоянное (блокчейн)
- memory: временное (функция)
- calldata: только чтение (параметры external)

СПЕЦИАЛЬНЫЕ ФУНКЦИИ:
- constructor: один раз при деплое
- fallback: неизвестная функция
- receive: получение ETH

СОБЫТИЯ:
- Логирование для фронтенда
- indexed - можно фильтровать (макс 3)
- Дешевле чем хранение в storage
*/
````
### Наследование и интерфейсы
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== БАЗОВЫЙ КОНТРАКТ ==========

contract Ownable {
    address public owner;
    
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    
    constructor() {
        owner = msg.sender;
        emit OwnershipTransferred(address(0), msg.sender);
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "Invalid address");
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }
}

// ========== ПРОСТОЕ НАСЛЕДОВАНИЕ ==========

contract MyContract is Ownable {
    uint public value;
    
    // Наследуем owner, onlyOwner, transferOwnership
    
    function setValue(uint newValue) public onlyOwner {
        value = newValue;
    }
}

// ========== МНОЖЕСТВЕННОЕ НАСЛЕДОВАНИЕ ==========

contract Pausable {
    bool public paused;
    
    event Paused(address account);
    event Unpaused(address account);
    
    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }
    
    modifier whenPaused() {
        require(paused, "Contract is not paused");
        _;
    }
    
    function _pause() internal {
        paused = true;
        emit Paused(msg.sender);
    }
    
    function _unpause() internal {
        paused = false;
        emit Unpaused(msg.sender);
    }
}

// Наследование от нескольких контрактов
contract AdvancedContract is Ownable, Pausable {
    uint public counter;
    
    function increment() public onlyOwner whenNotPaused {
        counter += 1;
    }
    
    function pause() public onlyOwner {
        _pause();
    }
    
    function unpause() public onlyOwner {
        _unpause();
    }
}

// ========== ПЕРЕОПРЕДЕЛЕНИЕ ФУНКЦИЙ ==========

contract Base {
    uint public x = 10;
    
    // virtual - функция может быть переопределена
    function getValue() public view virtual returns (uint) {
        return x;
    }
}

contract Derived is Base {
    // override - переопределяем функцию
    function getValue() public view override returns (uint) {
        return x * 2;
    }
}

// ========== МНОЖЕСТВЕННОЕ НАСЛЕДОВАНИЕ С ПЕРЕОПРЕДЕЛЕНИЕМ ==========

contract A {
    function foo() public pure virtual returns (string memory) {
        return "A";
    }
}

contract B {
    function foo() public pure virtual returns (string memory) {
        return "B";
    }
}

// Порядок важен! Самый правый родитель имеет приоритет
contract C is A, B {
    // Должны указать оба контракта в override
    function foo() public pure override(A, B) returns (string memory) {
        return "C";
    }
}

// ========== ВЫЗОВ РОДИТЕЛЬСКИХ ФУНКЦИЙ ==========

contract Parent {
    event Log(string message);
    
    function foo() public virtual {
        emit Log("Parent.foo called");
    }
    
    function bar() public virtual {
        emit Log("Parent.bar called");
    }
}

contract Child is Parent {
    function foo() public override {
        emit Log("Child.foo called");
        
        // Вызов функции родителя
        Parent.foo();
    }
    
    function bar() public override {
        emit Log("Child.bar called");
        
        // Вызов через super (рекомендуется)
        super.bar();
    }
}

// ========== АБСТРАКТНЫЕ КОНТРАКТЫ ==========

abstract contract Animal {
    // Функция без реализации
    function makeSound() public virtual returns (string memory);
    
    // Функция с реализацией
    function sleep() public pure returns (string memory) {
        return "Zzz...";
    }
}

contract Dog is Animal {
    // ОБЯЗАТЕЛЬНО реализовать все виртуальные функции
    function makeSound() public pure override returns (string memory) {
        return "Woof!";
    }
}

contract Cat is Animal {
    function makeSound() public pure override returns (string memory) {
        return "Meow!";
    }
}

// ========== ИНТЕРФЕЙСЫ ==========

// Interface - контракт только с объявлениями функций
interface IERC20 {
    function totalSupply() external view returns (uint);
    function balanceOf(address account) external view returns (uint);
    function transfer(address recipient, uint amount) external returns (bool);
    
    event Transfer(address indexed from, address indexed to, uint value);
}

// Реализация интерфейса
contract MyToken is IERC20 {
    mapping(address => uint) private balances;
    uint private _totalSupply;
    
    constructor(uint initialSupply) {
        _totalSupply = initialSupply;
        balances[msg.sender] = initialSupply;
    }
    
    function totalSupply() external view override returns (uint) {
        return _totalSupply;
    }
    
    function balanceOf(address account) external view override returns (uint) {
        return balances[account];
    }
    
    function transfer(address recipient, uint amount) external override returns (bool) {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[recipient] += amount;
        
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }
}

// ========== ИСПОЛЬЗОВАНИЕ ИНТЕРФЕЙСОВ ==========

contract TokenUser {
    // Взаимодействие с любым ERC20 токеном
    function transferTokens(
        address tokenAddress,
        address recipient,
        uint amount
    ) public returns (bool) {
        IERC20 token = IERC20(tokenAddress);
        return token.transfer(recipient, amount);
    }
    
    function checkBalance(address tokenAddress, address account) public view returns (uint) {
        IERC20 token = IERC20(tokenAddress);
        return token.balanceOf(account);
    }
}

// ========== БИБЛИОТЕКИ (LIBRARY) ==========

library Math {
    function max(uint a, uint b) internal pure returns (uint) {
        return a >= b ? a : b;
    }
    
    function min(uint a, uint b) internal pure returns (uint) {
        return a < b ? a : b;
    }
}

library SafeMath {
    function add(uint a, uint b) internal pure returns (uint) {
        uint c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
    
    function sub(uint a, uint b) internal pure returns (uint) {
        require(b <= a, "SafeMath: subtraction overflow");
        return a - b;
    }
}

contract Calculator {
    using SafeMath for uint; // Подключаем библиотеку
    using Math for uint;
    
    function calculate(uint a, uint b) public pure returns (uint, uint, uint) {
        uint sum = a.add(b);       // Используем через .
        uint diff = a.sub(b);
        uint maximum = a.max(b);
        
        return (sum, diff, maximum);
    }
}

// ========== ПРАКТИЧЕСКИЙ ПРИМЕР ==========

// Базовая функциональность токена
abstract contract BaseToken {
    string public name;
    string public symbol;
    uint8 public decimals;
    
    mapping(address => uint) internal balances;
    uint internal _totalSupply;
    
    constructor(string memory _name, string memory _symbol, uint8 _decimals) {
        name = _name;
        symbol = _symbol;
        decimals = _decimals;
    }
    
    function totalSupply() public view returns (uint) {
        return _totalSupply;
    }
    
    function balanceOf(address account) public view returns (uint) {
        return balances[account];
    }
}

// Добавляем возможность паузы
contract PausableToken is BaseToken, Pausable, Ownable {
    constructor(
        string memory _name,
        string memory _symbol,
        uint initialSupply
    ) BaseToken(_name, _symbol, 18) {
        _totalSupply = initialSupply;
        balances[msg.sender] = initialSupply;
    }
    
    function transfer(address to, uint amount) public whenNotPaused returns (bool) {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[to] += amount;
        
        return true;
    }
    
    function pause() public onlyOwner {
        _pause();
    }
    
    function unpause() public onlyOwner {
        _unpause();
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ:

НАСЛЕДОВАНИЕ:
- is - ключевое слово наследования
- Порядок важен при множественном наследовании
- Самый правый контракт имеет приоритет

ПЕРЕОПРЕДЕЛЕНИЕ:
- virtual - функция может быть переопределена
- override - переопределяем функцию
- super - вызов родительской функции

АБСТРАКТНЫЕ КОНТРАКТЫ:
- abstract - контракт с нереализованными функциями
- Нельзя деплоить напрямую
- Должны быть реализованы в наследниках

ИНТЕРФЕЙСЫ:
- Только объявления функций
- Все функции external
- Не может иметь конструктор
- Определяет контракт для взаимодействия

БИБЛИОТЕКИ:
- library - переиспользуемый код
- using для подключения
- Обычно internal или private функции
- Не может иметь состояние

МОДИФИКАТОРЫ:
- Наследуются вместе с контрактом
- Можно переопределить
- Комбинируются при множественном наследовании
*/
````

### Работа с Ether и безопасность
````solidity
/// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== РАБОТА С ETHER ==========

contract EtherWallet {
    address public owner;
    
    event Received(address indexed from, uint amount);
    event Withdrawn(address indexed to, uint amount);
    
    constructor() {
        owner = msg.sender;
    }
    
    // Получение ETH
    receive() external payable {
        emit Received(msg.sender, msg.value);
    }
    
    fallback() external payable {
        emit Received(msg.sender, msg.value);
    }
    
    // Проверка баланса контракта
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
    
    // ========== ТРИ СПОСОБА ОТПРАВКИ ETHER ==========
    
    // 1. TRANSFER (2300 газа, откатывается при ошибке)
    function withdrawTransfer(address payable recipient, uint amount) public {
        require(msg.sender == owner, "Not owner");
        require(address(this).balance >= amount, "Insufficient balance");
        
        // Автоматический revert при ошибке
        recipient.transfer(amount);
        emit Withdrawn(recipient, amount);
    }
    
    // 2. SEND (2300 газа, возвращает bool)
    function withdrawSend(address payable recipient, uint amount) public returns (bool) {
        require(msg.sender == owner, "Not owner");
        require(address(this).balance >= amount, "Insufficient balance");
        
        // Возвращает false при ошибке
        bool success = recipient.send(amount);
        if (success) {
            emit Withdrawn(recipient, amount);
        }
        return success;
    }
    
    // 3. CALL (рекомендуется, передает весь газ)
    function withdrawCall(address payable recipient, uint amount) public {
        require(msg.sender == owner, "Not owner");
        require(address(this).balance >= amount, "Insufficient balance");
        
        // Самый безопасный и гибкий способ
        (bool success, ) = recipient.call{value: amount}("");
        require(success, "Transfer failed");
        emit Withdrawn(recipient, amount);
    }
    
    // Массовая отправка
    function batchTransfer(address payable[] memory recipients, uint[] memory amounts) public {
        require(msg.sender == owner, "Not owner");
        require(recipients.length == amounts.length, "Arrays length mismatch");
        
        for (uint i = 0; i < recipients.length; i++) {
            (bool success, ) = recipients[i].call{value: amounts[i]}("");
            require(success, "Transfer failed");
        }
    }
}

// ========== ЗАЩИТА ОТ REENTRANCY ==========

contract VulnerableBank {
    mapping(address => uint) public balances;
    
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }
    
    // УЯЗВИМАЯ ФУНКЦИЯ!
    function withdrawVulnerable() public {
        uint amount = balances[msg.sender];
        
        // ОПАСНО: внешний вызов перед обновлением состояния
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success);
        
        balances[msg.sender] = 0;
    }
}

contract SecureBank {
    mapping(address => uint) public balances;
    bool private locked;
    
    // Модификатор против reentrancy
    modifier noReentrancy() {
        require(!locked, "No reentrancy");
        locked = true;
        _;
        locked = false;
    }
    
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }
    
    // БЕЗОПАСНАЯ ВЕРСИЯ
    function withdraw() public noReentrancy {
        uint amount = balances[msg.sender];
        require(amount > 0, "Insufficient balance");
        
        // 1. Проверки
        // 2. Изменение состояния (СНАЧАЛА!)
        balances[msg.sender] = 0;
        
        // 3. Внешние взаимодействия (В КОНЦЕ!)
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// ========== CHECKS-EFFECTS-INTERACTIONS ПАТТЕРН ==========

contract CEIPattern {
    mapping(address => uint) public balances;
    
    function withdraw(uint amount) public {
        // 1. CHECKS - все проверки
        require(balances[msg.sender] >= amount, "Insufficient balance");
        require(amount > 0, "Amount must be positive");
        
        // 2. EFFECTS - изменение состояния
        balances[msg.sender] -= amount;
        
        // 3. INTERACTIONS - внешние вызовы
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// ========== PULL OVER PUSH ПАТТЕРН ==========

contract PullPayment {
    mapping(address => uint) public pendingWithdrawals;
    
    // Вместо отправки напрямую - начисляем для вывода
    function addPayment(address recipient, uint amount) internal {
        pendingWithdrawals[recipient] += amount;
    }
    
    // Пользователь сам забирает свои средства
    function withdraw() public {
        uint amount = pendingWithdrawals[msg.sender];
        require(amount > 0, "No pending withdrawals");
        
        pendingWithdrawals[msg.sender] = 0;
        
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// ========== ЗАЩИТА ОТ INTEGER OVERFLOW (до 0.8.0) ==========

contract SafeMathExample {
    // В Solidity >= 0.8.0 overflow проверяется автоматически!
    
    function safeAdd(uint a, uint b) public pure returns (uint) {
        uint c = a + b;
        // После 0.8.0 это происходит автоматически
        require(c >= a, "Addition overflow");
        return c;
    }
    
    function safeSub(uint a, uint b) public pure returns (uint) {
        require(b <= a, "Subtraction overflow");
        return a - b;
    }
    
    function safeMul(uint a, uint b) public pure returns (uint) {
        if (a == 0) return 0;
        uint c = a * b;
        require(c / a == b, "Multiplication overflow");
        return c;
    }
    
    // Если нужно НЕ проверять overflow
    function uncheckedAdd(uint a, uint b) public pure returns (uint) {
        unchecked {
            return a + b; // Может переполниться
        }
    }
}

// ========== ACCESS CONTROL ==========

contract AccessControl {
    address public owner;
    mapping(address => bool) public admins;
    mapping(address => bool) public moderators;
    
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    event AdminAdded(address indexed admin);
    event AdminRemoved(address indexed admin);
    
    constructor() {
        owner = msg.sender;
        admins[msg.sender] = true;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    modifier onlyAdmin() {
        require(admins[msg.sender], "Not an admin");
        _;
    }
    
    modifier onlyModerator() {
        require(moderators[msg.sender], "Not a moderator");
        _;
    }
    
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0), "Invalid address");
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }
    
    function addAdmin(address admin) public onlyOwner {
        admins[admin] = true;
        emit AdminAdded(admin);
    }
    
    function removeAdmin(address admin) public onlyOwner {
        admins[admin] = false;
        emit AdminRemoved(admin);
    }
    
    function addModerator(address moderator) public onlyAdmin {
        moderators[moderator] = true;
    }
}

// ========== ЗАЩИТА ОТ DOS ==========

contract SecureAuction {
    address public highestBidder;
    uint public highestBid;
    
    // НЕ БЕЗОПАСНО - отправка напрямую
    function bidUnsafe() public payable {
        require(msg.value > highestBid, "Bid too low");
        
        // ПРОБЛЕМА: если transfer не проходит, вся функция откатывается
        payable(highestBidder).transfer(highestBid);
        
        highestBidder = msg.sender;
        highestBid = msg.value;
    }
    
    // БЕЗОПАСНО - pull payment
    mapping(address => uint) public pendingReturns;
    
    function bidSafe() public payable {
        require(msg.value > highestBid, "Bid too low");
        
        if (highestBidder != address(0)) {
            pendingReturns[highestBidder] += highestBid;
        }
        
        highestBidder = msg.sender;
        highestBid = msg.value;
    }
    
    function withdrawBid() public {
        uint amount = pendingReturns[msg.sender];
        require(amount > 0, "Nothing to withdraw");
        
        pendingReturns[msg.sender] = 0;
        
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Withdrawal failed");
    }
}

// ========== ЗАЩИТА ОТ FRONT-RUNNING ==========

contract CommitReveal {
    mapping(address => bytes32) public commits;
    mapping(address => uint) public reveals;
    
    // Фаза 1: Коммит (скрытое значение)
    function commit(bytes32 hash) public {
        commits[msg.sender] = hash;
    }
    
    // Фаза 2: Раскрытие
    function reveal(uint value, bytes32 secret) public {
        bytes32 hash = keccak256(abi.encodePacked(value, secret));
        require(hash == commits[msg.sender], "Invalid reveal");
        
        reveals[msg.sender] = value;
    }
}

// ========== TIME LOCKS ==========

contract TimeLock {
    mapping(address => uint) public lockedUntil;
    mapping(address => uint) public balances;
    
    function deposit(uint lockDuration) public payable {
        balances[msg.sender] += msg.value;
        lockedUntil[msg.sender] = block.timestamp + lockDuration;
    }
    
    function withdraw() public {
        require(block.timestamp >= lockedUntil[msg.sender], "Still locked");
        require(balances[msg.sender] > 0, "No balance");
        
        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;
        
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

// ========== ПРАКТИЧЕСКИЕ СОВЕТЫ ПО БЕЗОПАСНОСТИ ==========

contract SecurityBestPractices {
    /*
    1. ВСЕГДА используйте Checks-Effects-Interactions паттерн
    2. Используйте ReentrancyGuard для критичных функций
    3. Prefer pull over push для платежей
    4. Используйте call вместо transfer/send
    5. Всегда проверяйте результат call
    6. Ограничивайте доступ (onlyOwner, onlyAdmin)
    7. Используйте события для логирования
    8. Не полагайтесь на block.timestamp для критичной логики
    9. Тестируйте все edge cases
    10. Проводите аудит перед продакшеном
    */
    
    bool private locked;
    
    modifier nonReentrant() {
        require(!locked, "Reentrant call");
        locked = true;
        _;
        locked = false;
    }
    
    // Безопасная функция с всеми проверками
    function secureFunction(address recipient, uint amount) 
        public 
        nonReentrant 
    {
        // 1. Валидация входных данных
        require(recipient != address(0), "Invalid recipient");
        require(amount > 0, "Amount must be positive");
        require(amount <= address(this).balance, "Insufficient balance");
        
        // 2. Изменение состояния
        // (ваша логика здесь)
        
        // 3. Внешние взаимодействия
        (bool success, ) = recipient.call{value: amount}("");
        require(success, "Transfer failed");
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ БЕЗОПАСНОСТИ:

СПОСОБЫ ОТПРАВКИ ETH:
- transfer: 2300 газа, revert при ошибке (устарел)
- send: 2300 газа, возвращает bool (устарел)
- call: передает весь газ, возвращает (bool, bytes) (РЕКОМЕНДУЕТСЯ)

REENTRANCY:
- Изменяйте состояние ДО внешних вызовов
- Используйте nonReentrant modifier
- Следуйте CEI паттерну

CHECKS-EFFECTS-INTERACTIONS:
1. Checks - все проверки
2. Effects - изменение состояния
3. Interactions - внешние вызовы

ЗАЩИТА ОТ DOS:
- Pull over Push паттерн
- Не отправляйте ETH в циклах
- Ограничивайте размер массивов

ACCESS CONTROL:
- onlyOwner для критичных функций
- Роли (admin, moderator, user)
- Проверка адресов

OVERFLOW:
- Solidity >= 0.8.0 защищает автоматически
- unchecked для отключения проверок

FRONT-RUNNING:
- Commit-Reveal схема
- Приватные транзакции (flashbots)
- Батчинг транзакций
*/
````
### ERC-20 токен (стандарт)
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ERC-20 ИНТЕРФЕЙС ==========

interface IERC20 {
    // Возвращает общее количество токенов
    function totalSupply() external view returns (uint256);
    
    // Возвращает баланс аккаунта
    function balanceOf(address account) external view returns (uint256);
    
    // Переводит токены получателю
    function transfer(address recipient, uint256 amount) external returns (bool);
    
    // Возвращает количество токенов, которые spender может потратить от owner
    function allowance(address owner, address spender) external view returns (uint256);
    
    // Устанавливает amount как allowance для spender
    function approve(address spender, uint256 amount) external returns (bool);
    
    // Переводит amount от sender к recipient используя allowance
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    
    // События
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

// ========== БАЗОВАЯ РЕАЛИЗАЦИЯ ERC-20 ==========

contract BasicERC20 is IERC20 {
    // Данные токена
    string public name;
    string public symbol;
    uint8 public decimals;
    
    // Состояние
    uint256 private _totalSupply;
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    
    constructor(
        string memory _name,
        string memory _symbol,
        uint8 _decimals,
        uint256 initialSupply
    ) {
        name = _name;
        symbol = _symbol;
        decimals = _decimals;
        
        _totalSupply = initialSupply;
        _balances[msg.sender] = initialSupply;
        
        emit Transfer(address(0), msg.sender, initialSupply);
    }
    
    // Общее количество токенов
    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }
    
    // Баланс аккаунта
    function balanceOf(address account) public view override returns (uint256) {
        return _balances[account];
    }
    
    // Перевод токенов
    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(msg.sender, recipient, amount);
        return true;
    }
    
    // Проверка allowance
    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }
    
    // Установка allowance
    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(msg.sender, spender, amount);
        return true;
    }
    
    // Перевод от имени другого аккаунта
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public override returns (bool) {
        _transfer(sender, recipient, amount);
        
        uint256 currentAllowance = _allowances[sender][msg.sender];
        require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
        
        unchecked {
            _approve(sender, msg.sender, currentAllowance - amount);
        }
        
        return true;
    }
    
    // ========== ВНУТРЕННИЕ ФУНКЦИИ ==========
    
    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal {
        require(sender != address(0), "ERC20: transfer from zero address");
        require(recipient != address(0), "ERC20: transfer to zero address");
        
        uint256 senderBalance = _balances[sender];
        require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
        
        unchecked {
            _balances[sender] = senderBalance - amount;
        }
        _balances[recipient] += amount;
        
        emit Transfer(sender, recipient, amount);
    }
    
    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal {
        require(owner != address(0), "ERC20: approve from zero address");
        require(spender != address(0), "ERC20: approve to zero address");
        
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
    
    function _mint(address account, uint256 amount) internal {
        require(account != address(0), "ERC20: mint to zero address");
        
        _totalSupply += amount;
        _balances[account] += amount;
        
        emit Transfer(address(0), account, amount);
    }
    
    function _burn(address account, uint256 amount) internal {
        require(account != address(0), "ERC20: burn from zero address");
        
        uint256 accountBalance = _balances[account];
        require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
        
        unchecked {
            _balances[account] = accountBalance - amount;
        }
        _totalSupply -= amount;
        
        emit Transfer(account, address(0), amount);
    }
}

// ========== РАСШИРЕННЫЙ ERC-20 С ДОПОЛНИТЕЛЬНЫМИ ФУНКЦИЯМИ ==========

contract AdvancedERC20 is BasicERC20 {
    address public owner;
    bool public paused;
    
    mapping(address => bool) public blacklist;
    
    event Paused(address account);
    event Unpaused(address account);
    event Blacklisted(address account);
    event Unblacklisted(address account);
    
    constructor(
        string memory _name,
        string memory _symbol,
        uint256 initialSupply
    ) BasicERC20(_name, _symbol, 18, initialSupply) {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }
    
    modifier notBlacklisted(address account) {
        require(!blacklist[account], "Account is blacklisted");
        _;
    }
    
    // Перевод с проверками
    function transfer(address recipient, uint256 amount)
        public
        override
        whenNotPaused
        notBlacklisted(msg.sender)
        notBlacklisted(recipient)
        returns (bool)
    {
        return super.transfer(recipient, amount);
    }
    
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    )
        public
        override
        whenNotPaused
        notBlacklisted(sender)
        notBlacklisted(recipient)
        returns (bool)
    {
        return super.transferFrom(sender, recipient, amount);
    }
    
    // Mint новых токенов (только owner)
    function mint(address account, uint256 amount) public onlyOwner {
        _mint(account, amount);
    }
    
    // Burn токенов
    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
    }
    
    // Burn токенов от имени другого
    function burnFrom(address account, uint256 amount) public {
        uint256 currentAllowance = allowance(account, msg.sender);
        require(currentAllowance >= amount, "Burn amount exceeds allowance");
        
        _approve(account, msg.sender, currentAllowance - amount);
        _burn(account, amount);
    }
    
    // Пауза
    function pause() public onlyOwner {
        paused = true;
        emit Paused(msg.sender);
    }
    
    function unpause() public onlyOwner {
        paused = false;
        emit Unpaused(msg.sender);
    }
    
    // Blacklist
    function addToBlacklist(address account) public onlyOwner {
        blacklist[account] = true;
        emit Blacklisted(account);
    }
    
    function removeFromBlacklist(address account) public onlyOwner {
        blacklist[account] = false;
        emit Unblacklisted(account);
    }
}

// ========== ПРИМЕР ИСПОЛЬЗОВАНИЯ ТОКЕНА ==========

contract TokenSale {
    AdvancedERC20 public token;
    address public owner;
    uint256 public tokenPrice; // цена в wei
    
    event TokensPurchased(address indexed buyer, uint256 amount, uint256 cost);
    event TokensSold(address indexed seller, uint256 amount, uint256 refund);
    
    constructor(address tokenAddress, uint256 _tokenPrice) {
        token = AdvancedERC20(tokenAddress);
        owner = msg.sender;
        tokenPrice = _tokenPrice;
    }
    
    // Купить токены
    function buyTokens() public payable {
        require(msg.value > 0, "Send ETH to buy tokens");
        
        uint256 tokenAmount = msg.value / tokenPrice;
        require(tokenAmount > 0, "Insufficient ETH");
        
        require(
            token.balanceOf(address(this)) >= tokenAmount,
            "Not enough tokens in contract"
        );
        
        token.transfer(msg.sender, tokenAmount);
        emit TokensPurchased(msg.sender, tokenAmount, msg.value);
    }
    
    // Продать токены обратно
    function sellTokens(uint256 tokenAmount) public {
        require(tokenAmount > 0, "Specify amount");
        require(
            token.balanceOf(msg.sender) >= tokenAmount,
            "Insufficient token balance"
        );
        
        uint256 ethAmount = tokenAmount * tokenPrice;
        require(
            address(this).balance >= ethAmount,
            "Contract has insufficient ETH"
        );
        
        // Пользователь должен сначала approve
        token.transferFrom(msg.sender, address(this), tokenAmount);
        
        payable(msg.sender).transfer(ethAmount);
        emit TokensSold(msg.sender, tokenAmount, ethAmount);
    }
    
    // Пополнить контракт токенами
    function fundContract(uint256 amount) public {
        token.transferFrom(msg.sender, address(this), amount);
    }
    
    // Вывести ETH (только owner)
    function withdrawETH() public {
        require(msg.sender == owner, "Not owner");
        payable(owner).transfer(address(this).balance);
    }
    
    receive() external payable {}
}

// ========== СТЕЙКИНГ ТОКЕНОВ ==========

contract TokenStaking {
    AdvancedERC20 public stakingToken;
    AdvancedERC20 public rewardToken;
    
    uint256 public rewardRate = 100; // 100 токенов в день
    
    struct Stake {
        uint256 amount;
        uint256 timestamp;
        uint256 rewardDebt;
    }
    
    mapping(address => Stake) public stakes;
    
    event Staked(address indexed user, uint256 amount);
    event Unstaked(address indexed user, uint256 amount);
    event RewardsClaimed(address indexed user, uint256 amount);
    
    constructor(address _stakingToken, address _rewardToken) {
        stakingToken = AdvancedERC20(_stakingToken);
        rewardToken = AdvancedERC20(_rewardToken);
    }
    
    // Застейкать токены
    function stake(uint256 amount) public {
        require(amount > 0, "Amount must be positive");
        
        // Сначала пользователь должен approve
        stakingToken.transferFrom(msg.sender, address(this), amount);
        
        Stake storage userStake = stakes[msg.sender];
        
        // Если уже есть стейк, выплачиваем награды
        if (userStake.amount > 0) {
            uint256 pending = calculateRewards(msg.sender);
            if (pending > 0) {
                rewardToken.transfer(msg.sender, pending);
            }
        }
        
        userStake.amount += amount;
        userStake.timestamp = block.timestamp;
        userStake.rewardDebt = 0;
        
        emit Staked(msg.sender, amount);
    }
    
    // Снять токены
    function unstake(uint256 amount) public {
        Stake storage userStake = stakes[msg.sender];
        require(userStake.amount >= amount, "Insufficient staked amount");
        
        // Выплачиваем награды
        uint256 pending = calculateRewards(msg.sender);
        if (pending > 0) {
            rewardToken.transfer(msg.sender, pending);
        }
        
        userStake.amount -= amount;
        userStake.timestamp = block.timestamp;
        userStake.rewardDebt = 0;
        
        stakingToken.transfer(msg.sender, amount);
        
        emit Unstaked(msg.sender, amount);
    }
    
    // Забрать награды
    function claimRewards() public {
        uint256 pending = calculateRewards(msg.sender);
        require(pending > 0, "No rewards available");
        
        Stake storage userStake = stakes[msg.sender];
        userStake.timestamp = block.timestamp;
        userStake.rewardDebt = 0;
        
        rewardToken.transfer(msg.sender, pending);
        
        emit RewardsClaimed(msg.sender, pending);
    }
    
    // Расчет наград
    function calculateRewards(address user) public view returns (uint256) {
        Stake memory userStake = stakes[user];
        if (userStake.amount == 0) return 0;
        
        uint256 stakingTime = block.timestamp - userStake.timestamp;
        uint256 rewards = (userStake.amount * rewardRate * stakingTime) / (1 days * 1e18);
        
        return rewards;
    }
    
    // Информация о стейке
    function getStakeInfo(address user) public view returns (
        uint256 stakedAmount,
        uint256 pendingRewards,
        uint256 stakingDuration
    ) {
        Stake memory userStake = stakes[user];
        stakedAmount = userStake.amount;
        pendingRewards = calculateRewards(user);
        stakingDuration = block.timestamp - userStake.timestamp;
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ ERC-20:

ОСНОВНЫЕ ФУНКЦИИ:
- totalSupply: общее количество токенов
- balanceOf: баланс аккаунта
- transfer: перевод токенов
- approve: разрешение на трату
- allowance: проверка разрешения
- transferFrom: перевод от имени другого

ДОПОЛНИТЕЛЬНЫЕ ФУНКЦИИ:
- mint: создание новых токенов
- burn: сжигание токенов
- pause/unpause: остановка переводов
- blacklist: блокировка аккаунтов

ИСПОЛЬЗОВАНИЕ:
1. Approve токены перед transferFrom
2. Проверяйте баланс перед переводом
3. Обрабатывайте ошибки
4. Используйте события для отслеживания

БЕЗОПАСНОСТЬ:
- Проверка на zero address
- Проверка достаточности баланса
- Проверка allowance
- События для всех изменений

ПАТТЕРНЫ:
- Token Sale: продажа токенов
- Staking: застейкивание для наград
- Vesting: постепенная разблокировка
- Airdrop: массовая раздача
*/
````
### Практический проект - DeFi приложение
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ПРОСТАЯ ДЕЦЕНТРАЛИЗОВАННАЯ БИРЖА (DEX) ==========

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
}

// ========== LIQUIDITY POOL (AMM - Automated Market Maker) ==========

contract SimpleDEX {
    IERC20 public tokenA;
    IERC20 public tokenB;
    
    uint256 public reserveA;
    uint256 public reserveB;
    
    uint256 public totalLiquidity;
    mapping(address => uint256) public liquidity;
    
    uint256 public constant FEE_PERCENT = 3; // 0.3% комиссия
    
    event LiquidityAdded(address indexed provider, uint256 amountA, uint256 amountB, uint256 liquidityMinted);
    event LiquidityRemoved(address indexed provider, uint256 amountA, uint256 amountB, uint256 liquidityBurned);
    event Swap(address indexed user, address tokenIn, uint256 amountIn, uint256 amountOut);
    
    constructor(address _tokenA, address _tokenB) {
        tokenA = IERC20(_tokenA);
        tokenB = IERC20(_tokenB);
    }
    
    // ========== ДОБАВИТЬ ЛИКВИДНОСТЬ ==========
    
    function addLiquidity(uint256 amountA, uint256 amountB) public returns (uint256 liquidityMinted) {
        require(amountA > 0 && amountB > 0, "Amounts must be positive");
        
        // Первое добавление ликвидности
        if (totalLiquidity == 0) {
            liquidityMinted = sqrt(amountA * amountB);
        } else {
            // Поддержание пропорции резервов
            uint256 liquidityA = (amountA * totalLiquidity) / reserveA;
            uint256 liquidityB = (amountB * totalLiquidity) / reserveB;
            liquidityMinted = min(liquidityA, liquidityB);
        }
        
        require(liquidityMinted > 0, "Insufficient liquidity minted");
        
        // Переводим токены от пользователя
        tokenA.transferFrom(msg.sender, address(this), amountA);
        tokenB.transferFrom(msg.sender, address(this), amountB);
        
        // Обновляем резервы
        reserveA += amountA;
        reserveB += amountB;
        
        // Начисляем ликвидность
        liquidity[msg.sender] += liquidityMinted;
        totalLiquidity += liquidityMinted;
        
        emit LiquidityAdded(msg.sender, amountA, amountB, liquidityMinted);
    }
    
    // ========== УБРАТЬ ЛИКВИДНОСТЬ ==========
    
    function removeLiquidity(uint256 liquidityAmount) public returns (uint256 amountA, uint256 amountB) {
        require(liquidityAmount > 0, "Amount must be positive");
        require(liquidity[msg.sender] >= liquidityAmount, "Insufficient liquidity");
        
        // Рассчитываем долю токенов
        amountA = (liquidityAmount * reserveA) / totalLiquidity;
        amountB = (liquidityAmount * reserveB) / totalLiquidity;
        
        require(amountA > 0 && amountB > 0, "Insufficient amounts");
        
        // Обновляем состояние
        liquidity[msg.sender] -= liquidityAmount;
        totalLiquidity -= liquidityAmount;
        reserveA -= amountA;
        reserveB -= amountB;
        
        // Отправляем токены
        tokenA.transfer(msg.sender, amountA);
        tokenB.transfer(msg.sender, amountB);
        
        emit LiquidityRemoved(msg.sender, amountA, amountB, liquidityAmount);
    }
    
    // ========== ОБМЕН ТОКЕНОВ (SWAP) ==========
    
    function swap(address tokenIn, uint256 amountIn) public returns (uint256 amountOut) {
        require(amountIn > 0, "Amount must be positive");
        require(
            tokenIn == address(tokenA) || tokenIn == address(tokenB),
            "Invalid token"
        );
        
        bool isTokenA = tokenIn == address(tokenA);
        
        (IERC20 inputToken, IERC20 outputToken, uint256 inputReserve, uint256 outputReserve) = isTokenA
            ? (tokenA, tokenB, reserveA, reserveB)
            : (tokenB, tokenA, reserveB, reserveA);
        
        // Переводим входной токен
        inputToken.transferFrom(msg.sender, address(this), amountIn);
        
        // Рассчитываем выходное количество (с учетом комиссии)
        uint256 amountInWithFee = amountIn * (1000 - FEE_PERCENT);
        amountOut = (amountInWithFee * outputReserve) / (inputReserve * 1000 + amountInWithFee);
        
        require(amountOut > 0, "Insufficient output amount");
        require(amountOut < outputReserve, "Insufficient liquidity");
        
        // Обновляем резервы
        if (isTokenA) {
            reserveA += amountIn;
            reserveB -= amountOut;
        } else {
            reserveB += amountIn;
            reserveA -= amountOut;
        }
        
        // Отправляем выходной токен
        outputToken.transfer(msg.sender, amountOut);
        
        emit Swap(msg.sender, tokenIn, amountIn, amountOut);
    }
    
    // ========== РАСЧЕТ ЦЕНЫ ==========
    
    function getAmountOut(address tokenIn, uint256 amountIn) public view returns (uint256) {
        require(amountIn > 0, "Amount must be positive");
        require(
            tokenIn == address(tokenA) || tokenIn == address(tokenB),
            "Invalid token"
        );
        
        bool isTokenA = tokenIn == address(tokenA);
        (uint256 inputReserve, uint256 outputReserve) = isTokenA
            ? (reserveA, reserveB)
            : (reserveB, reserveA);
        
        uint256 amountInWithFee = amountIn * (1000 - FEE_PERCENT);
        uint256 amountOut = (amountInWithFee * outputReserve) / (inputReserve * 1000 + amountInWithFee);
        
        return amountOut;
    }
    
    // ========== ИНФОРМАЦИЯ О ПУЛЕ ==========
    
    function getReserves() public view returns (uint256, uint256) {
        return (reserveA, reserveB);
    }
    
    function getUserLiquidity(address user) public view returns (uint256) {
        return liquidity[user];
    }
    
    function getPrice() public view returns (uint256 priceAtoB, uint256 priceBtoA) {
        require(reserveA > 0 && reserveB > 0, "No liquidity");
        priceAtoB = (reserveB * 1e18) / reserveA;
        priceBtoA = (reserveA * 1e18) / reserveB;
    }
    
    // ========== ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ==========
    
    function sqrt(uint256 x) internal pure returns (uint256) {
        if (x == 0) return 0;
        uint256 z = (x + 1) / 2;
        uint256 y = x;
        while (z < y) {
            y = z;
            z = (x / z + z) / 2;
        }
        return y;
    }
    
    function min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a < b ? a : b;
    }
}

// ========== LENDING PROTOCOL (Кредитование) ==========

contract SimpleLending {
    IERC20 public token;
    
    uint256 public constant COLLATERAL_RATIO = 150; // 150% залог
    uint256 public constant INTEREST_RATE = 5; // 5% годовых
    
    struct Deposit {
        uint256 amount;
        uint256 timestamp;
    }
    
    struct Loan {
        uint256 collateral;
        uint256 borrowed;
        uint256 timestamp;
    }
    
    mapping(address => Deposit) public deposits;
    mapping(address => Loan) public loans;
    
    uint256 public totalDeposits;
    uint256 public totalBorrowed;
    
    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);
    event Borrowed(address indexed user, uint256 amount, uint256 collateral);
    event Repaid(address indexed user, uint256 amount);
    event Liquidated(address indexed user, address indexed liquidator, uint256 amount);
    
    constructor(address _token) {
        token = IERC20(_token);
    }
    
    // ========== ДЕПОЗИТ ==========
    
    function deposit(uint256 amount) public {
        require(amount > 0, "Amount must be positive");
        
        token.transferFrom(msg.sender, address(this), amount);
        
        deposits[msg.sender].amount += amount;
        deposits[msg.sender].timestamp = block.timestamp;
        totalDeposits += amount;
        
        emit Deposited(msg.sender, amount);
    }
    
    // ========== ВЫВОД ==========
    
    function withdraw(uint256 amount) public {
        require(amount > 0, "Amount must be positive");
        require(deposits[msg.sender].amount >= amount, "Insufficient balance");
        require(totalDeposits - amount >= totalBorrowed, "Insufficient liquidity");
        
        deposits[msg.sender].amount -= amount;
        totalDeposits -= amount;
        
        token.transfer(msg.sender, amount);
        
        emit Withdrawn(msg.sender, amount);
    }
    
    // ========== ВЗЯТЬ КРЕДИТ ==========
    
    function borrow(uint256 amount, uint256 collateralAmount) public {
        require(amount > 0, "Amount must be positive");
        require(collateralAmount >= (amount * COLLATERAL_RATIO) / 100, "Insufficient collateral");
        require(totalDeposits >= totalBorrowed + amount, "Insufficient liquidity");
        
        token.transferFrom(msg.sender, address(this), collateralAmount);
        
        loans[msg.sender].collateral += collateralAmount;
        loans[msg.sender].borrowed += amount;
        loans[msg.sender].timestamp = block.timestamp;
        totalBorrowed += amount;
        
        token.transfer(msg.sender, amount);
        
        emit Borrowed(msg.sender, amount, collateralAmount);
    }
    
    // ========== ПОГАСИТЬ КРЕДИТ ==========
    
    function repay(uint256 amount) public {
        Loan storage loan = loans[msg.sender];
        require(loan.borrowed > 0, "No active loan");
        
        uint256 interest = calculateInterest(msg.sender);
        uint256 totalOwed = loan.borrowed + interest;
        require(amount <= totalOwed, "Amount exceeds debt");
        
        token.transferFrom(msg.sender, address(this), amount);
        
        if (amount >= totalOwed) {
            // Полное погашение - возврат залога
            uint256 collateralToReturn = loan.collateral;
            totalBorrowed -= loan.borrowed;
            
            delete loans[msg.sender];
            
            token.transfer(msg.sender, collateralToReturn);
        } else {
            // Частичное погашение
            loan.borrowed -= amount;
            loan.timestamp = block.timestamp;
        }
        
        emit Repaid(msg.sender, amount);
    }
    
    // ========== ЛИКВИДАЦИЯ ==========
    
    function liquidate(address borrower) public {
        Loan storage loan = loans[borrower];
        require(loan.borrowed > 0, "No active loan");
        
        uint256 interest = calculateInterest(borrower);
        uint256 totalOwed = loan.borrowed + interest;
        uint256 requiredCollateral = (totalOwed * COLLATERAL_RATIO) / 100;
        
        require(loan.collateral < requiredCollateral, "Loan is healthy");
        
        // Ликвидатор платит долг
        token.transferFrom(msg.sender, address(this), totalOwed);
        
        // Ликвидатор получает залог
        uint256 collateral = loan.collateral;
        totalBorrowed -= loan.borrowed;
        
        delete loans[borrower];
        
        token.transfer(msg.sender, collateral);
        
        emit Liquidated(borrower, msg.sender, totalOwed);
    }
    
    // ========== РАСЧЕТ ПРОЦЕНТОВ ==========
    
    function calculateInterest(address borrower) public view returns (uint256) {
        Loan memory loan = loans[borrower];
        if (loan.borrowed == 0) return 0;
        
        uint256 timeElapsed = block.timestamp - loan.timestamp;
        uint256 interest = (loan.borrowed * INTEREST_RATE * timeElapsed) / (365 days * 100);
        
        return interest;
    }
    
    // ========== ИНФОРМАЦИЯ О КРЕДИТЕ ==========
    
    function getLoanHealth(address borrower) public view returns (uint256) {
        Loan memory loan = loans[borrower];
        if (loan.borrowed == 0) return 0;
        
        uint256 interest = calculateInterest(borrower);
        uint256 totalOwed = loan.borrowed + interest;
        
        return (loan.collateral * 100) / totalOwed;
    }
    
    function getUserInfo(address user) public view returns (
        uint256 depositAmount,
        uint256 borrowedAmount,
        uint256 collateralAmount,
        uint256 interestOwed,
        uint256 healthFactor
    ) {
        depositAmount = deposits[user].amount;
        borrowedAmount = loans[user].borrowed;
        collateralAmount = loans[user].collateral;
        interestOwed = calculateInterest(user);
        healthFactor = getLoanHealth(user);
    }
}

// ========== STAKING С НАГРАДАМИ ==========

contract YieldFarming {
    IERC20 public stakingToken;
    IERC20 public rewardToken;
    
    uint256 public rewardRate = 100; // токенов в секунду на весь пул
    uint256 public lastUpdateTime;
    uint256 public rewardPerTokenStored;
    
    mapping(address => uint256) public userRewardPerTokenPaid;
    mapping(address => uint256) public rewards;
    
    uint256 public totalStaked;
    mapping(address => uint256) public balances;
    
    event Staked(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);
    event RewardPaid(address indexed user, uint256 reward);
    
    constructor(address _stakingToken, address _rewardToken) {
        stakingToken = IERC20(_stakingToken);
        rewardToken = IERC20(_rewardToken);
        lastUpdateTime = block.timestamp;
    }
    
    // ========== МОДИФИКАТОРЫ ДЛЯ ОБНОВЛЕНИЯ НАГРАД ==========
    
    modifier updateReward(address account) {
        rewardPerTokenStored = rewardPerToken();
        lastUpdateTime = block.timestamp;
        
        if (account != address(0)) {
            rewards[account] = earned(account);
            userRewardPerTokenPaid[account] = rewardPerTokenStored;
        }
        _;
    }
    
    // ========== ФУНКЦИИ ==========
    
    function stake(uint256 amount) public updateReward(msg.sender) {
        require(amount > 0, "Cannot stake 0");
        
        totalStaked += amount;
        balances[msg.sender] += amount;
        
        stakingToken.transferFrom(msg.sender, address(this), amount);
        
        emit Staked(msg.sender, amount);
    }
    
    function withdraw(uint256 amount) public updateReward(msg.sender) {
        require(amount > 0, "Cannot withdraw 0");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        totalStaked -= amount;
        balances[msg.sender] -= amount;
        
        stakingToken.transfer(msg.sender, amount);
        
        emit Withdrawn(msg.sender, amount);
    }
    
    function getReward() public updateReward(msg.sender) {
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            rewardToken.transfer(msg.sender, reward);
            emit RewardPaid(msg.sender, reward);
        }
    }
    
    function exit() external {
        withdraw(balances[msg.sender]);
        getReward();
    }
    
    // ========== РАСЧЕТ НАГРАД ==========
    
    function rewardPerToken() public view returns (uint256) {
        if (totalStaked == 0) {
            return rewardPerTokenStored;
        }
        return rewardPerTokenStored + 
            (((block.timestamp - lastUpdateTime) * rewardRate * 1e18) / totalStaked);
    }
    
    function earned(address account) public view returns (uint256) {
        return ((balances[account] * 
            (rewardPerToken() - userRewardPerTokenPaid[account])) / 1e18) + 
            rewards[account];
    }
    
    // ========== ИНФОРМАЦИЯ ==========
    
    function getStakeInfo(address user) public view returns (
        uint256 stakedAmount,
        uint256 earnedRewards,
        uint256 totalPoolStaked
    ) {
        stakedAmount = balances[user];
        earnedRewards = earned(user);
        totalPoolStaked = totalStaked;
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ DeFi:

DEX (Децентрализованная биржа):
- AMM (Automated Market Maker)
- Liquidity Pools (пулы ликвидности)
- Формула x * y = k (константное произведение)
- Комиссии для провайдеров ликвидности

LENDING (Кредитование):
- Депозиты под проценты
- Займы под залог
- Коэффициент залога (collateral ratio)
- Ликвидация при недостаточном залоге
- Расчет процентов

YIELD FARMING (Фарминг):
- Стейкинг токенов
- Начисление наград
- Reward rate (скорость наград)
- Составные проценты

ВАЖНЫЕ ПАТТЕРНЫ:
- updateReward modifier для синхронизации
- Pull over Push для выплат
- Проверка здоровья кредита
- Slippage protection

БЕЗОПАСНОСТЬ:
- Проверка approve перед transferFrom
- Защита от reentrancy
- Проверка достаточности ликвидности
- Валидация входных данных
- События для всех операций

МАТЕМАТИКА:
- Расчет цены через резервы
- Формула AMM: amountOut = (amountIn * reserveOut) / (reserveIn + amountIn)
- Проценты: (principal * rate * time) / (365 days * 100)
- Здоровье кредита: (collateral * 100) / debt
*/

````
Практические задания:
* Задание 1 (Легкое): Создайте простой контракт кошелька с мультиподписью (multisig), где для вывода средств нужно 2 из 3 владельцев.
* Задание 2 (Среднее): Реализуйте NFT маркетплейс, где можно:

Выставить NFT на продажу
* Купить NFT
* Отменить продажу
* Сделать аукцион

Задание 3 (Сложное): Создайте DAO (Decentralized Autonomous Organization):

* Создание предложений (proposals)
* Голосование токенами
* Исполнение одобренных предложений
* Таймлок для исполнения
### ERC-721 (NFT) - Невзаимозаменяемые токены
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ERC-721 ИНТЕРФЕЙС ==========

interface IERC721 {
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);
    event ApprovalForAll(address indexed owner, address indexed operator, bool approved);
    
    function balanceOf(address owner) external view returns (uint256 balance);
    function ownerOf(uint256 tokenId) external view returns (address owner);
    function safeTransferFrom(address from, address to, uint256 tokenId) external;
    function transferFrom(address from, address to, uint256 tokenId) external;
    function approve(address to, uint256 tokenId) external;
    function getApproved(uint256 tokenId) external view returns (address operator);
    function setApprovalForAll(address operator, bool approved) external;
    function isApprovedForAll(address owner, address operator) external view returns (bool);
    function safeTransferFrom(address from, address to, uint256 tokenId, bytes calldata data) external;
}

interface IERC721Metadata {
    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function tokenURI(uint256 tokenId) external view returns (string memory);
}

interface IERC721Receiver {
    function onERC721Received(
        address operator,
        address from,
        uint256 tokenId,
        bytes calldata data
    ) external returns (bytes4);
}

// ========== БАЗОВАЯ РЕАЛИЗАЦИЯ ERC-721 ==========

contract ERC721 is IERC721, IERC721Metadata {
    string public name;
    string public symbol;
    
    // Маппинги для отслеживания NFT
    mapping(uint256 => address) private _owners;
    mapping(address => uint256) private _balances;
    mapping(uint256 => address) private _tokenApprovals;
    mapping(address => mapping(address => bool)) private _operatorApprovals;
    
    constructor(string memory _name, string memory _symbol) {
        name = _name;
        symbol = _symbol;
    }
    
    // ========== ОСНОВНЫЕ ФУНКЦИИ ==========
    
    function balanceOf(address owner) public view override returns (uint256) {
        require(owner != address(0), "ERC721: balance query for zero address");
        return _balances[owner];
    }
    
    function ownerOf(uint256 tokenId) public view override returns (address) {
        address owner = _owners[tokenId];
        require(owner != address(0), "ERC721: owner query for nonexistent token");
        return owner;
    }
    
    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
        require(_exists(tokenId), "ERC721: URI query for nonexistent token");
        
        string memory baseURI = _baseURI();
        return bytes(baseURI).length > 0 
            ? string(abi.encodePacked(baseURI, _toString(tokenId)))
            : "";
    }
    
    function _baseURI() internal view virtual returns (string memory) {
        return "";
    }
    
    // ========== APPROVE ФУНКЦИИ ==========
    
    function approve(address to, uint256 tokenId) public override {
        address owner = ownerOf(tokenId);
        require(to != owner, "ERC721: approval to current owner");
        require(
            msg.sender == owner || isApprovedForAll(owner, msg.sender),
            "ERC721: approve caller is not owner nor approved for all"
        );
        
        _approve(to, tokenId);
    }
    
    function getApproved(uint256 tokenId) public view override returns (address) {
        require(_exists(tokenId), "ERC721: approved query for nonexistent token");
        return _tokenApprovals[tokenId];
    }
    
    function setApprovalForAll(address operator, bool approved) public override {
        require(operator != msg.sender, "ERC721: approve to caller");
        _operatorApprovals[msg.sender][operator] = approved;
        emit ApprovalForAll(msg.sender, operator, approved);
    }
    
    function isApprovedForAll(address owner, address operator) public view override returns (bool) {
        return _operatorApprovals[owner][operator];
    }
    
    // ========== TRANSFER ФУНКЦИИ ==========
    
    function transferFrom(address from, address to, uint256 tokenId) public override {
        require(_isApprovedOrOwner(msg.sender, tokenId), "ERC721: transfer caller is not owner nor approved");
        _transfer(from, to, tokenId);
    }
    
    function safeTransferFrom(address from, address to, uint256 tokenId) public override {
        safeTransferFrom(from, to, tokenId, "");
    }
    
    function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data) public override {
        require(_isApprovedOrOwner(msg.sender, tokenId), "ERC721: transfer caller is not owner nor approved");
        _safeTransfer(from, to, tokenId, data);
    }
    
    // ========== ВНУТРЕННИЕ ФУНКЦИИ ==========
    
    function _safeTransfer(address from, address to, uint256 tokenId, bytes memory data) internal {
        _transfer(from, to, tokenId);
        require(_checkOnERC721Received(from, to, tokenId, data), "ERC721: transfer to non ERC721Receiver");
    }
    
    function _exists(uint256 tokenId) internal view returns (bool) {
        return _owners[tokenId] != address(0);
    }
    
    function _isApprovedOrOwner(address spender, uint256 tokenId) internal view returns (bool) {
        require(_exists(tokenId), "ERC721: operator query for nonexistent token");
        address owner = ownerOf(tokenId);
        return (spender == owner || getApproved(tokenId) == spender || isApprovedForAll(owner, spender));
    }
    
    function _safeMint(address to, uint256 tokenId) internal {
        _safeMint(to, tokenId, "");
    }
    
    function _safeMint(address to, uint256 tokenId, bytes memory data) internal {
        _mint(to, tokenId);
        require(_checkOnERC721Received(address(0), to, tokenId, data), "ERC721: transfer to non ERC721Receiver");
    }
    
    function _mint(address to, uint256 tokenId) internal {
        require(to != address(0), "ERC721: mint to zero address");
        require(!_exists(tokenId), "ERC721: token already minted");
        
        _balances[to] += 1;
        _owners[tokenId] = to;
        
        emit Transfer(address(0), to, tokenId);
    }
    
    function _burn(uint256 tokenId) internal {
        address owner = ownerOf(tokenId);
        
        _approve(address(0), tokenId);
        
        _balances[owner] -= 1;
        delete _owners[tokenId];
        
        emit Transfer(owner, address(0), tokenId);
    }
    
    function _transfer(address from, address to, uint256 tokenId) internal {
        require(ownerOf(tokenId) == from, "ERC721: transfer from incorrect owner");
        require(to != address(0), "ERC721: transfer to zero address");
        
        _approve(address(0), tokenId);
        
        _balances[from] -= 1;
        _balances[to] += 1;
        _owners[tokenId] = to;
        
        emit Transfer(from, to, tokenId);
    }
    
    function _approve(address to, uint256 tokenId) internal {
        _tokenApprovals[tokenId] = to;
        emit Approval(ownerOf(tokenId), to, tokenId);
    }
    
    function _checkOnERC721Received(
        address from,
        address to,
        uint256 tokenId,
        bytes memory data
    ) private returns (bool) {
        if (to.code.length > 0) {
            try IERC721Receiver(to).onERC721Received(msg.sender, from, tokenId, data) returns (bytes4 retval) {
                return retval == IERC721Receiver.onERC721Received.selector;
            } catch {
                return false;
            }
        } else {
            return true;
        }
    }
    
    function _toString(uint256 value) internal pure returns (string memory) {
        if (value == 0) return "0";
        uint256 temp = value;
        uint256 digits;
        while (temp != 0) {
            digits++;
            temp /= 10;
        }
        bytes memory buffer = new bytes(digits);
        while (value != 0) {
            digits -= 1;
            buffer[digits] = bytes1(uint8(48 + uint256(value % 10)));
            value /= 10;
        }
        return string(buffer);
    }
}

// ========== ПОЛНОФУНКЦИОНАЛЬНЫЙ NFT КОНТРАКТ ==========

contract MyNFT is ERC721 {
    uint256 private _tokenIdCounter;
    address public owner;
    
    string private _baseTokenURI;
    uint256 public maxSupply;
    uint256 public mintPrice;
    
    mapping(uint256 => string) private _tokenURIs;
    
    event NFTMinted(address indexed to, uint256 indexed tokenId);
    event BaseURIUpdated(string newBaseURI);
    
    constructor(
        string memory name,
        string memory symbol,
        uint256 _maxSupply,
        uint256 _mintPrice
    ) ERC721(name, symbol) {
        owner = msg.sender;
        maxSupply = _maxSupply;
        mintPrice = _mintPrice;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    // ========== МИНТ ФУНКЦИИ ==========
    
    function mint(address to) public payable returns (uint256) {
        require(_tokenIdCounter < maxSupply, "Max supply reached");
        require(msg.value >= mintPrice, "Insufficient payment");
        
        uint256 tokenId = _tokenIdCounter;
        _tokenIdCounter++;
        
        _safeMint(to, tokenId);
        
        emit NFTMinted(to, tokenId);
        return tokenId;
    }
    
    function mintBatch(address to, uint256 quantity) public payable returns (uint256[] memory) {
        require(_tokenIdCounter + quantity <= maxSupply, "Exceeds max supply");
        require(msg.value >= mintPrice * quantity, "Insufficient payment");
        
        uint256[] memory tokenIds = new uint256[](quantity);
        
        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = _tokenIdCounter;
            _tokenIdCounter++;
            _safeMint(to, tokenId);
            tokenIds[i] = tokenId;
            emit NFTMinted(to, tokenId);
        }
        
        return tokenIds;
    }
    
    function ownerMint(address to, uint256 quantity) public onlyOwner {
        require(_tokenIdCounter + quantity <= maxSupply, "Exceeds max supply");
        
        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = _tokenIdCounter;
            _tokenIdCounter++;
            _safeMint(to, tokenId);
            emit NFTMinted(to, tokenId);
        }
    }
    
    // ========== URI ФУНКЦИИ ==========
    
    function _baseURI() internal view override returns (string memory) {
        return _baseTokenURI;
    }
    
    function setBaseURI(string memory baseURI) public onlyOwner {
        _baseTokenURI = baseURI;
        emit BaseURIUpdated(baseURI);
    }
    
    function setTokenURI(uint256 tokenId, string memory _tokenURI) public onlyOwner {
        require(_exists(tokenId), "Token does not exist");
        _tokenURIs[tokenId] = _tokenURI;
    }
    
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_exists(tokenId), "Token does not exist");
        
        string memory _tokenURI = _tokenURIs[tokenId];
        string memory base = _baseURI();
        
        if (bytes(_tokenURI).length > 0) {
            return _tokenURI;
        }
        
        if (bytes(base).length > 0) {
            return string(abi.encodePacked(base, _toString(tokenId), ".json"));
        }
        
        return "";
    }
    
    // ========== ДОПОЛНИТЕЛЬНЫЕ ФУНКЦИИ ==========
    
    function totalSupply() public view returns (uint256) {
        return _tokenIdCounter;
    }
    
    function tokensOfOwner(address _owner) public view returns (uint256[] memory) {
        uint256 ownerBalance = balanceOf(_owner);
        uint256[] memory tokenIds = new uint256[](ownerBalance);
        uint256 currentIndex = 0;
        
        for (uint256 i = 0; i < _tokenIdCounter; i++) {
            if (_exists(i) && ownerOf(i) == _owner) {
                tokenIds[currentIndex] = i;
                currentIndex++;
            }
        }
        
        return tokenIds;
    }
    
    function burn(uint256 tokenId) public {
        require(_isApprovedOrOwner(msg.sender, tokenId), "Not approved to burn");
        _burn(tokenId);
    }
    
    function setMintPrice(uint256 newPrice) public onlyOwner {
        mintPrice = newPrice;
    }
    
    function withdraw() public onlyOwner {
        uint256 balance = address(this).balance;
        payable(owner).transfer(balance);
    }
}

// ========== NFT MARKETPLACE ==========

contract NFTMarketplace {
    struct Listing {
        address seller;
        address nftContract;
        uint256 tokenId;
        uint256 price;
        bool active;
    }
    
    struct Offer {
        address buyer;
        uint256 price;
        bool active;
    }
    
    uint256 public listingCounter;
    uint256 public feePercent = 25; // 2.5%
    address public feeRecipient;
    
    mapping(uint256 => Listing) public listings;
    mapping(address => mapping(uint256 => uint256)) public nftToListingId;
    mapping(uint256 => Offer[]) public offers;
    
    event NFTListed(uint256 indexed listingId, address indexed seller, address nftContract, uint256 tokenId, uint256 price);
    event NFTSold(uint256 indexed listingId, address indexed buyer, uint256 price);
    event ListingCancelled(uint256 indexed listingId);
    event OfferMade(uint256 indexed listingId, address indexed buyer, uint256 price);
    event OfferAccepted(uint256 indexed listingId, uint256 offerIndex);
    
    constructor(address _feeRecipient) {
        feeRecipient = _feeRecipient;
    }
    
    // ========== ЛИСТИНГ ==========
    
    function listNFT(address nftContract, uint256 tokenId, uint256 price) public returns (uint256) {
        require(price > 0, "Price must be positive");
        
        IERC721 nft = IERC721(nftContract);
        require(nft.ownerOf(tokenId) == msg.sender, "Not the owner");
        require(
            nft.isApprovedForAll(msg.sender, address(this)) || 
            nft.getApproved(tokenId) == address(this),
            "Marketplace not approved"
        );
        
        uint256 listingId = listingCounter++;
        
        listings[listingId] = Listing({
            seller: msg.sender,
            nftContract: nftContract,
            tokenId: tokenId,
            price: price,
            active: true
        });
        
        nftToListingId[nftContract][tokenId] = listingId;
        
        emit NFTListed(listingId, msg.sender, nftContract, tokenId, price);
        return listingId;
    }
    
    function cancelListing(uint256 listingId) public {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(listing.seller == msg.sender, "Not the seller");
        
        listing.active = false;
        delete nftToListingId[listing.nftContract][listing.tokenId];
        
        emit ListingCancelled(listingId);
    }
    
    function updatePrice(uint256 listingId, uint256 newPrice) public {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(listing.seller == msg.sender, "Not the seller");
        require(newPrice > 0, "Price must be positive");
        
        listing.price = newPrice;
    }
    
    // ========== ПОКУПКА ==========
    
    function buyNFT(uint256 listingId) public payable {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(msg.value >= listing.price, "Insufficient payment");
        
        listing.active = false;
        delete nftToListingId[listing.nftContract][listing.tokenId];
        
        // Расчет комиссии
        uint256 fee = (listing.price * feePercent) / 1000;
        uint256 sellerAmount = listing.price - fee;
        
        // Переводы
        IERC721(listing.nftContract).transferFrom(listing.seller, msg.sender, listing.tokenId);
        payable(listing.seller).transfer(sellerAmount);
        payable(feeRecipient).transfer(fee);
        
        // Возврат излишка
        if (msg.value > listing.price) {
            payable(msg.sender).transfer(msg.value - listing.price);
        }
        
        emit NFTSold(listingId, msg.sender, listing.price);
    }
    
    // ========== ОФФЕРЫ ==========
    
    function makeOffer(uint256 listingId) public payable {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(msg.value > 0, "Offer must be positive");
        
        offers[listingId].push(Offer({
            buyer: msg.sender,
            price: msg.value,
            active: true
        }));
        
        emit OfferMade(listingId, msg.sender, msg.value);
    }
    
    function acceptOffer(uint256 listingId, uint256 offerIndex) public {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(listing.seller == msg.sender, "Not the seller");
        
        Offer storage offer = offers[listingId][offerIndex];
        require(offer.active, "Offer not active");
        
        listing.active = false;
        offer.active = false;
        delete nftToListingId[listing.nftContract][listing.tokenId];
        
        uint256 fee = (offer.price * feePercent) / 1000;
        uint256 sellerAmount = offer.price - fee;
        
        IERC721(listing.nftContract).transferFrom(msg.sender, offer.buyer, listing.tokenId);
        payable(msg.sender).transfer(sellerAmount);
        payable(feeRecipient).transfer(fee);
        
        // Возврат остальных офферов
        for (uint256 i = 0; i < offers[listingId].length; i++) {
            if (i != offerIndex && offers[listingId][i].active) {
                Offer storage otherOffer = offers[listingId][i];
                otherOffer.active = false;
                payable(otherOffer.buyer).transfer(otherOffer.price);
            }
        }
        
        emit OfferAccepted(listingId, offerIndex);
    }
    
    function cancelOffer(uint256 listingId, uint256 offerIndex) public {
        Offer storage offer = offers[listingId][offerIndex];
        require(offer.active, "Offer not active");
        require(offer.buyer == msg.sender, "Not the buyer");
        
        offer.active = false;
        payable(msg.sender).transfer(offer.price);
    }
    
    // ========== ИНФОРМАЦИЯ ==========
    
    function getActiveListings() public view returns (uint256[] memory) {
        uint256 activeCount = 0;
        for (uint256 i = 0; i < listingCounter; i++) {
            if (listings[i].active) activeCount++;
        }
        
        uint256[] memory activeListings = new uint256[](activeCount);
        uint256 currentIndex = 0;
        
        for (uint256 i = 0; i < listingCounter; i++) {
            if (listings[i].active) {
                activeListings[currentIndex] = i;
                currentIndex++;
            }
        }
        
        return activeListings;
    }
    
    function getOffers(uint256 listingId) public view returns (Offer[] memory) {
        return offers[listingId];
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ ERC-721:

ОСНОВНЫЕ ФУНКЦИИ:
- balanceOf: количество NFT у владельца
- ownerOf: владелец конкретного токена
- transferFrom: обычный перевод
- safeTransferFrom: безопасный перевод с проверкой
- approve: разрешение на управление токеном
- setApprovalForAll: разрешение на все токены

METADATA:
- tokenURI: ссылка на метаданные (JSON)
- Обычно хранится на IPFS
- Содержит image, name, description, attributes

БЕЗОПАСНОСТЬ:
- _checkOnERC721Received: проверка получателя
- _isApprovedOrOwner: проверка прав
- require проверки во всех функциях

MARKETPLACE:
- Листинг NFT на продажу
- Прямая покупка
- Система офферов
- Комиссия маркетплейса

ПАТТЕРНЫ:
- Enumeration: отслеживание всех токенов
- Batch operations: массовый минт
- Royalties: авторские отчисления
- Whitelist: ограничение минта
*/
````
### ERC-1155 - Мультитокен стандарт
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ERC-1155 ИНТЕРФЕЙС ==========

interface IERC1155 {
    event TransferSingle(address indexed operator, address indexed from, address indexed to, uint256 id, uint256 value);
    event TransferBatch(address indexed operator, address indexed from, address indexed to, uint256[] ids, uint256[] values);
    event ApprovalForAll(address indexed account, address indexed operator, bool approved);
    event URI(string value, uint256 indexed id);
    
    function balanceOf(address account, uint256 id) external view returns (uint256);
    function balanceOfBatch(address[] calldata accounts, uint256[] calldata ids) external view returns (uint256[] memory);
    function setApprovalForAll(address operator, bool approved) external;
    function isApprovedForAll(address account, address operator) external view returns (bool);
    function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) external;
    function safeBatchTransferFrom(address from, address to, uint256[] calldata ids, uint256[] calldata amounts, bytes calldata data) external;
}

interface IERC1155MetadataURI {
    function uri(uint256 id) external view returns (string memory);
}

interface IERC1155Receiver {
    function onERC1155Received(
        address operator,
        address from,
        uint256 id,
        uint256 value,
        bytes calldata data
    ) external returns (bytes4);
    
    function onERC1155BatchReceived(
        address operator,
        address from,
        uint256[] calldata ids,
        uint256[] calldata values,
        bytes calldata data
    ) external returns (bytes4);
}

// ========== БАЗОВАЯ РЕАЛИЗАЦИЯ ERC-1155 ==========

contract ERC1155 is IERC1155, IERC1155MetadataURI {
    // Балансы: владелец => id токена => количество
    mapping(address => mapping(uint256 => uint256)) private _balances;
    
    // Разрешения: владелец => оператор => разрешено
    mapping(address => mapping(address => bool)) private _operatorApprovals;
    
    string private _uri;
    
    constructor(string memory uri_) {
        _uri = uri_;
    }
    
    // ========== ОСНОВНЫЕ ФУНКЦИИ ==========
    
    function uri(uint256) public view virtual override returns (string memory) {
        return _uri;
    }
    
    function balanceOf(address account, uint256 id) public view override returns (uint256) {
        require(account != address(0), "ERC1155: balance query for zero address");
        return _balances[account][id];
    }
    
    function balanceOfBatch(
        address[] memory accounts,
        uint256[] memory ids
    ) public view override returns (uint256[] memory) {
        require(accounts.length == ids.length, "ERC1155: accounts and ids length mismatch");
        
        uint256[] memory batchBalances = new uint256[](accounts.length);
        
        for (uint256 i = 0; i < accounts.length; ++i) {
            batchBalances[i] = balanceOf(accounts[i], ids[i]);
        }
        
        return batchBalances;
    }
    
    // ========== APPROVE ФУНКЦИИ ==========
    
    function setApprovalForAll(address operator, bool approved) public virtual override {
        require(msg.sender != operator, "ERC1155: setting approval for self");
        _operatorApprovals[msg.sender][operator] = approved;
        emit ApprovalForAll(msg.sender, operator, approved);
    }
    
    function isApprovedForAll(address account, address operator) public view override returns (bool) {
        return _operatorApprovals[account][operator];
    }
    
    // ========== TRANSFER ФУНКЦИИ ==========
    
    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) public virtual override {
        require(
            from == msg.sender || isApprovedForAll(from, msg.sender),
            "ERC1155: caller is not owner nor approved"
        );
        _safeTransferFrom(from, to, id, amount, data);
    }
    
    function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) public virtual override {
        require(
            from == msg.sender || isApprovedForAll(from, msg.sender),
            "ERC1155: caller is not owner nor approved"
        );
        _safeBatchTransferFrom(from, to, ids, amounts, data);
    }
    
    // ========== ВНУТРЕННИЕ ФУНКЦИИ ==========
    
    function _safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) internal virtual {
        require(to != address(0), "ERC1155: transfer to zero address");
        
        address operator = msg.sender;
        
        uint256 fromBalance = _balances[from][id];
        require(fromBalance >= amount, "ERC1155: insufficient balance");
        unchecked {
            _balances[from][id] = fromBalance - amount;
        }
        _balances[to][id] += amount;
        
        emit TransferSingle(operator, from, to, id, amount);
        
        _doSafeTransferAcceptanceCheck(operator, from, to, id, amount, data);
    }
    
    function _safeBatchTransferFrom(
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) internal virtual {
        require(ids.length == amounts.length, "ERC1155: ids and amounts length mismatch");
        require(to != address(0), "ERC1155: transfer to zero address");
        
        address operator = msg.sender;
        
        for (uint256 i = 0; i < ids.length; ++i) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];
            
            uint256 fromBalance = _balances[from][id];
            require(fromBalance >= amount, "ERC1155: insufficient balance");
            unchecked {
                _balances[from][id] = fromBalance - amount;
            }
            _balances[to][id] += amount;
        }
        
        emit TransferBatch(operator, from, to, ids, amounts);
        
        _doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, amounts, data);
    }
    
    function _mint(
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) internal virtual {
        require(to != address(0), "ERC1155: mint to zero address");
        
        address operator = msg.sender;
        
        _balances[to][id] += amount;
        emit TransferSingle(operator, address(0), to, id, amount);
        
        _doSafeTransferAcceptanceCheck(operator, address(0), to, id, amount, data);
    }
    
    function _mintBatch(
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) internal virtual {
        require(to != address(0), "ERC1155: mint to zero address");
        require(ids.length == amounts.length, "ERC1155: ids and amounts length mismatch");
        
        address operator = msg.sender;
        
        for (uint256 i = 0; i < ids.length; i++) {
            _balances[to][ids[i]] += amounts[i];
        }
        
        emit TransferBatch(operator, address(0), to, ids, amounts);
        
        _doSafeBatchTransferAcceptanceCheck(operator, address(0), to, ids, amounts, data);
    }
    
    function _burn(
        address from,
        uint256 id,
        uint256 amount
    ) internal virtual {
        require(from != address(0), "ERC1155: burn from zero address");
        
        address operator = msg.sender;
        
        uint256 fromBalance = _balances[from][id];
        require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");
        unchecked {
            _balances[from][id] = fromBalance - amount;
        }
        
        emit TransferSingle(operator, from, address(0), id, amount);
    }
    
    function _burnBatch(
        address from,
        uint256[] memory ids,
        uint256[] memory amounts
    ) internal virtual {
        require(from != address(0), "ERC1155: burn from zero address");
        require(ids.length == amounts.length, "ERC1155: ids and amounts length mismatch");
        
        address operator = msg.sender;
        
        for (uint256 i = 0; i < ids.length; i++) {
            uint256 id = ids[i];
            uint256 amount = amounts[i];
            
            uint256 fromBalance = _balances[from][id];
            require(fromBalance >= amount, "ERC1155: burn amount exceeds balance");
            unchecked {
                _balances[from][id] = fromBalance - amount;
            }
        }
        
        emit TransferBatch(operator, from, address(0), ids, amounts);
    }
    
    function _setURI(string memory newuri) internal virtual {
        _uri = newuri;
    }
    
    function _doSafeTransferAcceptanceCheck(
        address operator,
        address from,
        address to,
        uint256 id,
        uint256 amount,
        bytes memory data
    ) private {
        if (to.code.length > 0) {
            try IERC1155Receiver(to).onERC1155Received(operator, from, id, amount, data) returns (bytes4 response) {
                if (response != IERC1155Receiver.onERC1155Received.selector) {
                    revert("ERC1155: ERC1155Receiver rejected");
                }
            } catch Error(string memory reason) {
                revert(reason);
            } catch {
                revert("ERC1155: transfer to non ERC1155Receiver");
            }
        }
    }
    
    function _doSafeBatchTransferAcceptanceCheck(
        address operator,
        address from,
        address to,
        uint256[] memory ids,
        uint256[] memory amounts,
        bytes memory data
    ) private {
        if (to.code.length > 0) {
            try IERC1155Receiver(to).onERC1155BatchReceived(operator, from, ids, amounts, data) returns (bytes4 response) {
                if (response != IERC1155Receiver.onERC1155BatchReceived.selector) {
                    revert("ERC1155: ERC1155Receiver rejected");
                }
            } catch Error(string memory reason) {
                revert(reason);
            } catch {
                revert("ERC1155: transfer to non ERC1155Receiver");
            }
        }
    }
}

// ========== ИГРОВОЙ КОНТРАКТ С ERC-1155 ==========

contract GameItems is ERC1155 {
    address public owner;
    uint256 public nextTokenId;
    
    // Типы предметов
    enum ItemType { COMMON, RARE, EPIC, LEGENDARY }
    
    struct Item {
        string name;
        ItemType itemType;
        uint256 maxSupply;
        uint256 currentSupply;
        uint256 mintPrice;
        bool exists;
    }
    
    mapping(uint256 => Item) public items;
    mapping(uint256 => string) private _tokenURIs;
    
    event ItemCreated(uint256 indexed id, string name, ItemType itemType, uint256 maxSupply);
    event ItemMinted(address indexed to, uint256 indexed id, uint256 amount);
    
    constructor() ERC1155("https://game.example/api/item/{id}.json") {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    // ========== СОЗДАНИЕ ПРЕДМЕТОВ ==========
    
    function createItem(
        string memory name,
        ItemType itemType,
        uint256 maxSupply,
        uint256 mintPrice,
        string memory tokenURI
    ) public onlyOwner returns (uint256) {
        uint256 id = nextTokenId++;
        
        items[id] = Item({
            name: name,
            itemType: itemType,
            maxSupply: maxSupply,
            currentSupply: 0,
            mintPrice: mintPrice,
            exists: true
        });
        
        _tokenURIs[id] = tokenURI;
        
        emit ItemCreated(id, name, itemType, maxSupply);
        return id;
    }
    
    // ========== МИНТ ПРЕДМЕТОВ ==========
    
    function mint(address to, uint256 id, uint256 amount) public payable {
        Item storage item = items[id];
        require(item.exists, "Item does not exist");
        require(item.currentSupply + amount <= item.maxSupply, "Exceeds max supply");
        require(msg.value >= item.mintPrice * amount, "Insufficient payment");
        
        item.currentSupply += amount;
        _mint(to, id, amount, "");
        
        emit ItemMinted(to, id, amount);
        
        // Возврат излишка
        if (msg.value > item.mintPrice * amount) {
            payable(msg.sender).transfer(msg.value - item.mintPrice * amount);
        }
    }
    
    function mintBatch(
        address to,
        uint256[] memory ids,
        uint256[] memory amounts
    ) public payable {
        require(ids.length == amounts.length, "Length mismatch");
        
        uint256 totalCost = 0;
        
        for (uint256 i = 0; i < ids.length; i++) {
            Item storage item = items[ids[i]];
            require(item.exists, "Item does not exist");
            require(item.currentSupply + amounts[i] <= item.maxSupply, "Exceeds max supply");
            
            totalCost += item.mintPrice * amounts[i];
            item.currentSupply += amounts[i];
        }
        
        require(msg.value >= totalCost, "Insufficient payment");
        
        _mintBatch(to, ids, amounts, "");
        
        if (msg.value > totalCost) {
            payable(msg.sender).transfer(msg.value - totalCost);
        }
    }
    
    function ownerMint(address to, uint256 id, uint256 amount) public onlyOwner {
        Item storage item = items[id];
        require(item.exists, "Item does not exist");
        require(item.currentSupply + amount <= item.maxSupply, "Exceeds max supply");
        
        item.currentSupply += amount;
        _mint(to, id, amount, "");
        
        emit ItemMinted(to, id, amount);
    }
    
    // ========== СЖИГАНИЕ ==========
    
    function burn(address from, uint256 id, uint256 amount) public {
        require(
            from == msg.sender || isApprovedForAll(from, msg.sender),
            "Not approved to burn"
        );
        
        _burn(from, id, amount);
    }
    
    function burnBatch(address from, uint256[] memory ids, uint256[] memory amounts) public {
        require(
            from == msg.sender || isApprovedForAll(from, msg.sender),
            "Not approved to burn"
        );
        
        _burnBatch(from, ids, amounts);
    }
    
    // ========== URI ==========
    
    function uri(uint256 id) public view override returns (string memory) {
        require(items[id].exists, "Item does not exist");
        
        string memory tokenURI = _tokenURIs[id];
        if (bytes(tokenURI).length > 0) {
            return tokenURI;
        }
        
        return super.uri(id);
    }
    
    function setURI(uint256 id, string memory tokenURI) public onlyOwner {
        require(items[id].exists, "Item does not exist");
        _tokenURIs[id] = tokenURI;
        emit URI(tokenURI, id);
    }
    
    // ========== ИНФОРМАЦИЯ ==========
    
    function getItemInfo(uint256 id) public view returns (
        string memory name,
        ItemType itemType,
        uint256 maxSupply,
        uint256 currentSupply,
        uint256 mintPrice
    ) {
        Item memory item = items[id];
        require(item.exists, "Item does not exist");
        
        return (item.name, item.itemType, item.maxSupply, item.currentSupply, item.mintPrice);
    }
    
    function getUserItems(address user) public view returns (uint256[] memory, uint256[] memory) {
        uint256[] memory ids = new uint256[](nextTokenId);
        uint256[] memory amounts = new uint256[](nextTokenId);
        
        for (uint256 i = 0; i < nextTokenId; i++) {
            ids[i] = i;
            amounts[i] = balanceOf(user, i);
        }
        
        return (ids, amounts);
    }
    
    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}

// ========== КРАФТИНГ СИСТЕМА ==========

contract ItemCrafting {
    GameItems public gameItems;
    
    struct Recipe {
        uint256[] inputIds;
        uint256[] inputAmounts;
        uint256 outputId;
        uint256 outputAmount;
        bool exists;
    }
    
    uint256 public nextRecipeId;
    mapping(uint256 => Recipe) public recipes;
    
    event RecipeCreated(uint256 indexed recipeId, uint256 outputId, uint256 outputAmount);
    event ItemCrafted(address indexed crafter, uint256 indexed recipeId, uint256 outputId, uint256 amount);
    
    constructor(address _gameItems) {
        gameItems = GameItems(_gameItems);
    }
    
    // Создать рецепт (только owner игровых предметов)
    function createRecipe(
        uint256[] memory inputIds,
        uint256[] memory inputAmounts,
        uint256 outputId,
        uint256 outputAmount
    ) public returns (uint256) {
        require(msg.sender == gameItems.owner(), "Not authorized");
        require(inputIds.length == inputAmounts.length, "Length mismatch");
        require(inputIds.length > 0, "Empty recipe");
        
        uint256 recipeId = nextRecipeId++;
        
        recipes[recipeId] = Recipe({
            inputIds: inputIds,
            inputAmounts: inputAmounts,
            outputId: outputId,
            outputAmount: outputAmount,
            exists: true
        });
        
        emit RecipeCreated(recipeId, outputId, outputAmount);
        return recipeId;
    }
    
    // Крафт предмета
    function craft(uint256 recipeId) public {
        Recipe memory recipe = recipes[recipeId];
        require(recipe.exists, "Recipe does not exist");
        
        // Проверяем и сжигаем входные предметы
        for (uint256 i = 0; i < recipe.inputIds.length; i++) {
            require(
                gameItems.balanceOf(msg.sender, recipe.inputIds[i]) >= recipe.inputAmounts[i],
                "Insufficient items"
            );
        }
        
        gameItems.burnBatch(msg.sender, recipe.inputIds, recipe.inputAmounts);
        
        // Создаем выходной предмет
        gameItems.ownerMint(msg.sender, recipe.outputId, recipe.outputAmount);
        
        emit ItemCrafted(msg.sender, recipeId, recipe.outputId, recipe.outputAmount);
    }
    
    // Массовый крафт
    function craftMultiple(uint256 recipeId, uint256 times) public {
        Recipe memory recipe = recipes[recipeId];
        require(recipe.exists, "Recipe does not exist");
        require(times > 0, "Invalid times");
        
        // Проверяем достаточность предметов
        for (uint256 i = 0; i < recipe.inputIds.length; i++) {
            require(
                gameItems.balanceOf(msg.sender, recipe.inputIds[i]) >= recipe.inputAmounts[i] * times,
                "Insufficient items"
            );
        }
        
        // Подготовка массивов для batch операции
        uint256[] memory burnAmounts = new uint256[](recipe.inputIds.length);
        for (uint256 i = 0; i < recipe.inputIds.length; i++) {
            burnAmounts[i] = recipe.inputAmounts[i] * times;
        }
        
        gameItems.burnBatch(msg.sender, recipe.inputIds, burnAmounts);
        gameItems.ownerMint(msg.sender, recipe.outputId, recipe.outputAmount * times);
        
        emit ItemCrafted(msg.sender, recipeId, recipe.outputId, recipe.outputAmount * times);
    }
    
    function getRecipe(uint256 recipeId) public view returns (
        uint256[] memory inputIds,
        uint256[] memory inputAmounts,
        uint256 outputId,
        uint256 outputAmount
    ) {
        Recipe memory recipe = recipes[recipeId];
        require(recipe.exists, "Recipe does not exist");
        
        return (recipe.inputIds, recipe.inputAmounts, recipe.outputId, recipe.outputAmount);
    }
}

// ========== МАРКЕТПЛЕЙС ДЛЯ ERC-1155 ==========

contract MultiTokenMarketplace {
    GameItems public gameItems;
    
    struct Listing {
        address seller;
        uint256 tokenId;
        uint256 amount;
        uint256 pricePerItem;
        bool active;
    }
    
    uint256 public listingCounter;
    uint256 public feePercent = 25; // 2.5%
    address public feeRecipient;
    
    mapping(uint256 => Listing) public listings;
    
    event Listed(uint256 indexed listingId, address indexed seller, uint256 tokenId, uint256 amount, uint256 price);
    event Purchased(uint256 indexed listingId, address indexed buyer, uint256 amount);
    event Cancelled(uint256 indexed listingId);
    
    constructor(address _gameItems, address _feeRecipient) {
        gameItems = GameItems(_gameItems);
        feeRecipient = _feeRecipient;
    }
    
    function listItem(uint256 tokenId, uint256 amount, uint256 pricePerItem) public returns (uint256) {
        require(amount > 0, "Amount must be positive");
        require(pricePerItem > 0, "Price must be positive");
        require(gameItems.balanceOf(msg.sender, tokenId) >= amount, "Insufficient balance");
        require(
            gameItems.isApprovedForAll(msg.sender, address(this)),
            "Marketplace not approved"
        );
        
        uint256 listingId = listingCounter++;
        
        listings[listingId] = Listing({
            seller: msg.sender,
            tokenId: tokenId,
            amount: amount,
            pricePerItem: pricePerItem,
            active: true
        });
        
        emit Listed(listingId, msg.sender, tokenId, amount, pricePerItem);
        return listingId;
    }
    
    function buyItem(uint256 listingId, uint256 amount) public payable {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(amount > 0 && amount <= listing.amount, "Invalid amount");
        
        uint256 totalPrice = listing.pricePerItem * amount;
        require(msg.value >= totalPrice, "Insufficient payment");
        
        uint256 fee = (totalPrice * feePercent) / 1000;
        uint256 sellerAmount = totalPrice - fee;
        
        listing.amount -= amount;
        if (listing.amount == 0) {
            listing.active = false;
        }
        
        gameItems.safeTransferFrom(listing.seller, msg.sender, listing.tokenId, amount, "");
        
        payable(listing.seller).transfer(sellerAmount);
        payable(feeRecipient).transfer(fee);
        
        if (msg.value > totalPrice) {
            payable(msg.sender).transfer(msg.value - totalPrice);
        }
        
        emit Purchased(listingId, msg.sender, amount);
    }
    
    function cancelListing(uint256 listingId) public {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(listing.seller == msg.sender, "Not the seller");
        
        listing.active = false;
        emit Cancelled(listingId);
    }
    
    function updatePrice(uint256 listingId, uint256 newPricePerItem) public {
        Listing storage listing = listings[listingId];
        require(listing.active, "Listing not active");
        require(listing.seller == msg.sender, "Not the seller");
        require(newPricePerItem > 0, "Price must be positive");
        
        listing.pricePerItem = newPricePerItem;
    }
    
    function getActiveListings() public view returns (uint256[] memory) {
        uint256 activeCount = 0;
        for (uint256 i = 0; i < listingCounter; i++) {
            if (listings[i].active) activeCount++;
        }
        
        uint256[] memory activeListings = new uint256[](activeCount);
        uint256 currentIndex = 0;
        
        for (uint256 i = 0; i < listingCounter; i++) {
            if (listings[i].active) {
                activeListings[currentIndex] = i;
                currentIndex++;
            }
        }
        
        return activeListings;
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ ERC-1155:

ПРЕИМУЩЕСТВА:
- Один контракт для множества токенов
- Batch операции (экономия газа)
- Поддержка fungible и non-fungible токенов
- Эффективная передача множества токенов

ОСНОВНЫЕ ФУНКЦИИ:
- balanceOf: баланс одного токена
- balanceOfBatch: балансы множества токенов
- safeTransferFrom: перевод одного токена
- safeBatchTransferFrom: перевод множества токенов
- setApprovalForAll: разрешение для всех токенов

ИСПОЛЬЗОВАНИЕ:
- Игровые предметы (оружие, броня, ресурсы)
- Коллекционные карточки
- Крипто-арт коллекции
- Билеты и купоны

ПАТТЕРНЫ:
- Crafting system: создание из компонентов
- Multi-token marketplace: торговля разными токенами
- Batch operations: массовые операции
- Supply tracking: отслеживание эмиссии

БЕЗОПАСНОСТЬ:
- onERC1155Received: проверка получателя
- Batch validation: проверка массивов
- Supply limits: ограничение эмиссии
- Approval management: управление разрешениями

GAS ЭФФЕКТИВНОСТЬ:
- Batch mint: создание множества токенов за раз
- Batch transfer: перевод множества токенов
- Меньше storage операций
- Оптимизация для игр
*/
````
### Upgradeable контракты - Proxy паттерны
````
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ЗАЧЕМ НУЖНЫ UPGRADEABLE КОНТРАКТЫ ==========
/*
По умолчанию смарт-контракты ИММУТАБЕЛЬНЫ (неизменяемы).
Но иногда нужно:
- Исправить баги
- Добавить новую функциональность
- Оптимизировать код

РЕШЕНИЕ: Proxy Pattern
- Proxy контракт (не изменяется, хранит данные)
- Implementation контракт (логика, может обновляться)
- Все вызовы идут через delegatecall
*/

// ========== 1. TRANSPARENT PROXY PATTERN ==========

// Storage для Proxy (ВАЖНО: одинаковый layout!)
contract ProxyStorage {
    address internal _implementation;
    address internal _admin;
}

contract TransparentProxy is ProxyStorage {
    event Upgraded(address indexed implementation);
    event AdminChanged(address indexed previousAdmin, address indexed newAdmin);
    
    constructor(address implementation, address admin) {
        _implementation = implementation;
        _admin = admin;
        emit Upgraded(implementation);
    }
    
    modifier onlyAdmin() {
        require(msg.sender == _admin, "Not admin");
        _;
    }
    
    // Получить текущую имплементацию
    function implementation() external view returns (address) {
        return _implementation;
    }
    
    // Получить админа
    function admin() external view returns (address) {
        return _admin;
    }
    
    // Обновить имплементацию
    function upgradeTo(address newImplementation) external onlyAdmin {
        require(newImplementation != address(0), "Invalid address");
        _implementation = newImplementation;
        emit Upgraded(newImplementation);
    }
    
    // Сменить админа
    function changeAdmin(address newAdmin) external onlyAdmin {
        require(newAdmin != address(0), "Invalid address");
        address oldAdmin = _admin;
        _admin = newAdmin;
        emit AdminChanged(oldAdmin, newAdmin);
    }
    
    // Fallback - перенаправляет все вызовы на implementation
    fallback() external payable {
        address impl = _implementation;
        
        assembly {
            // Копируем calldata
            calldatacopy(0, 0, calldatasize())
            
            // Делаем delegatecall к implementation
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            
            // Копируем return data
            returndatacopy(0, 0, returndatasize())
            
            // Возвращаем результат
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
    
    receive() external payable {}
}

// ========== IMPLEMENTATION V1 ==========

contract TokenV1 is ProxyStorage {
    // ВАЖНО: переменные должны быть В ТОМ ЖЕ ПОРЯДКЕ!
    string public name;
    string public symbol;
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    // Инициализация вместо constructor (для upgradeable)
    function initialize(
        string memory _name,
        string memory _symbol,
        uint256 _initialSupply
    ) external {
        require(bytes(name).length == 0, "Already initialized");
        
        name = _name;
        symbol = _symbol;
        totalSupply = _initialSupply;
        balanceOf[msg.sender] = _initialSupply;
        
        emit Transfer(address(0), msg.sender, _initialSupply);
    }
    
    function transfer(address to, uint256 amount) external returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        
        balanceOf[msg.sender] -= amount;
        balanceOf[to] += amount;
        
        emit Transfer(msg.sender, to, amount);
        return true;
    }
    
    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }
    
    function transferFrom(address from, address to, uint256 amount) external returns (bool) {
        require(balanceOf[from] >= amount, "Insufficient balance");
        require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
        
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        allowance[from][msg.sender] -= amount;
        
        emit Transfer(from, to, amount);
        return true;
    }
}

// ========== IMPLEMENTATION V2 (UPGRADED) ==========

contract TokenV2 is ProxyStorage {
    // КРИТИЧНО: Сохраняем тот же порядок переменных!
    string public name;
    string public symbol;
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    // НОВЫЕ переменные добавляем ТОЛЬКО В КОНЕЦ!
    bool public paused;
    mapping(address => bool) public blacklist;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    event Paused(bool status);
    event Blacklisted(address indexed account, bool status);
    
    modifier whenNotPaused() {
        require(!paused, "Contract is paused");
        _;
    }
    
    modifier notBlacklisted(address account) {
        require(!blacklist[account], "Account blacklisted");
        _;
    }
    
    // Все старые функции
    function transfer(address to, uint256 amount) 
        external 
        whenNotPaused 
        notBlacklisted(msg.sender) 
        notBlacklisted(to) 
        returns (bool) 
    {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        
        balanceOf[msg.sender] -= amount;
        balanceOf[to] += amount;
        
        emit Transfer(msg.sender, to, amount);
        return true;
    }
    
    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }
    
    function transferFrom(address from, address to, uint256 amount) 
        external 
        whenNotPaused 
        notBlacklisted(from) 
        notBlacklisted(to) 
        returns (bool) 
    {
        require(balanceOf[from] >= amount, "Insufficient balance");
        require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
        
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        allowance[from][msg.sender] -= amount;
        
        emit Transfer(from, to, amount);
        return true;
    }
    
    // НОВЫЕ функции
    function pause() external {
        require(msg.sender == _admin, "Not admin");
        paused = true;
        emit Paused(true);
    }
    
    function unpause() external {
        require(msg.sender == _admin, "Not admin");
        paused = false;
        emit Paused(false);
    }
    
    function setBlacklist(address account, bool status) external {
        require(msg.sender == _admin, "Not admin");
        blacklist[account] = status;
        emit Blacklisted(account, status);
    }
    
    // НОВАЯ функция - burn
    function burn(uint256 amount) external returns (bool) {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        
        emit Transfer(msg.sender, address(0), amount);
        return true;
    }
}

// ========== 2. UUPS PROXY PATTERN ==========
// (Universal Upgradeable Proxy Standard)

contract UUPSProxy {
    address internal _implementation;
    address internal _admin;
    
    event Upgraded(address indexed implementation);
    
    constructor(address implementation, bytes memory data) {
        _implementation = implementation;
        _admin = msg.sender;
        
        if (data.length > 0) {
            (bool success,) = implementation.delegatecall(data);
            require(success, "Initialization failed");
        }
        
        emit Upgraded(implementation);
    }
    
    function implementation() external view returns (address) {
        return _implementation;
    }
    
    fallback() external payable {
        address impl = _implementation;
        
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
    
    receive() external payable {}
}

// UUPS Implementation (логика апгрейда внутри implementation)
abstract contract UUPSUpgradeable {
    address internal _implementation;
    address internal _admin;
    
    event Upgraded(address indexed newImplementation);
    
    modifier onlyAdmin() {
        require(msg.sender == _admin, "Not admin");
        _;
    }
    
    // Функция апгрейда находится В САМОЙ ИМПЛЕМЕНТАЦИИ
    function upgradeTo(address newImplementation) external onlyAdmin {
        require(newImplementation != address(0), "Invalid address");
        _authorizeUpgrade(newImplementation);
        _implementation = newImplementation;
        emit Upgraded(newImplementation);
    }
    
    // Переопределяется в наследниках для дополнительных проверок
    function _authorizeUpgrade(address newImplementation) internal virtual;
}

// ========== 3. BEACON PROXY PATTERN ==========
// (Для управления множеством proxy одновременно)

contract UpgradeableBeacon {
    address private _implementation;
    address public owner;
    
    event Upgraded(address indexed implementation);
    
    constructor(address implementation_) {
        _setImplementation(implementation_);
        owner = msg.sender;
    }
    
    function implementation() public view returns (address) {
        return _implementation;
    }
    
    function upgradeTo(address newImplementation) public {
        require(msg.sender == owner, "Not owner");
        _setImplementation(newImplementation);
    }
    
    function _setImplementation(address newImplementation) private {
        require(newImplementation.code.length > 0, "Not a contract");
        _implementation = newImplementation;
        emit Upgraded(newImplementation);
    }
}

contract BeaconProxy {
    address private immutable _beacon;
    
    constructor(address beacon, bytes memory data) {
        _beacon = beacon;
        
        if (data.length > 0) {
            address impl = UpgradeableBeacon(beacon).implementation();
            (bool success,) = impl.delegatecall(data);
            require(success, "Initialization failed");
        }
    }
    
    function _implementation() internal view returns (address) {
        return UpgradeableBeacon(_beacon).implementation();
    }
    
    fallback() external payable {
        address impl = _implementation();
        
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
    
    receive() external payable {}
}

// ========== 4. DIAMOND PATTERN (EIP-2535) ==========
// (Самый продвинутый, позволяет модульные апгрейды)

contract Diamond {
    struct FacetAddressAndPosition {
        address facetAddress;
        uint96 functionSelectorPosition;
    }
    
    struct FacetFunctionSelectors {
        bytes4[] functionSelectors;
        uint256 facetAddressPosition;
    }
    
    mapping(bytes4 => FacetAddressAndPosition) internal selectorToFacetAndPosition;
    mapping(address => FacetFunctionSelectors) internal facetFunctionSelectors;
    address[] internal facetAddresses;
    
    address internal contractOwner;
    
    event DiamondCut(address[] facetAddresses, bytes4[][] selectors);
    
    constructor(address _contractOwner) {
        contractOwner = _contractOwner;
    }
    
    // Добавление/замена/удаление функций
    enum FacetCutAction { Add, Replace, Remove }
    
    struct FacetCut {
        address facetAddress;
        FacetCutAction action;
        bytes4[] functionSelectors;
    }
    
    function diamondCut(FacetCut[] memory _diamondCut) external {
        require(msg.sender == contractOwner, "Not owner");
        
        for (uint256 facetIndex; facetIndex < _diamondCut.length; facetIndex++) {
            FacetCut memory facet = _diamondCut[facetIndex];
            
            if (facet.action == FacetCutAction.Add) {
                addFunctions(facet.facetAddress, facet.functionSelectors);
            } else if (facet.action == FacetCutAction.Replace) {
                replaceFunctions(facet.facetAddress, facet.functionSelectors);
            } else if (facet.action == FacetCutAction.Remove) {
                removeFunctions(facet.facetAddress, facet.functionSelectors);
            }
        }
    }
    
    function addFunctions(address facetAddress, bytes4[] memory selectors) internal {
        require(selectors.length > 0, "No selectors");
        uint96 selectorPosition = uint96(facetFunctionSelectors[facetAddress].functionSelectors.length);
        
        if (selectorPosition == 0) {
            facetAddresses.push(facetAddress);
        }
        
        for (uint256 i; i < selectors.length; i++) {
            bytes4 selector = selectors[i];
            address oldFacet = selectorToFacetAndPosition[selector].facetAddress;
            require(oldFacet == address(0), "Function already exists");
            
            facetFunctionSelectors[facetAddress].functionSelectors.push(selector);
            selectorToFacetAndPosition[selector].facetAddress = facetAddress;
            selectorToFacetAndPosition[selector].functionSelectorPosition = selectorPosition;
            selectorPosition++;
        }
    }
    
    function replaceFunctions(address facetAddress, bytes4[] memory selectors) internal {
        require(selectors.length > 0, "No selectors");
        
        for (uint256 i; i < selectors.length; i++) {
            bytes4 selector = selectors[i];
            address oldFacet = selectorToFacetAndPosition[selector].facetAddress;
            require(oldFacet != address(0), "Function doesn't exist");
            require(oldFacet != facetAddress, "Same function");
            
            selectorToFacetAndPosition[selector].facetAddress = facetAddress;
        }
    }
    
    function removeFunctions(address facetAddress, bytes4[] memory selectors) internal {
        require(selectors.length > 0, "No selectors");
        
        for (uint256 i; i < selectors.length; i++) {
            bytes4 selector = selectors[i];
            address oldFacet = selectorToFacetAndPosition[selector].facetAddress;
            require(oldFacet != address(0), "Function doesn't exist");
            
            delete selectorToFacetAndPosition[selector];
        }
    }
    
    fallback() external payable {
        address facet = selectorToFacetAndPosition[msg.sig].facetAddress;
        require(facet != address(0), "Function doesn't exist");
        
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
    
    receive() external payable {}
}

// ========== ПРИМЕР ИСПОЛЬЗОВАНИЯ ==========

contract ProxyFactory {
    event ProxyDeployed(address indexed proxy, address indexed implementation);
    
    // Деплой Transparent Proxy
    function deployTransparentProxy(
        address implementation,
        address admin,
        bytes memory initData
    ) external returns (address) {
        TransparentProxy proxy = new TransparentProxy(implementation, admin);
        
        if (initData.length > 0) {
            (bool success,) = address(proxy).call(initData);
            require(success, "Initialization failed");
        }
        
        emit ProxyDeployed(address(proxy), implementation);
        return address(proxy);
    }
    
    // Деплой UUPS Proxy
    function deployUUPSProxy(
        address implementation,
        bytes memory initData
    ) external returns (address) {
        UUPSProxy proxy = new UUPSProxy(implementation, initData);
        
        emit ProxyDeployed(address(proxy), implementation);
        return address(proxy);
    }
    
    // Деплой Beacon Proxy
    function deployBeaconProxy(
        address beacon,
        bytes memory initData
    ) external returns (address) {
        BeaconProxy proxy = new BeaconProxy(beacon, initData);
        
        address implementation = UpgradeableBeacon(beacon).implementation();
        emit ProxyDeployed(address(proxy), implementation);
        return address(proxy);
    }
}

// ========== ТЕСТИРОВАНИЕ АПГРЕЙДА ==========

contract UpgradeTest {
    TransparentProxy public proxy;
    address public admin;
    
    constructor() {
        admin = msg.sender;
        
        // 1. Деплоим Implementation V1
        TokenV1 implV1 = new TokenV1();
        
        // 2. Деплоим Proxy
        proxy = new TransparentProxy(address(implV1), admin);
        
        // 3. Инициализируем через proxy
        TokenV1(address(proxy)).initialize("MyToken", "MTK", 1000000);
    }
    
    function upgradeToV2() external {
        require(msg.sender == admin, "Not admin");
        
        // Деплоим Implementation V2
        TokenV2 implV2 = new TokenV2();
        
        // Апгрейдим proxy
        proxy.upgradeTo(address(implV2));
        
        // Теперь proxy использует V2!
        // Все данные сохранены (balances, totalSupply и т.д.)
        // Доступны новые функции (pause, burn и т.д.)
    }
    
    function testUpgrade() external view returns (
        string memory name,
        uint256 totalSupply,
        bool paused,
        uint256 myBalance
    ) {
        TokenV2 token = TokenV2(address(proxy));
        
        name = token.name();
        totalSupply = token.totalSupply();
        paused = token.paused();
        myBalance = token.balanceOf(msg.sender);
        
        return (name, totalSupply, paused, myBalance);
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ UPGRADEABLE КОНТРАКТОВ:

ПАТТЕРНЫ:
1. Transparent Proxy:
   - Простой и понятный
   - Админ не может вызывать функции implementation
   - Дороже по газу

2. UUPS (Universal Upgradeable Proxy Standard):
   - Логика апгрейда в implementation
   - Дешевле по газу
   - Требует осторожности (можно "сломать" апгрейд)

3. Beacon Proxy:
   - Один beacon контролирует множество proxy
   - Обновление всех proxy одновременно
   - Идеально для множества одинаковых контрактов

4. Diamond Pattern (EIP-2535):
   - Модульные апгрейды
   - Обход лимита размера контракта (24KB)
   - Самый сложный

КРИТИЧЕСКИЕ ПРАВИЛА:
✅ НИКОГДА не меняйте порядок переменных в storage
✅ Новые переменные ТОЛЬКО в конец
✅ НЕ используйте constructor (используйте initialize)
✅ Защищайте initialize от повторного вызова
✅ Тестируйте апгрейды на testnet
✅ Используйте storage gaps для будущих переменных

STORAGE LAYOUT:
contract MyContract {
    uint256 public a;      // slot 0
    uint256 public b;      // slot 1
    address public owner;  // slot 2
    
    // Storage gap для будущих переменных
    uint256[50] private __gap;
}

БЕЗОПАСНОСТЬ:
- Multisig для админа
- Timelock для апгрейдов
- Аудит перед апгрейдом
- Тестирование на форке mainnet
- Проверка storage layout

ИНСТРУМЕНТЫ:
- OpenZeppelin Upgrades Plugin
- Hardhat Upgrades
- Storage Layout проверка
- Foundry для тестирования

DELEGATECALL:
- Выполняет код другого контракта
- В контексте вызывающего (storage, msg.sender и т.д.)
- msg.sender сохраняется
- Основа всех proxy паттернов
*/
````
### Оракулы и внешние данные (Chainlink)
````
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== ЧТО ТАКОЕ ОРАКУЛЫ? ==========
/*
Блокчейн не может получать данные извне (off-chain):
- Цены криптовалют
- Погода
- Спортивные результаты
- API данные

ОРАКУЛ - это мост между блокчейном и внешним миром.

ПРОБЛЕМА: Доверие к централизованному оракулу
РЕШЕНИЕ: Децентрализованные оракулы (Chainlink)
*/

// ========== CHAINLINK PRICE FEEDS ==========

// Интерфейс Chainlink Price Feed
interface AggregatorV3Interface {
    function decimals() external view returns (uint8);
    function description() external view returns (string memory);
    function version() external view returns (uint256);
    
    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        );
}

// ========== ПРИМЕР 1: ПОЛУЧЕНИЕ ЦЕНЫ ETH/USD ==========

contract PriceConsumer {
    AggregatorV3Interface internal priceFeed;
    
    /**
     * Network: Ethereum Mainnet
     * Aggregator: ETH/USD
     * Address: 0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419
     */
    constructor(address _priceFeed) {
        priceFeed = AggregatorV3Interface(_priceFeed);
    }
    
    /**
     * Получить последнюю цену ETH в USD
     */
    function getLatestPrice() public view returns (int) {
        (
            /*uint80 roundID*/,
            int price,
            /*uint startedAt*/,
            /*uint timeStamp*/,
            /*uint80 answeredInRound*/
        ) = priceFeed.latestRoundData();
        
        return price; // Цена с 8 decimals (например, 200000000000 = $2000.00)
    }
    
    function getDecimals() public view returns (uint8) {
        return priceFeed.decimals();
    }
    
    function getPriceInUSD() public view returns (uint256) {
        int price = getLatestPrice();
        uint8 decimals = getDecimals();
        
        // Конвертируем в USD с 2 знаками после запятой
        return uint256(price) / (10 ** (decimals - 2));
    }
}

// ========== ПРИМЕР 2: LENDING С CHAINLINK ==========

contract ChainlinkLending {
    AggregatorV3Interface internal ethUsdPriceFeed;
    
    uint256 public constant COLLATERAL_RATIO = 150; // 150%
    uint256 public constant LIQUIDATION_THRESHOLD = 120; // 120%
    
    struct Loan {
        uint256 collateralETH;
        uint256 borrowedUSD;
        uint256 timestamp;
    }
    
    mapping(address => Loan) public loans;
    mapping(address => uint256) public deposits;
    
    uint256 public totalDeposits;
    uint256 public totalBorrowed;
    
    event Deposited(address indexed user, uint256 amount);
    event CollateralDeposited(address indexed user, uint256 amount);
    event Borrowed(address indexed user, uint256 amountUSD, uint256 collateralETH);
    event Repaid(address indexed user, uint256 amount);
    event Liquidated(address indexed borrower, address indexed liquidator, uint256 amount);
    
    constructor(address _ethUsdPriceFeed) {
        ethUsdPriceFeed = AggregatorV3Interface(_ethUsdPriceFeed);
    }
    
    // Депозит USD (в виде stablecoin, упрощено)
    function deposit() external payable {
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
        emit Deposited(msg.sender, msg.value);
    }
    
    // Взять кредит в USD под залог ETH
    function borrow(uint256 amountUSD) external payable {
        require(msg.value > 0, "Need collateral");
        require(amountUSD > 0, "Invalid amount");
        
        uint256 ethPriceUSD = getETHPrice();
        uint256 collateralValueUSD = (msg.value * ethPriceUSD) / 1e18;
        
        // Проверка достаточности залога
        uint256 maxBorrow = (collateralValueUSD * 100) / COLLATERAL_RATIO;
        require(amountUSD <= maxBorrow, "Insufficient collateral");
        
        // Проверка ликвидности
        require(totalDeposits >= totalBorrowed + amountUSD, "Insufficient liquidity");
        
        // Обновление кредита
        loans[msg.sender].collateralETH += msg.value;
        loans[msg.sender].borrowedUSD += amountUSD;
        loans[msg.sender].timestamp = block.timestamp;
        totalBorrowed += amountUSD;
        
        // Отправка USD (упрощено, в реальности это stablecoin)
        payable(msg.sender).transfer(amountUSD);
        
        emit Borrowed(msg.sender, amountUSD, msg.value);
    }
    
    // Погасить кредит
    function repay() external payable {
        Loan storage loan = loans[msg.sender];
        require(loan.borrowedUSD > 0, "No active loan");
        require(msg.value >= loan.borrowedUSD, "Insufficient repayment");
        
        uint256 collateral = loan.collateralETH;
        uint256 borrowed = loan.borrowedUSD;
        
        totalBorrowed -= borrowed;
        delete loans[msg.sender];
        
        // Возврат залога
        payable(msg.sender).transfer(collateral);
        
        // Возврат излишка
        if (msg.value > borrowed) {
            payable(msg.sender).transfer(msg.value - borrowed);
        }
        
        emit Repaid(msg.sender, borrowed);
    }
    
    // Ликвидация undercollateralized кредита
    function liquidate(address borrower) external payable {
        Loan storage loan = loans[borrower];
        require(loan.borrowedUSD > 0, "No active loan");
        
        uint256 healthFactor = getLoanHealth(borrower);
        require(healthFactor < LIQUIDATION_THRESHOLD, "Loan is healthy");
        
        require(msg.value >= loan.borrowedUSD, "Insufficient payment");
        
        uint256 collateral = loan.collateralETH;
        uint256 borrowed = loan.borrowedUSD;
        
        totalBorrowed -= borrowed;
        delete loans[borrower];
        
        // Ликвидатор получает залог
        payable(msg.sender).transfer(collateral);
        
        // Возврат излишка
        if (msg.value > borrowed) {
            payable(msg.sender).transfer(msg.value - borrowed);
        }
        
        emit Liquidated(borrower, msg.sender, borrowed);
    }
    
    // Получить цену ETH в USD
    function getETHPrice() public view returns (uint256) {
        (, int price,,,) = ethUsdPriceFeed.latestRoundData();
        return uint256(price) * 1e10; // Приводим к 18 decimals
    }
    
    // Здоровье кредита (collateral value / borrowed * 100)
    function getLoanHealth(address borrower) public view returns (uint256) {
        Loan memory loan = loans[borrower];
        if (loan.borrowedUSD == 0) return 0;
        
        uint256 ethPriceUSD = getETHPrice();
        uint256 collateralValueUSD = (loan.collateralETH * ethPriceUSD) / 1e18;
        
        return (collateralValueUSD * 100) / loan.borrowedUSD;
    }
}

// ========== CHAINLINK VRF (VERIFIABLE RANDOM FUNCTION) ==========

// Интерфейс Chainlink VRF
interface VRFCoordinatorV2Interface {
    function requestRandomWords(
        bytes32 keyHash,
        uint64 subId,
        uint16 minimumRequestConfirmations,
        uint32 callbackGasLimit,
        uint32 numWords
    ) external returns (uint256 requestId);
}

abstract contract VRFConsumerBaseV2 {
    error OnlyCoordinatorCanFulfill(address have, address want);
    address private immutable vrfCoordinator;
    
    constructor(address _vrfCoordinator) {
        vrfCoordinator = _vrfCoordinator;
    }
    
    function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal virtual;
    
    function rawFulfillRandomWords(uint256 requestId, uint256[] memory randomWords) external {
        if (msg.sender != vrfCoordinator) {
            revert OnlyCoordinatorCanFulfill(msg.sender, vrfCoordinator);
        }
        fulfillRandomWords(requestId, randomWords);
    }
}

// ========== ЛОТЕРЕЯ С CHAINLINK VRF ==========

contract RandomLottery is VRFConsumerBaseV2 {
    VRFCoordinatorV2Interface COORDINATOR;
    
    uint64 s_subscriptionId;
    bytes32 keyHash;
    uint32 callbackGasLimit = 100000;
    uint16 requestConfirmations = 3;
    uint32 numWords = 1;
    
    address public owner;
    address[] public players;
    address public recentWinner;
    
    enum LotteryState { OPEN, CALCULATING }
    LotteryState public lotteryState;
    
    uint256 public ticketPrice = 0.01 ether;
    
    event LotteryEntered(address indexed player);
    event WinnerRequested(uint256 indexed requestId);
    event WinnerPicked(address indexed winner, uint256 amount);
    
    constructor(
        uint64 subscriptionId,
        address vrfCoordinator,
        bytes32 _keyHash
    ) VRFConsumerBaseV2(vrfCoordinator) {
        COORDINATOR = VRFCoordinatorV2Interface(vrfCoordinator);
        s_subscriptionId = subscriptionId;
        keyHash = _keyHash;
        owner = msg.sender;
        lotteryState = LotteryState.OPEN;
    }
    
    function enterLottery() public payable {
        require(lotteryState == LotteryState.OPEN, "Lottery not open");
        require(msg.value >= ticketPrice, "Not enough ETH");
        
        players.push(msg.sender);
        emit LotteryEntered(msg.sender);
    }
    
    function requestRandomWinner() public {
        require(msg.sender == owner, "Not owner");
        require(lotteryState == LotteryState.OPEN, "Already calculating");
        require(players.length > 0, "No players");
        
        lotteryState = LotteryState.CALCULATING;
        
        uint256 requestId = COORDINATOR.requestRandomWords(
            keyHash,
            s_subscriptionId,
            requestConfirmations,
            callbackGasLimit,
            numWords
        );
        
        emit WinnerRequested(requestId);
    }
    
    function fulfillRandomWords(
        uint256, /* requestId */
        uint256[] memory randomWords
    ) internal override {
        uint256 indexOfWinner = randomWords[0] % players.length;
        address winner = players[indexOfWinner];
        recentWinner = winner;
        
        uint256 prize = address(this).balance;
        
        // Сброс лотереи
        players = new address[](0);
        lotteryState = LotteryState.OPEN;
        
        // Отправка приза
        (bool success, ) = winner.call{value: prize}("");
        require(success, "Transfer failed");
        
        emit WinnerPicked(winner, prize);
    }
    
    function getPlayers() public view returns (address[] memory) {
        return players;
    }
    
    function getPlayerCount() public view returns (uint256) {
        return players.length;
    }
}

// ========== CHAINLINK AUTOMATION (KEEPERS) ==========

// Интерфейс для Automation
interface AutomationCompatibleInterface {
    function checkUpkeep(bytes calldata checkData) external returns (bool upkeepNeeded, bytes memory performData);
    function performUpkeep(bytes calldata performData) external;
}

// ========== AUTO-COMPOUND STAKING ==========

contract AutoCompoundStaking is AutomationCompatibleInterface {
    uint256 public totalStaked;
    uint256 public rewardRate = 100; // 100 токенов в секунду
    uint256 public lastUpdateTime;
    uint256 public rewardPerTokenStored;
    
    uint256 public constant COMPOUND_INTERVAL = 1 days;
    uint256 public lastCompoundTime;
    
    mapping(address => uint256) public balances;
    mapping(address => uint256) public userRewardPerTokenPaid;
    mapping(address => uint256) public rewards;
    
    event Staked(address indexed user, uint256 amount);
    event Compounded(uint256 totalRewards);
    
    constructor() {
        lastUpdateTime = block.timestamp;
        lastCompoundTime = block.timestamp;
    }
    
    modifier updateReward(address account) {
        rewardPerTokenStored = rewardPerToken();
        lastUpdateTime = block.timestamp;
        
        if (account != address(0)) {
            rewards[account] = earned(account);
            userRewardPerTokenPaid[account] = rewardPerTokenStored;
        }
        _;
    }
    
    function stake(uint256 amount) external payable updateReward(msg.sender) {
        require(amount > 0, "Cannot stake 0");
        
        totalStaked += amount;
        balances[msg.sender] += amount;
        
        emit Staked(msg.sender, amount);
    }
    
    function rewardPerToken() public view returns (uint256) {
        if (totalStaked == 0) {
            return rewardPerTokenStored;
        }
        return rewardPerTokenStored + 
            (((block.timestamp - lastUpdateTime) * rewardRate * 1e18) / totalStaked);
    }
    
    function earned(address account) public view returns (uint256) {
        return ((balances[account] * 
            (rewardPerToken() - userRewardPerTokenPaid[account])) / 1e18) + 
            rewards[account];
    }
    
    // Chainlink Automation вызывает эту функцию для проверки
    function checkUpkeep(bytes calldata /* checkData */) 
        external 
        view 
        override 
        returns (bool upkeepNeeded, bytes memory /* performData */) 
    {
        upkeepNeeded = (block.timestamp - lastCompoundTime) > COMPOUND_INTERVAL;
        // performData пустой в нашем случае
    }
    
    // Chainlink Automation вызывает эту функцию автоматически
    function performUpkeep(bytes calldata /* performData */) external override {
        if ((block.timestamp - lastCompoundTime) > COMPOUND_INTERVAL) {
            lastCompoundTime = block.timestamp;
            compound();
        }
    }
    
    // Реинвестирование наград
    function compound() internal {
        uint256 totalRewardsToCompound = 0;
        
        // В реальности нужно итерировать по всем stakers
        // Здесь упрощенная версия - автоматический compound для контракта
        
        rewardPerTokenStored = rewardPerToken();
        lastUpdateTime = block.timestamp;
        
        emit Compounded(totalRewardsToCompound);
    }
}

// ========== CHAINLINK ANY API (CUSTOM API CALLS) ==========

interface ChainlinkRequestInterface {
    function sendChainlinkRequest(
        bytes32 _requestId,
        uint256 _payment,
        address _callbackAddress,
        bytes4 _callbackFunctionSignature,
        uint256 _nonce,
        uint256 _dataVersion,
        bytes calldata _data
    ) external;
}

// Chainlink Client для API запросов
abstract contract ChainlinkClient {
    uint256 private constant LINK_DIVISIBILITY = 10**18;
    uint256 private constant AMOUNT_OVERRIDE = 0;
    address private s_linkToken;
    address private s_oracle;
    
    struct Request {
        bytes32 id;
        address callbackAddress;
        bytes4 callbackFunctionId;
        uint256 nonce;
        bytes32 dataVersion;
    }
    
    constructor(address linkToken, address oracle) {
        s_linkToken = linkToken;
        s_oracle = oracle;
    }
}

// ========== ПОГОДНОЕ СТРАХОВАНИЕ С CHAINLINK ==========

contract WeatherInsurance {
    AggregatorV3Interface internal temperatureFeed;
    
    struct Policy {
        address holder;
        uint256 premium;
        uint256 payout;
        int256 triggerTemperature;
        uint256 expiryDate;
        bool active;
        bool claimed;
    }
    
    uint256 public policyCounter;
    mapping(uint256 => Policy) public policies;
    mapping(address => uint256[]) public userPolicies;
    
    event PolicyCreated(uint256 indexed policyId, address indexed holder, int256 triggerTemp);
    event PayoutClaimed(uint256 indexed policyId, address indexed holder, uint256 amount);
    
    constructor(address _temperatureFeed) {
        temperatureFeed = AggregatorV3Interface(_temperatureFeed);
    }
    
    // Купить полис
    function buyPolicy(
        int256 triggerTemperature,
        uint256 duration,
        uint256 payout
    ) external payable returns (uint256) {
        require(msg.value > 0, "Premium required");
        require(payout > msg.value, "Payout must exceed premium");
        
        uint256 policyId = policyCounter++;
        
        policies[policyId] = Policy({
            holder: msg.sender,
            premium: msg.value,
            payout: payout,
            triggerTemperature: triggerTemperature,
            expiryDate: block.timestamp + duration,
            active: true,
            claimed: false
        });
        
        userPolicies[msg.sender].push(policyId);
        
        emit PolicyCreated(policyId, msg.sender, triggerTemperature);
        return policyId;
    }
    
    // Получить выплату если температура ниже триггера
    function claimPayout(uint256 policyId) external {
        Policy storage policy = policies[policyId];
        
        require(policy.active, "Policy not active");
        require(!policy.claimed, "Already claimed");
        require(policy.holder == msg.sender, "Not policy holder");
        require(block.timestamp <= policy.expiryDate, "Policy expired");
        
        // Получаем текущую температуру
        int256 currentTemp = getCurrentTemperature();
        
        require(currentTemp <= policy.triggerTemperature, "Temperature not low enough");
        
        policy.claimed = true;
        policy.active = false;
        
        payable(msg.sender).transfer(policy.payout);
        
        emit PayoutClaimed(policyId, msg.sender, policy.payout);
    }
    
    function getCurrentTemperature() public view returns (int256) {
        (, int temperature,,,) = temperatureFeed.latestRoundData();
        return temperature;
    }
    
    function getUserPolicies(address user) external view returns (uint256[] memory) {
        return userPolicies[user];
    }
    
    function getPolicyInfo(uint256 policyId) external view returns (
        address holder,
        uint256 premium,
        uint256 payout,
        int256 triggerTemperature,
        uint256 expiryDate,
        bool active,
        bool claimed
    ) {
        Policy memory policy = policies[policyId];
        return (
            policy.holder,
            policy.premium,
            policy.payout,
            policy.triggerTemperature,
            policy.expiryDate,
            policy.active,
            policy.claimed
        );
    }
    
    // Владелец может пополнить контракт для выплат
    receive() external payable {}
}

// ========== СПОРТИВНЫЕ СТАВКИ С ОРАКУЛОМ ==========

contract SportsBetting {
    address public owner;
    address public oracle; // Адрес оракула, который предоставляет результаты
    
    enum MatchResult { NOT_STARTED, TEAM_A_WIN, TEAM_B_WIN, DRAW }
    
    struct Match {
        string teamA;
        string teamB;
        uint256 startTime;
        MatchResult result;
        bool finalized;
        uint256 totalBetsA;
        uint256 totalBetsB;
        uint256 totalBetsDraw;
    }
    
    struct Bet {
        address bettor;
        uint256 amount;
        MatchResult prediction;
        bool claimed;
    }
    
    uint256 public matchCounter;
    mapping(uint256 => Match) public matches;
    mapping(uint256 => Bet[]) public matchBets;
    mapping(address => uint256[]) public userBets;
    
    uint256 public constant FEE_PERCENT = 2; // 2% комиссия
    
    event MatchCreated(uint256 indexed matchId, string teamA, string teamB, uint256 startTime);
    event BetPlaced(uint256 indexed matchId, address indexed bettor, MatchResult prediction, uint256 amount);
    event MatchFinalized(uint256 indexed matchId, MatchResult result);
    event WinningsClaimed(uint256 indexed matchId, address indexed bettor, uint256 amount);
    
    constructor(address _oracle) {
        owner = msg.sender;
        oracle = _oracle;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    modifier onlyOracle() {
        require(msg.sender == oracle, "Not oracle");
        _;
    }
    
    // Создать матч
    function createMatch(
        string memory teamA,
        string memory teamB,
        uint256 startTime
    ) external onlyOwner returns (uint256) {
        require(startTime > block.timestamp, "Invalid start time");
        
        uint256 matchId = matchCounter++;
        
        matches[matchId] = Match({
            teamA: teamA,
            teamB: teamB,
            startTime: startTime,
            result: MatchResult.NOT_STARTED,
            finalized: false,
            totalBetsA: 0,
            totalBetsB: 0,
            totalBetsDraw: 0
        });
        
        emit MatchCreated(matchId, teamA, teamB, startTime);
        return matchId;
    }
    
    // Сделать ставку
    function placeBet(uint256 matchId, MatchResult prediction) external payable {
        require(msg.value > 0, "Bet amount required");
        Match storage matchData = matches[matchId];
        require(!matchData.finalized, "Match already finalized");
        require(block.timestamp < matchData.startTime, "Betting closed");
        require(prediction != MatchResult.NOT_STARTED, "Invalid prediction");
        
        // Обновляем тотал ставок
        if (prediction == MatchResult.TEAM_A_WIN) {
            matchData.totalBetsA += msg.value;
        } else if (prediction == MatchResult.TEAM_B_WIN) {
            matchData.totalBetsB += msg.value;
        } else {
            matchData.totalBetsDraw += msg.value;
        }
        
        // Сохраняем ставку
        matchBets[matchId].push(Bet({
            bettor: msg.sender,
            amount: msg.value,
            prediction: prediction,
            claimed: false
        }));
        
        userBets[msg.sender].push(matchId);
        
        emit BetPlaced(matchId, msg.sender, prediction, msg.value);
    }
    
    // Оракул финализирует результат
    function finalizeMatch(uint256 matchId, MatchResult result) external onlyOracle {
        Match storage matchData = matches[matchId];
        require(!matchData.finalized, "Already finalized");
        require(block.timestamp >= matchData.startTime, "Match not started");
        require(result != MatchResult.NOT_STARTED, "Invalid result");
        
        matchData.result = result;
        matchData.finalized = true;
        
        emit MatchFinalized(matchId, result);
    }
    
    // Забрать выигрыш
    function claimWinnings(uint256 matchId, uint256 betIndex) external {
        Match memory matchData = matches[matchId];
        require(matchData.finalized, "Match not finalized");
        
        Bet storage bet = matchBets[matchId][betIndex];
        require(bet.bettor == msg.sender, "Not your bet");
        require(!bet.claimed, "Already claimed");
        require(bet.prediction == matchData.result, "Bet lost");
        
        bet.claimed = true;
        
        // Расчет выигрыша
        uint256 totalPool = matchData.totalBetsA + matchData.totalBetsB + matchData.totalBetsDraw;
        uint256 winningPool;
        
        if (matchData.result == MatchResult.TEAM_A_WIN) {
            winningPool = matchData.totalBetsA;
        } else if (matchData.result == MatchResult.TEAM_B_WIN) {
            winningPool = matchData.totalBetsB;
        } else {
            winningPool = matchData.totalBetsDraw;
        }
        
        // Расчет: (ставка / выигрышный пул) * общий пул * (1 - комиссия)
        uint256 winnings = (bet.amount * totalPool * (100 - FEE_PERCENT)) / (winningPool * 100);
        
        payable(msg.sender).transfer(winnings);
        
        emit WinningsClaimed(matchId, msg.sender, winnings);
    }
    
    function getMatchBets(uint256 matchId) external view returns (Bet[] memory) {
        return matchBets[matchId];
    }
    
    function getUserBets(address user) external view returns (uint256[] memory) {
        return userBets[user];
    }
    
    function getMatchOdds(uint256 matchId) external view returns (
        uint256 oddsA,
        uint256 oddsB,
        uint256 oddsDraw
    ) {
        Match memory matchData = matches[matchId];
        uint256 totalPool = matchData.totalBetsA + matchData.totalBetsB + matchData.totalBetsDraw;
        
        if (totalPool == 0) return (0, 0, 0);
        
        // Odds = (Total Pool / Specific Bet Pool)
        oddsA = matchData.totalBetsA > 0 ? (totalPool * 100) / matchData.totalBetsA : 0;
        oddsB = matchData.totalBetsB > 0 ? (totalPool * 100) / matchData.totalBetsB : 0;
        oddsDraw = matchData.totalBetsDraw > 0 ? (totalPool * 100) / matchData.totalBetsDraw : 0;
    }
    
    function withdrawFees() external onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}

// ========== DECENTRALIZED ORACLE (ПРОСТАЯ ВЕРСИЯ) ==========

contract SimpleOracle {
    address public owner;
    
    struct DataPoint {
        int256 value;
        uint256 timestamp;
        bool exists;
    }
    
    mapping(bytes32 => DataPoint) public data;
    mapping(address => bool) public reporters;
    
    event DataReported(bytes32 indexed key, int256 value, uint256 timestamp);
    event ReporterAdded(address indexed reporter);
    event ReporterRemoved(address indexed reporter);
    
    constructor() {
        owner = msg.sender;
        reporters[msg.sender] = true;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    modifier onlyReporter() {
        require(reporters[msg.sender], "Not a reporter");
        _;
    }
    
    function addReporter(address reporter) external onlyOwner {
        reporters[reporter] = true;
        emit ReporterAdded(reporter);
    }
    
    function removeReporter(address reporter) external onlyOwner {
        reporters[reporter] = false;
        emit ReporterRemoved(reporter);
    }
    
    function reportData(bytes32 key, int256 value) external onlyReporter {
        data[key] = DataPoint({
            value: value,
            timestamp: block.timestamp,
            exists: true
        });
        
        emit DataReported(key, value, block.timestamp);
    }
    
    function getData(bytes32 key) external view returns (int256, uint256, bool) {
        DataPoint memory dp = data[key];
        return (dp.value, dp.timestamp, dp.exists);
    }
    
    function getLatestValue(bytes32 key) external view returns (int256) {
        require(data[key].exists, "No data for key");
        return data[key].value;
    }
}

/*
КЛЮЧЕВЫЕ КОНЦЕПЦИИ ОРАКУЛОВ:

ТИПЫ ОРАКУЛОВ:
1. Price Feeds:
   - Цены криптовалют (ETH/USD, BTC/USD)
   - Цены активов (золото, нефть)
   - Обменные курсы

2. VRF (Verifiable Random Function):
   - Честная генерация случайных чисел
   - Лотереи, игры, NFT
   - Защита от манипуляций

3. Automation (Keepers):
   - Автоматическое выполнение функций
   - Compound interest
   - Ликвидации
   - Автоматические выплаты

4. Any API:
   - Погода
   - Спортивные результаты
   - IoT данные
   - Web2 API интеграция

CHAINLINK КОМПОНЕНТЫ:
- Node Operators: запускают оракул ноды
- Data Providers: предоставляют данные
- Aggregator: собирает данные от множества источников
- Consumer: ваш контракт, использующий данные

БЕЗОПАСНОСТЬ:
✅ Используйте децентрализованные оракулы
✅ Проверяйте свежесть данных (timestamp)
✅ Множественные источники данных
✅ Slashing для недобросовестных reporters
✅ Проверка отклонений (deviation threshold)

PRICE FEEDS:
- Обновляются каждые N секунд или при отклонении X%
- Heartbeat: минимальная частота обновления
- Deviation: порог изменения цены
- Aggregation: медиана от множества источников

VRF ИСПОЛЬЗОВАНИЕ:
1. Request randomness
2. Pay LINK fee
3. Wait for callback
4. Receive verifiable random number

AUTOMATION:
1. checkUpkeep: проверка условий
2. performUpkeep: выполнение действия
3. Автоматический вызов по расписанию

СТОИМОСТЬ:
- Price Feeds: бесплатно для чтения
- VRF: платить LINK за запрос
- Automation: платить LINK за автоматизацию
- Any API: платить LINK за запросы

ЛУЧШИЕ ПРАКТИКИ:
- Кешируйте данные оракулов
- Устанавливайте максимальный возраст данных
- Обрабатывайте ошибки оракула
- Используйте fallback источники
- Тестируйте с mock оракулами
*/
````
### Gas оптимизация - Техники экономии газа
````
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ========== GAS OPTIMIZATION - ТЕХНИКИ ЭКОНОМИИ ==========

/*
СТОИМОСТЬ ОПЕРАЦИЙ (примерно):
- SSTORE (запись в storage): 20,000 газа (первая запись), 5,000 (изменение)
- SLOAD (чтение из storage): 2,100 газа
- Memory операции: ~3 газа
- Арифметические операции: 3-5 газа
- CALL: 2,600 газа + передача газа
*/

// ========== 1. УПАКОВКА ПЕРЕМЕННЫХ В STORAGE ==========

contract BadStoragePacking {
    // Каждая переменная занимает 32 байта (1 слот)
    uint8 a;    // Слот 0: занимает 32 байта, использует 1 байт
    uint256 b;  // Слот 1: занимает 32 байта
    uint8 c;    // Слот 2: занимает 32 байта, использует 1 байт
    uint256 d;  // Слот 3: занимает 32 байта
    
    // Итого: 4 слота = 4 SSTORE операции
}

contract GoodStoragePacking {
    // Упаковываем маленькие переменные вместе
    uint8 a;    // Слот 0 (байты 0-0)
    uint8 c;    // Слот 0 (байты 1-1)
    uint256 b;  // Слот 1
    uint256 d;  // Слот 2
    
    // Итого: 3 слота = 3 SSTORE операции
    // Экономия: ~20,000 газа при инициализации
}

contract OptimalStoragePacking {
    // Максимальная оптимизация
    uint128 a;  // Слот 0 (байты 0-15)
    uint128 b;  // Слот 0 (байты 16-31)
    uint64 c;   // Слот 1 (байты 0-7)
    uint64 d;   // Слот 1 (байты 8-15)
    uint64 e;   // Слот 1 (байты 16-23)
    uint64 f;   // Слот 1 (байты 24-31)
    
    // 6 переменных в 2 слотах!
}

// ========== 2. ИСПОЛЬЗОВАНИЕ MEMORY VS STORAGE ==========

contract MemoryVsStorage {
    struct User {
        string name;
        uint256 balance;
        bool active;
    }
    
    mapping(address => User) public users;
    
    // ПЛОХО: многократное чтение из storage
    function badUpdate(address user) external {
        users[user].balance += 100;    // SLOAD + SSTORE
        users[user].balance += 200;    // SLOAD + SSTORE
        users[user].balance += 300;    // SLOAD + SSTORE
        // 3 SLOAD + 3 SSTORE
    }
    
    // ХОРОШО: кеширование в memory
    function goodUpdate(address user) external {
        uint256 balance = users[user].balance;  // 1 SLOAD
        balance += 100;
        balance += 200;
        balance += 300;
        users[user].balance = balance;          // 1 SSTORE
        // 1 SLOAD + 1 SSTORE - экономия ~10,000 газа
    }
    
    // ПЛОХО: storage указатель
    function badModifyUser(address user) external {
        User storage u = users[user];
        u.balance = 1000;     // SSTORE
        u.active = true;      // SSTORE
        // Множественные SSTORE
    }
    
    // ХОРОШО: пакетное обновление
    function goodModifyUser(address user) external {
        users[user] = User("Alice", 1000, true);
        // Одно пакетное обновление
    }
}

// ========== 3. ИСПОЛЬЗОВАНИЕ CALLDATA ДЛЯ EXTERNAL ФУНКЦИЙ ==========

contract CalldataOptimization {
    // ПЛОХО: копирование в memory
    function badProcess(uint256[] memory data) external pure returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < data.length; i++) {
            sum += data[i];
        }
        return sum;
    }
    
    // ХОРОШО: использование calldata (дешевле)
    function goodProcess(uint256[] calldata data) external pure returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < data.length; i++) {
            sum += data[i];
        }
        return sum;
    }
    
    // Calldata: read-only, не копируется, дешевле
    // Memory: можно изменять, копируется, дороже
}

// ========== 4. ОПТИМИЗАЦИЯ ЦИКЛОВ ==========

contract LoopOptimization {
    uint256[] public items;
    
    // ПЛОХО: length вычисляется каждую итерацию
    function badLoop() external view returns (uint256) {
        uint256 sum = 0;
        for (uint256 i = 0; i < items.length; i++) {
            sum += items[i];
        }
        return sum;
    }
    
    // ХОРОШО: кешируем length
    function goodLoop() external view returns (uint256) {
        uint256 sum = 0;
        uint256 length = items.length;  // Кеш
        for (uint256 i = 0; i < length; i++) {
            sum += items[i];
        }
        return sum;
    }
    
    // ЛУЧШЕ: unchecked для экономии на overflow проверках
    function betterLoop() external view returns (uint256) {
        uint256 sum = 0;
        uint256 length = items.length;
        for (uint256 i = 0; i < length;) {
            sum += items[i];
            unchecked { ++i; }  // Экономия ~100 газа на итерацию
        }
        return sum;
    }
    
    // ОПТИМАЛЬНО: ++i вместо i++
    function optimalLoop() external view returns (uint256) {
        uint256 sum = 0;
        uint256 length = items.length;
        for (uint256 i = 0; i < length;) {
            sum += items[i];
            unchecked { ++i; }  // ++i дешевле чем i++
        }
        return sum;
    }
}

// ========== 5. ИСПОЛЬЗОВАНИЕ UINT256 ВМЕСТО МЕНЬШИХ ТИПОВ ==========

contract UintOptimization {
    // ПЛОХО: uint8 требует дополнительных операций
    function badSum(uint8 a, uint8 b) external pure returns (uint8) {
        return a + b;
        // EVM работает с 256-битными словами,
        // uint8 требует преобразований
    }
    
    // ХОРОШО: uint256 - нативный тип EVM
    function goodSum(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
        // Нет дополнительных преобразований
    }
    
    // ИСКЛЮЧЕНИЕ: упаковка в storage (см. выше)
}

// ========== 6. SHORT-CIRCUIT EVALUATION ==========

contract ShortCircuitOptimization {
    bool public expensiveCheckPassed;
    
    function expensiveCheck() internal view returns (bool) {
        // Дорогая операция
        return expensiveCheckPassed;
    }
    
    // ПЛОХО: дорогая проверка всегда выполняется
    function badValidation(bool cheap) external view returns (bool) {
        return expensiveCheck() && cheap;
    }
    
    // ХОРОШО: дешевая проверка первой
    function goodValidation(bool cheap) external view returns (bool) {
        return cheap && expensiveCheck();
        // Если cheap == false, expensiveCheck() не вызывается
    }
    
    // Порядок проверок: от дешевых к дорогим
    function optimalValidation(
        bool veryCheap,
        bool cheap,
        bool moderate
    ) external view returns (bool) {
        return veryCheap && cheap && moderate && expensiveCheck();
    }
}

// ========== 7. КОНСТАНТЫ И IMMUTABLE ==========

contract ConstantsOptimization {
    // ПЛОХО: занимает storage слот
    uint256 public badMaxSupply = 1000000;
    
    // ХОРОШО: константа встраивается в bytecode
    uint256 public constant GOOD_MAX_SUPPLY = 1000000;
    
    // ХОРОШО: immutable устанавливается в constructor, затем в bytecode
    uint256 public immutable owner;
    address public immutable token;
    
    constructor(address _token) {
        owner = uint256(uint160(msg.sender));
        token = _token;
    }
    
    // constant и immutable не занимают storage
    // Экономия: ~2,100 газа на каждое чтение
}

// ========== 8. СОБЫТИЯ ВМЕСТО STORAGE ==========

contract EventsVsStorage {
    // ПЛОХО: хранение истории в storage
    struct Transaction {
        address from;
        address to;
        uint256 amount;
        uint256 timestamp;
    }
    
    Transaction[] public badTransactionHistory;
    
    function badAddTransaction(address to, uint256 amount) external {
        badTransactionHistory.push(Transaction({
            from: msg.sender,
            to: to,
            amount: amount,
            timestamp: block.timestamp
        }));
        // Очень дорого: ~40,000+ газа
    }
    
    // ХОРОШО: использование событий
    event TransactionMade(
        address indexed from,
        address indexed to,
        uint256 amount,
        uint256 timestamp
    );
    
    function goodAddTransaction(address to, uint256 amount) external {
        emit TransactionMade(msg.sender, to, amount, block.timestamp);
        // Дешево: ~1,500 газа
        // Экономия: ~38,500 газа!
    }
    
    // События индексируются, доступны off-chain
    // Используйте события для логирования, не storage
}

// ========== 9. MAPPING VS ARRAY ==========

contract MappingVsArray {
    // ПЛОХО: поиск в массиве
    address[] public badUsers;
    
    function badAddUser(address user) external {
        // Проверка дубликатов - O(n)
        for (uint256 i = 0; i < badUsers.length; i++) {
            require(badUsers[i] != user, "Already exists");
        }
        badUsers.push(user);
    }
    
    function badRemoveUser(address user) external {
        // Поиск и удаление - O(n)
        for (uint256 i = 0; i < badUsers.length; i++) {
            if (badUsers[i] == user) {
                badUsers[i] = badUsers[badUsers.length - 1];
                badUsers.pop();
                break;
            }
        }
    }
    
    // ХОРОШО: mapping для быстрого доступа
    mapping(address => bool) public goodUsers;
    
    function goodAddUser(address user) external {
        require(!goodUsers[user], "Already exists");
        goodUsers[user] = true;
        // O(1) - константное время
    }
    
    function goodRemoveUser(address user) external {
        delete goodUsers[user];
        // O(1) - константное время
    }
    
    // Используйте mapping для поиска/проверки существования
    // Используйте array только если нужна итерация
}

// ========== 10. БИТОВЫЕ ОПЕРАЦИИ ==========

contract BitwiseOptimization {
    // ПЛОХО: хранение boolean флагов отдельно
    bool public flag1;
    bool public flag2;
    bool public flag3;
    // 3 storage слота
    
    // ХОРОШО: упаковка в uint256
    uint256 public flags;
    
    function setFlag(uint256 index, bool value) external {
        if (value) {
            flags |= (1 << index);  // Установить бит
        } else {
            flags &= ~(1 << index); // Сбросить бит
        }
    }
    
    function getFlag(uint256 index) external view returns (bool) {
        return (flags & (1 << index)) != 0;
    }
    
    // Можно хранить 256 boolean в одном uint256!
    // Экономия: ~252 storage слота
}

// ========== 11. ОПТИМИЗАЦИЯ СТРОК ==========

contract StringOptimization {
    // ПЛОХО: string для коротких значений
    string public badName = "Alice";  // Динамический, дорого
    
    // ХОРОШО: bytes32 для коротких строк
    bytes32 public goodName = "Alice";  // Фиксированный, дешево
    
    // Конвертация bytes32 в string (если нужно)
    function bytes32ToString(bytes32 _bytes32) public pure returns (string memory) {
        uint8 i = 0;
        while(i < 32 && _bytes32[i] != 0) {
            i++;
        }
        bytes memory bytesArray = new bytes(i);
        for (uint8 j = 0; j < i; j++) {
            bytesArray[j] = _bytes32[j];
        }
        return string(bytesArray);
    }
    
    // Используйте bytes32 для строк до 32 символов
}

// ========== 12. DELETE ДЛЯ ВОЗВРАТА ГАЗА ==========

contract DeleteOptimization {
    mapping(address => uint256) public balances;
    
    struct User {
        string name;
        uint256 balance;
        bool active;
    }
    
    mapping(address => User) public users;
    
    // Удаление данных возвращает газ (газ refund)
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        delete balances[msg.sender];  // Возврат газа
        payable(msg.sender).transfer(amount);
    }
    
    function removeUser(address user) external {
        delete users[user];  // Возврат газа за очищенные слоты
    }
    
    // delete устанавливает значение в 0/default
    // Возвращает до 15,000 газа за слот
}

// ========== 13. BATCH ОПЕРАЦИИ ==========

contract BatchOptimization {
    mapping(address => uint256) public balances;
    
    // ПЛОХО: множественные транзакции
    function badTransfer(address to, uint256 amount) external {
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
    // Пользователь вызывает 10 раз = 10 транзакций
    
    // ХОРОШО: batch операция
    function batchTransfer(
        address[] calldata recipients,
        uint256[] calldata amounts
    ) external {
        require(recipients.length == amounts.length, "Length mismatch");
        
        uint256 total = 0;
        uint256 length = recipients.length;
        
        for (uint256 i = 0; i < length;) {
            total += amounts[i];
            unchecked { ++i; }
        }
        
        require(balances[msg.sender] >= total, "Insufficient balance");
        balances[msg.sender] -= total;
        
        for (uint256 i = 0; i < length;) {
            balances[recipients[i]] += amounts[i];
            unchecked { ++i; }
        }
    }
    // 1 транзакция вместо 10 - экономия на базовых затратах (21,000 газа * 9)
}

// ========== 14. ОПТИМИЗАЦИЯ REQUIRE/REVERT ==========

contract RequireOptimization {
    // ПЛОХО: длинные error сообщения
    function badValidation(uint256 amount) external pure {
        require(
            amount > 0,
            "The amount must be greater than zero for this operation to proceed"
        );
        // Длинная строка = больше газа
    }
    
    // ХОРОШО: короткие сообщения
    function goodValidation(uint256 amount) external pure {
        require(amount > 0, "Amount is zero");
        // Короткая строка = меньше газа
    }
    
    // ЛУЧШЕ: custom errors (Solidity 0.8.4+)
    error AmountIsZero();
    error InsufficientBalance(uint256 available, uint256 required);
    
    function bestValidation(uint256 amount, uint256 balance) external pure {
        if (amount == 0) revert AmountIsZero();
        if (balance < amount) revert InsufficientBalance(balance, amount);
        // Custom errors намного дешевле!
        // Экономия: ~50 газа
    }
}

// ========== 15. ОПТИМИЗАЦИЯ ФУНКЦИЙ ==========

contract FunctionOptimization {
    uint256 public value;
    
    // ПЛОХО: public автоматически создает getter
    function badGetValue() public view returns (uint256) {
        return value;
    }
    // Зачем функция если уже есть public getter?
    
    // ХОРОШО: используйте public переменную
    // uint256 public value; уже имеет getter
    
    // ПЛОХО: external с небольшими параметрами
    function badSet(uint8 a) external {
        value = a;
    }
    
    // ХОРОШО: внешние функции с calldata
    function goodBatchSet(uint256[] calldata values) external {
        for (uint256 i = 0; i < values.length;) {
            value += values[i];
            unchecked { ++i; }
        }
    }
    
    // Используйте external вместо public если функция вызывается только извне
    // external на ~10 газа дешевле
}

// ========== 16. INLINE ASSEMBLY ДЛЯ КРИТИЧНЫХ УЧАСТКОВ ==========

contract AssemblyOptimization {
    // ПЛОХО: обычный Solidity
    function badAddition(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
    
    // ХОРОШО: assembly (только для критичных мест!)
    function goodAddition(uint256 a, uint256 b) external pure returns (uint256 result) {
        assembly {
            result := add(a, b)
        }
    }
    
    // Эффективная копия массива
    function copyArray(uint256[] calldata data) external pure returns (uint256[] memory) {
        uint256[] memory result = new uint256[](data.length);
        
        assembly {
            let length := mload(data.length)
            let src := data.offset
            let dst := add(result, 0x20)
            
            for { let i := 0 } lt(i, length) { i := add(i, 1) } {
                mstore(add(dst, mul(i, 0x20)), calldataload(add(src, mul(i, 0x20))))
            }
        }
        
        return result;
    }
    
    // ВНИМАНИЕ: assembly обходит проверки безопасности!
    // Используйте только если знаете что делаете
}

// ========== 17. ОПТИМИЗАЦИЯ MODIFIER ==========

contract ModifierOptimization {
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    // ПЛОХО: modifier с кодом
    modifier badOnlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }
    
    function badFunction() external badOnlyOwner {
        // Код modifier копируется в функцию
    }
    
    // ХОРОШО: internal функция вместо modifier
    function _onlyOwner() internal view {
        require(msg.sender == owner, "Not owner");
    }
    
    function goodFunction() external {
        _onlyOwner();
        // Код вызывается, не копируется
        // Экономия: ~10-20 газа
    }
    
    // Используйте internal функции вместо modifier если он используется редко
}

// ========== 18. LAZY EVALUATION ==========

contract LazyEvaluation {
    mapping(address => uint256) public scores;
    uint256 public lastCalculatedTotal;
    uint256 public lastCalculationTime;
    
    // ПЛОХО: вычисление при каждом обновлении
    function badUpdateScore(address user, uint256 score) external {
        scores[user] = score;
        
        // Пересчитываем тотал каждый раз
        uint256 total = 0;
        // ... сложные вычисления ...
        lastCalculatedTotal = total;
    }
    
    // ХОРОШО: вычисление по требованию
    function goodUpdateScore(address user, uint256 score) external {
        scores[user] = score;
        // Не пересчитываем сразу
    }
    
    function getTotalScore() external returns (uint256) {
        // Вычисляем только когда нужно
        if (block.timestamp - lastCalculationTime > 1 days) {
            // ... сложные вычисления ...
            lastCalculationTime = block.timestamp;
        }
        return lastCalculatedTotal;
    }
}

// ========== 19. ОПТИМИЗАЦИЯ TRANSFER ==========

contract TransferOptimization {
    // ПЛОХО: transfer (ограничен 2300 газа)
    function badSend(address payable to, uint256 amount) external {
        to.transfer(amount);
        // Может не работать с контрактами
    }
    
    // ХОРОШО: call с проверкой
    function goodSend(address payable to, uint256 amount) external {
        (bool success, ) = to.call{value: amount}("");
        require(success, "Transfer failed");
        // Передает весь газ, работает с контрактами
    }
}

// ========== 20. ПРАКТИЧЕСКИЙ ПРИМЕР: ОПТИМИЗИРОВАННЫЙ ERC20 ==========

contract OptimizedERC20 {
    string public constant name = "Optimized Token";
    string public constant symbol = "OPT";
    uint8 public constant decimals = 18;
    
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    error InsufficientBalance();
    error InsufficientAllowance();
    
    constructor(uint256 _totalSupply) {
        totalSupply = _totalSupply;
        balanceOf[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }
    
    function transfer(address to, uint256 amount) external returns (bool) {
        return _transfer(msg.sender, to, amount);
    }
    
    function transferFrom(address from, address to, uint256 amount) external returns (bool) {
        uint256 allowed = allowance[from][msg.sender];
        if (allowed != type(uint256).max) {
            if (allowed < amount) revert InsufficientAllowance();
            
            unchecked {
                allowance[from][msg.sender] = allowed - amount;
            }
        }
        
        return _transfer(from, to, amount);
    }
    
    function approve(address spender, uint256 amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }
    
    function _transfer(address from, address to, uint256 amount) internal returns (bool) {
        uint256 balance = balanceOf[from];
        if (balance < amount) revert InsufficientBalance();
        
        unchecked {
            balanceOf[from] = balance - amount;
            balanceOf[to] += amount;
        }
        
        emit Transfer(from, to, amount);
        return true;
    }
    
    // Batch transfer для экономии
    function batchTransfer(address[] calldata recipients, uint256[] calldata amounts) external {
        uint256 length = recipients.length;
        require(length == amounts.length);
        
        uint256 total = 0;
        for (uint256 i = 0; i < length;) {
            total += amounts[i];
            unchecked { ++i; }
        }
        
        uint256 balance = balanceOf[msg.sender];
        if (balance < total) revert InsufficientBalance();
        
        unchecked {
            balanceOf[msg.sender] = balance - total;
        }
        
        for (uint256 i = 0; i < length;) {
            address to = recipients[i];
            uint256 amount = amounts[i];
            
            balanceOf[to] += amount;
            emit Transfer(msg.sender, to, amount);
            
            unchecked { ++i; }
        }
    }
}

/*
РЕЗЮМЕ GAS ОПТИМИЗАЦИИ:

TOP ТЕХНИКИ:
1. ✅ Упаковка переменных в storage (uint128, uint64...)
2. ✅ Использование calldata для external функций
3. ✅ Кеширование storage переменных в memory
4. ✅ unchecked для операций без overflow
5. ✅ ++i вместо i++
6. ✅ constant и immutable
7. ✅ Custom errors вместо require строк
8. ✅ События вместо storage для логов
9. ✅ Mapping вместо массивов для поиска
10. ✅ Batch операции вместо множественных вызовов

ИЗМЕРЕНИЕ ГАЗА:
- Hardhat: hardhat-gas-reporter
- Foundry: forge test --gas-report
- Remix: встроенный gas estimation

ПРОФИЛИРОВАНИЕ:
- Тестируйте на реальных данных
- Сравнивайте до/после
- Фокусируйтесь на частых операциях
- Баланс между читаемостью и оптимизацией

ПРИОРИТЕТЫ:
1. Безопасность (всегда первое!)
2. Читаемость кода
3. Оптимизация газа
4. Сложность кода

НЕ ОПТИМИЗИРУЙТЕ ПРЕЖДЕВРЕМЕННО!
- Сначала напишите рабочий код
- Затем профилируйте
- Оптимизируйте узкие места
- Измеряйте результаты

ИНСТРУМЕНТЫ:
- Solidity Optimizer (runs parameter)
- via-ir (новый оптимизатор)
- Assembly для критичных мест
- Gas snapshots для отслеживания изменений
*/
````
## Assebly
Переменные и присваивание
````assembly
assembly {
    let x := 10          // объявление переменной
    let y := add(x, 5)   // y = 15
    x := mul(x, 2)       // x = 20
}
````
Арифметические операции
````assembly
assembly {
    let a := add(5, 3)      // сложение: 8
    let b := sub(10, 4)     // вычитание: 6
    let c := mul(3, 4)      // умножение: 12
    let d := div(10, 2)     // деление: 5
    let e := mod(10, 3)     // остаток: 1
    let f := exp(2, 3)      // возведение в степень: 8
}
````
Логические операции
```assembly
assembly {
    let x := and(0xFF, 0x0F)    // побитовое И
    let y := or(0xF0, 0x0F)     // побитовое ИЛИ
    let z := xor(0xFF, 0x0F)    // побитовое исключающее ИЛИ
    let w := not(0xFF)          // побитовое НЕ
}
```

## Работа с памятью

### Memory (временная память)
````assembly
assembly {
    // Выделение памяти
    let ptr := mload(0x40)  // получить указатель свободной памяти
    
    // Запись в память
    mstore(ptr, 0x123)      // записать 32 байта
    mstore8(ptr, 0xFF)      // записать 1 байт
    
    // Чтение из памяти
    let value := mload(ptr)
    
    // Обновление указателя свободной памяти
    mstore(0x40, add(ptr, 0x20))
}
````
Storage (постоянное хранилище)
````assembly
assembly {
    // Чтение из storage
    let value := sload(0)   // читать из слота 0
    
    // Запись в storage
    sstore(0, 42)           // записать в слот 0
}
````
Условные операторы
````assembly
assembly {
    let x := 10
    
    // if (эквивалент if без else)
    if gt(x, 5) {
        x := add(x, 1)
    }
    
    // switch (эквивалент switch/case)
    switch x
    case 10 {
        x := 100
    }
    case 20 {
        x := 200
    }
    default {
        x := 0
    }
}
````
Циклы
````assembly
assembly {
    // for цикл
    for { let i := 0 } lt(i, 10) { i := add(i, 1) } {
        // тело цикла
    }
    
    // бесконечный цикл с break
    let i := 0
    for {} true {} {
        i := add(i, 1)
        if eq(i, 10) { break }
    }
}
````
Функции в Assembly
````assembly
assembly {
    function power(base, exponent) -> result {
        result := 1
        for { let i := 0 } lt(i, exponent) { i := add(i, 1) } {
            result := mul(result, base)
        }
    }
    
    let result := power(2, 8)  // 256
}
````
Работа с вызовами (Calls)
````assembly
assembly {
    // call - вызов другого контракта
    let success := call(
        gas(),          // газ
        target,         // адрес
        value,          // ETH для отправки
        argsOffset,     // позиция аргументов в памяти
        argsSize,       // размер аргументов
        retOffset,      // позиция возвращаемых данных
        retSize         // размер возвращаемых данных
    )
    
    // delegatecall
    let success := delegatecall(gas(), target, argsOffset, argsSize, retOffset, retSize)
    
    // staticcall (только чтение)
    let success := staticcall(gas(), target, argsOffset, argsSize, retOffset, retSize)
}
````
Полезные встроенные функции
````assembly
assembly {
    // Информация о блоке и транзакции
    let blockNum := number()
    let timestamp := timestamp()
    let coinbase := coinbase()
    let difficulty := difficulty()
    let gasLimit := gaslimit()
    let chainId := chainid()
    
    // Информация о вызове
    let caller := caller()
    let callValue := callvalue()
    let gasLeft := gas()
    
    // Информация о контракте
    let addr := address()
    let balance := selfbalance()
    let codeSize := codesize()
    
    // Работа с calldata
    let dataSize := calldatasize()
    let dataValue := calldataload(0)
    calldatacopy(destOffset, offset, length)
    
    // Возврат и revert
    return(offset, length)
    revert(offset, length)
}
````
Практический пример
````solidity
contract AssemblyExample {
    // Оптимизированное сложение массива
    function sumArray(uint256[] memory arr) public pure returns (uint256 sum) {
        assembly {
            // arr указывает на длину массива в памяти
            let len := mload(arr)
            
            // Данные начинаются после длины (+ 0x20 байт)
            let data := add(arr, 0x20)
            
            // Конец данных
            let end := add(data, mul(len, 0x20))
            
            // Цикл по массиву
            for {} lt(data, end) {} {
                sum := add(sum, mload(data))
                data := add(data, 0x20)
            }
        }
    }
    
    // Эффективная проверка на ноль
    function isZero(uint256 x) public pure returns (bool result) {
        assembly {
            result := iszero(x)
        }
    }
}
````
Операторы сравнения
````assembly
assembly {
    let isLess := lt(5, 10)        // 5 < 10 = true (1)
    let isGreater := gt(10, 5)     // 10 > 5 = true (1)
    let isEqual := eq(5, 5)        // 5 == 5 = true (1)
    let isZero := iszero(0)        // true (1)
    let slt := slt(-1, 1)          // signed less than
    let sgt := sgt(1, -1)          // signed greater than
}
````
### Низкоуровневый вызов другого контракта
````solidity
contract B {
    address immutable _owner;

    constructor(address _addr) {
        _owner = _addr;
    }

    function start(uint256 a, uint256 b) public returns (uint256 result) {
        bytes4 selector = bytes4(keccak256("run(uint256,uint256)"));
        assembly {
            let ptr := mload(0x40) // свободная память

            // записываем селектор
            mstore(ptr, selector)
            // записываем аргументы
            mstore(add(ptr, 0x04), a)
            mstore(add(ptr, 0x24), b)

            let success := call(
                gas(),
                sload(_owner.slot), // читаем адрес из storage
                0,
                ptr,
                0x44, // 4 + 32 + 32
                ptr,
                0x20
            )

            if iszero(success) {
                revert(0, 0)
            }

            result := mload(ptr)
        }
    }
}
````

Checklist для тестирования уязвимостей

### ✅ Reentrancy - проверка состояния перед внешними вызовами

#### Минимальный контракт с защитой повторного входа

````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/**
 * @title SimpleVault
 * @dev Demonstrates the “checks‑effects‑interactions” pattern
 *      to prevent re‑entrancy.
 */
contract SimpleVault {
    // --------------------------------------------------------------------
    // State
    // --------------------------------------------------------------------
    mapping(address => uint256) public balances;

    // --------------------------------------------------------------------
    // Events
    // --------------------------------------------------------------------
    event Deposit(address indexed user, uint256 amount);
    event Withdrawal(address indexed user, uint256 amount);

    // --------------------------------------------------------------------
    // Public API
    // --------------------------------------------------------------------

    /// @notice Deposit ETH into the vault.
    function deposit() external payable {
        require(msg.value > 0, "Zero deposit");
        balances[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }

    /// @notice Withdraw all ETH belonging to the caller.
    ///         Uses the checks‑effects‑interactions pattern.
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "Nothing to withdraw");

        // ---------- Checks ----------
        // (Already done via the require above.)

        // ---------- Effects ----------
        // Update the state *before* the external call.
        balances[msg.sender] = 0;

        // ---------- Interactions ----------
        // Send ETH. Using low‑level call to forward all gas and
        // handle possible failure gracefully.
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");

        emit Withdrawal(msg.sender, amount);
    }
}
````
````solidity
contract Attacker {
    SimpleVault public target;

    constructor(address _target) {
        target = SimpleVault(_target);
    }

    // Receive ETH and try to re‑enter
    receive() external payable {
        // Attempt to call withdraw again
        target.withdraw();
    }

    function attack() external payable {
        // Fund the vault first
        target.deposit{value: msg.value}();
        // Then trigger the vulnerable withdrawal
        target.withdraw();
    }
}
````
Быстрая защита «одной строкой»
````
bool private locked;
modifier nonReentrant() {
    require(!locked, "Reentrancy detected");
    locked = true;
    _;
    locked = false;
}
function withdraw() external nonReentrant { … }
````
### ✅ Access Control - проверка модификаторов и прав доступа
Пример собственного модификатора onlyOwner
````
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract SimpleAccess {
    // Адрес, которому принадлежит контракт
    address public owner;

    // Событие, чтобы видеть изменения владельца
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    // Конструктор задаёт первоначального владельца
    constructor() {
        owner = msg.sender;
        emit OwnershipTransferred(address(0), owner);
    }

    // Модификатор, проверяющий, что вызывающий — владелец
    modifier onlyOwner() {
        require(msg.sender == owner, "SimpleAccess: caller is not the owner");
        _;
    }

    // Функция, доступная только владельцу
    function privilegedAction(string calldata data) external onlyOwner {
        // здесь может быть любая логика, требующая контроля доступа
    }

    // Позволяем передать владение другому адресу
    function transferOwnership(address newOwner) external onlyOwner {
        require(newOwner != address(0), "SimpleAccess: new owner is zero address");
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }
}
````
Пример с OpenZeppelinAccessControl
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

// Подключаем библиотеку из OpenZeppelin
import "@openzeppelin/contracts/access/AccessControl.sol";

contract RoleBasedAccess is AccessControl {
    // Определяем идентификаторы ролей (bytes32‑константы)
    bytes32 public constant ADMIN_ROLE  = keccak256("ADMIN_ROLE");
    bytes32 public constant WRITER_ROLE = keccak256("WRITER_ROLE");

    // Пример хранилища, к которому будет ограничен доступ
    string private secretData;

    // Событие, чтобы видеть изменения данных
    event DataUpdated(string newData);

    constructor() {
        // Деплойер получает админскую роль и роль администратора (по умолчанию)
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);   // может управлять другими ролями
        _setupRole(ADMIN_ROLE, msg.sender);
    }

    // ------------------- Управление ролями -------------------
    // Только администратор может назначать/удалять роли
    function grantWriter(address account) external onlyRole(ADMIN_ROLE) {
        grantRole(WRITER_ROLE, account);
    }

    function revokeWriter(address account) external onlyRole(ADMIN_ROLE) {
        revokeRole(WRITER_ROLE, account);
    }

    // ------------------- Защищённые функции -------------------
    // Только пользователи с ролью WRITER могут менять данные
    function setSecret(string calldata newData) external onlyRole(WRITER_ROLE) {
        secretData = newData;
        emit DataUpdated(newData);
    }

    // Любой может читать, но запись ограничена ролями
    function getSecret() external view returns (string memory) {
        return secretData;
    }
}
````

### ✅ Integer Overflow/Underflow - арифметические операции
Уязвимый контракт (до Solidity 0.8)
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.6;          // <0.8 → нет автоматических проверок

contract OverflowVulnerable {
    mapping(address => uint256) public balances;

    // Пополняем баланс (для простоты без проверки)
    function deposit() external payable {
        balances[msg.sender] += msg.value;   // ← потенциальный overflow
    }

    // Выводим всю сумму
    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "not enough balance");

        // Уменьшаем баланс (может произойти underflow!)
        balances[msg.sender] -= amount;      // ← потенциальный underflow

        // Отправляем эфир
        (bool ok, ) = msg.sender.call{value: amount}("");
        require(ok, "transfer failed");
    }
}
````
Если используете старый компилятор (0.7.x) – подключите SafeMath
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.6;

import "@openzeppelin/contracts/math/SafeMath.sol";

contract OverflowWithSafeMath {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] = balances[msg.sender].add(msg.value);
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "not enough balance");

        balances[msg.sender] = balances[msg.sender].sub(amount);
        (bool ok, ) = msg.sender.call{value: amount}("");
        require(ok, "transfer failed");
    }
}
````
* ✅ Delegatecall - корректность storage layout
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Библиотека, которую вызываем через delegatecall.
 *  Предполагается, что у вызывающего контракта первая переменная storage –
 *  `address public owner`.
 */
library VulnerableLib {
    // Ожидаем, что первый слот – это `owner`.
    // Записываем туда произвольный адрес, полученный от атакующего.
    function setOwner(address _newOwner) external {
        assembly {
            // slot 0 → первая переменная в storage вызывающего контракта
            sstore(0, _newOwner)
        }
    }
}

/*
 *  Основной контракт, который использует delegatecall к библиотеке.
 *  Ошибка: порядок объявленных переменных НЕ совпадает с тем,
 *  что ожидает библиотека.
 */
contract DelegateCaller {
    // 1️⃣ Первая переменная – не владелец, а обычный счётчик.
    uint256 public counter;          // occupies slot 0

    // 2️⃣ Вторая переменная – действительно владелец.
    address public owner;            // occupies slot 1

    // Адрес библиотеки, которую будем вызывать.
    address public libAddress;

    constructor(address _lib) {
        libAddress = _lib;
        owner = msg.sender;
    }

    // Функция, позволяющая изменить владельца через delegatecall.
    // Пользователь передаёт желаемый новый адрес.
    function changeOwner(address _newOwner) external {
        // Формируем calldata для функции `setOwner(address)`
        bytes memory data = abi.encodeWithSignature(
            "setOwner(address)",
            _newOwner
        );

        // Выполняем delegatecall
        (bool success, ) = libAddress.delegatecall(data);
        require(success, "delegatecall failed");
    }

    // Пример функции, меняющей счётчик – показывает, что
    // slot 0 действительно используется.
    function inc() external {
        counter += 1;
    }
}
````
### ✅ tx.origin - использование msg.sender вместо tx.origin
````solidity
// Уязвимый контракт
contract Victim {
    address public owner;

    constructor() {
        owner = msg.sender;               // владелец = тот, кто развернул контракт
    }

    // Функция, доступная только владельцу
    function privileged() external {
        require(tx.origin == owner, "Not owner"); // <-- Плохая проверка!
        // … критическая логика …
    }
}
````
* ✅ Unchecked External Calls - проверка возвращаемых значений
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простой «кошелёк», позволяющий пользователям выводить средства.
 *  Перевод делается через низкоуровневый call без проверки результата.
 */
contract UnsafeWallet {
    mapping(address => uint256) public balances;

    // Пополнение баланса
    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    // Вывод средств – **уязвим**
    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");

        // Уменьшаем баланс **до** выполнения перевода
        balances[msg.sender] -= amount;

        // Низкоуровневый call без проверки возврата!
        // Если получатель является контрактом, который отклонит платеж,
        // то ether будет потерян, а баланс уже уменьшен.
        msg.sender.call{value: amount}("");
    }
}
````
### ✅ DoS - циклы с неограниченной итерацией
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простой «список подписчиков», которым рассылаются уведомления.
 *  Функция broadcast() перебирает массив subscribers и отправляет каждому
 *  небольшую выплату. Количество подписчиков растёт без ограничений.
 *
 *  Если массив станет достаточно большим, broadcast() будет требовать
 *  всё больше газа и в итоге перестанет работать → DoS.
 */
contract UnboundedLoop {
    address[] public subscribers;
    uint256 public reward = 0.001 ether;

    // Любой может подписаться
    function subscribe() external {
        subscribers.push(msg.sender);
    }

    // Рассылка вознаграждения всем подписчикам
    function broadcast() external {
        uint256 len = subscribers.length;
        for (uint256 i = 0; i < len; i++) {
            // Низкоуровневый call, чтобы не зависеть от fallback‑газ‑лимита
            (bool ok, ) = subscribers[i].call{value: reward}("");
            require(ok, "Transfer failed");
        }
    }

    // Приём ether, чтобы иметь средства для выплат
    receive() external payable {}
}
````
### ✅ Front-running - чувствительность к порядку транзакций
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простейший DEX‑контракт, использующий средний курс
 *  от внешнего оракула (priceFeed). Оракул обновляется
 *  только через публичную функцию `setPrice`.
 *
 *  Пользователь вызывает `swapExactETHForToken`, передавая
 *  `minOut`. Если цена изменилась в тот же блок, пользователь
 *  может получить меньше токенов, чем ожидал.
 */
interface IPriceFeed {
    function price() external view returns (uint256); // ETH → token, 18‑decimals
}

contract VulnerableSwap {
    IPriceFeed public priceFeed;
    mapping(address => uint256) public balances;

    constructor(address _priceFeed) {
        priceFeed = IPriceFeed(_priceFeed);
    }

    // Оракул может обновлять цену в любой момент
    function setPrice(uint256 _price) external {
        // в реальном коде здесь будет только owner/keeper
        // (упрощаем для примера)
        (bool ok, ) = address(priceFeed).call(
            abi.encodeWithSignature("setPrice(uint256)", _price)
        );
        require(ok, "price update failed");
    }

    // Пользователь меняет ETH → токен
    function swapExactETHForToken(uint256 minOut) external payable {
        uint256 price = priceFeed.price();          // читаем цену
        uint256 tokenAmount = (msg.value * price) / 1e18;

        require(tokenAmount >= minOut, "slippage too high");
        balances[msg.sender] += tokenAmount;
    }
}
````
### ✅ Timestamp Dependence - использование block.timestamp
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простой аукцион, где победителем считается тот,
 *  кто поставил самую высокую ставку **до** `endTime`.
 *  Победитель может вызвать `claim()` только после окончания
 *  аукциона.
 *
 *  Уязвимость: победитель (или любой участник) может
 *  заставить майнера выставить `block.timestamp` **меньше**
 *  реального `endTime`, тем самым «замораживая» аукцион
 *  и удерживая средства в контракте.
 */
contract VulnerableAuction {
    address public highestBidder;
    uint256 public highestBid;
    uint256 public endTime;          // время окончания аукциона

    constructor(uint256 _duration) {
        endTime = block.timestamp + _duration; // e.g. 3 days
    }

    // Делать ставку
    function bid() external payable {
        require(block.timestamp < endTime, "Auction already ended");
        require(msg.value > highestBid, "Bid too low");

        // Возврат предыдущей ставки
        if (highestBidder != address(0)) {
            (bool ok, ) = highestBidder.call{value: highestBid}("");
            require(ok, "Refund failed");
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    // Получить выигрышный предмет после окончания
    function claim() external {
        require(block.timestamp >= endTime, "Auction not over yet");
        require(msg.sender == highestBidder, "Not winner");
        // выдача предмета…
    }
}
````
### ✅ Signature Replay - nonce и защита от повторов
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простой кошелёк, позволяющий снять средства,
 *  если пользователь подпишет сообщение:
 *      keccak256(abi.encodePacked(to, amount, deadline))
 *
 *  Нет nonce → подпись может быть использована многократно.
 */
contract ReplayVulnerable {
    mapping(address => uint256) public balances;

    // Пользователь вносит депозит
    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    // Вывод средств по подписи
    function withdrawSigned(
        address to,
        uint256 amount,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external {
        require(block.timestamp <= deadline, "Expired");

        // Формируем хеш, который подписал пользователь
        bytes32 hash = keccak256(abi.encodePacked(to, amount, deadline));
        address signer = ecrecover(hash, v, r, s);
        require(signer != address(0) && signer == msg.sender, "Bad sig");

        require(balances[signer] >= amount, "Insufficient");
        balances[signer] -= amount;
        (bool ok, ) = to.call{value: amount}("");
        require(ok, "Transfer failed");
    }
}
````
### ✅ Storage Collision - корректность proxy паттернов
````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/*
 *  Простейший proxy, который хранит только один адрес implementation.
 *  Важно: переменная `implementation` занимает **слот 0**.
 *
 *  Implementation‑контракт (Version 1) ожидает, что его первый слот —
 *  будет `owner`. Из‑за несовпадения происходит коллизия.
 */
contract BadProxy {
    // slot 0 – implementation address
    address public implementation;

    constructor(address _impl) {
        implementation = _impl;
    }

    // Любой вызов перенаправляется в implementation
    fallback() external payable {
        (bool ok, ) = implementation.delegatecall(msg.data);
        require(ok, "delegatecall failed");
    }

    receive() external payable {}
}

/* -------------------------------------------------------------
   Implementation‑контракт, ожидающий layout:
   slot 0 : owner
   slot 1 : storedValue
   ------------------------------------------------------------- */
contract BadImplementationV1 {
    address public owner;          // <-- ожидается в slot 0
    uint256 public storedValue;    // <-- slot 1

    constructor() {
        owner = msg.sender;
    }

    function setValue(uint256 _v) external {
        require(msg.sender == owner, "not owner");
        storedValue = _v;
    }
}
````
### ✅ Selfdestruct - зависимость от address(this).balance
````
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.6;

/*
 *  Уязвимый контракт:
 *  - Любой может вызвать `killMe()` и получить весь баланс.
 *  - Нет проверки, что вызывающий действительно имеет право.
 *  - Не проверяется, что баланс не изменился между проверкой и selfdestruct.
 */
contract VulnerableKill {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    // Приём ether
    receive() external payable {}

    // Любой может вызвать, если знает пароль (пример)
    function killMe(bytes32 password) external {
        require(keccak256(abi.encodePacked(password)) == 0xdeadbeef..., "bad pwd");

        // Плохая проверка баланса (может стать устаревшей)
        require(address(this).balance > 0, "empty");

        // Переходим к selfdestruct без обновления состояния
        selfdestruct(payable(msg.sender));
    }
}
````
Шпаргалка
Порядок старшинства операторов
Ниже приведен порядок приоритета операторов, перечисленных в порядке оценки.

Предшествование
````
Постфиксное приращение и уменьшение

++, --

Новое выражение

new <typename>

Подстрочные индексы массива

<array>[<index>]

Доступ для участников

<object>.<member>

Функциональный вызов

<func>(<args...>)

Круглые скобки

(<statement>)

2

Приращение и уменьшение префикса

++, --

Унарный минус

-

Унарные операции

delete

Логично НЕ

!

Побитовый НЕТ

~

3

Возведение в степень

**

4

Умножение, деление и модуль

*, , /%

5

Сложение и вычитание

+, -

6

Операторы побитового сдвига

<<, >>

7

Побитовое И

&

8

Побитовый XOR

^

9

Побитовое ИЛИ

|

10

Операторы неравенства

<, , , ><=>=

11

Операторы равенства

==, !=

12

Логическое И

&&

13

Логическое ИЛИ

||

14

Тернарный оператор

<conditional> ? <if-true> : <if-false>

Операторы присваивания

=, , , , , , , , , , |=^=&=<<=>>=+=-=*=/=%=

15

Оператор запятой

,

Функции кодирования и декодирования ABI
abi.decode(bytes memory encodedData, (...)) returns (...): ABI-декодирование предоставленные данные. Типы задаются в скобках в качестве второго аргумента. Пример: (uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))

abi.encode(...) returns (bytes memory): ABI-кодирует заданные аргументы

abi.encodePacked(...) returns (bytes memory): Выполняет упакованное кодирование Приведенные аргументы. Обратите внимание, что эта кодировка может быть неоднозначной!

abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory): ABI-кодирует заданные аргументы, начиная со второго и добавляет в начало заданного четырехбайтового селектора

abi.encodeCall(function functionPointer, (...)) returns (bytes memory): ABI кодирует вызов с аргументами, найденными в кортеж. Выполняет полную проверку типов, убедившись, что типы соответствуют сигнатуре функции. Результат равен functionPointerabi.encodeWithSelector(functionPointer.selector, ...)

abi.encodeWithSignature(string memory signature, ...) returns (bytes memory):Эквивалентный Кому abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)

Члены и bytesstring
bytes.concat(...) returns (bytes memory): Объединяет переменное число аргументы в массив в один байт

string.concat(...) returns (string memory): Объединяет переменное число аргументов в один строковый массив

Члены address
<address>.balance (uint256): баланс адреса в Вэй

<address>.code (bytes memory): код по адресу (может быть пустым)

<address>.codehash (bytes32): codehash адреса

<address>.call(bytes memory) returns (bool, bytes memory): выдача низкоуровневых с заданной полезной нагрузкой, Возвращает условие успеха и возвращаемые данныеCALL

<address>.delegatecall(bytes memory) returns (bool, bytes memory): выдача низкоуровневых с заданной полезной нагрузкой, Возвращает условие успеха и возвращаемые данныеDELEGATECALL

<address>.staticcall(bytes memory) returns (bool, bytes memory): выдача низкоуровневых с заданной полезной нагрузкой, Возвращает условие успеха и возвращаемые данныеSTATICCALL

<address payable>.send(uint256 amount) returns (bool): отправить указанное количество Wei на Адрес, Возврат при неудачеfalse

<address payable>.transfer(uint256 amount): отправить указанное количество Wei на Address, выкидывает при сбое

Свойства блока и транзакции
blockhash(uint blockNumber) returns (bytes32): хеш данного блока - работает только для 256 последних блоков

blobhash(uint index) returns (bytes32): версионный хеш -го большого двоичного объекта, связанного с текущей транзакцией. Версионный хеш состоит из одного байта, представляющего версию (в настоящее время), за которым следует последний 31 байт хеша SHA256 обязательства KZG (EIP-4844). Возвращает ноль, если большого двоичного объекта с заданным индексом не существует.index0x01

block.basefee (uint): базовая комиссия текущего блока (EIP-3198 и EIP-1559)

block.blobbasefee (uint): базовая плата за блоб текущего блока (EIP-7516 и EIP-4844)

block.chainid (uint): идентификатор текущей цепочки

block.coinbase (address payable): адрес майнера текущего блока

block.difficulty (uint): сложность текущего блока (). Для других версий EVM он ведет себя как устаревший псевдоним, который будет удален в следующем ломающем выпускеEVM < Parisblock.prevrandao

block.gaslimit (uint): предел газа в блоке тока

block.number (uint): текущий номер блока

block.prevrandao (uint): случайное число, выдаваемое цепочкой маяка () (см. EIP-4399EVM >= Paris )

block.timestamp (uint): временная метка текущего блока в секундах с момента эпохи Unix

gasleft() returns (uint256): оставшийся газ

msg.data (bytes): полные данные вызова

msg.sender (address): отправитель сообщения (текущий вызов)

msg.sig (bytes4): первые четыре байта calldata (т.е. идентификатор функции)

msg.value (uint): количество вэй, отправленных с сообщением

tx.gasprice (uint): цена газа по сделке

tx.origin (address): отправитель транзакции (полная цепочка вызовов)

Проверки и утверждения
assert(bool condition): прервать выполнение и отменить изменения состояния, если условие равно (используется для внутренней ошибки)false

require(bool condition): прервать выполнение и отменить изменения состояния, если условие равно (use за неправильно сформированный ввод или ошибку во внешнем компоненте)false

require(bool condition, string memory message): прерывание выполнения и отмена изменений состояния, если условие (используется для неправильно сформированного ввода или ошибки во внешнем компоненте). Также предоставьте сообщение об ошибке.false

revert(): прерывание выполнения и отмена изменений состояния

revert(string memory message): прерывание выполнения и отмена изменений состояния, предоставление пояснительной строки

Математические и криптографические функции
keccak256(bytes memory) returns (bytes32): вычислить хеш Keccak-256 входных данных

sha256(bytes memory) returns (bytes32): вычисление хэша SHA-256 входных данных

ripemd160(bytes memory) returns (bytes20): вычисление хэша RIPEMD-160 входных данных

ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address): восстановить адрес, связанный с Открытый ключ из подписи эллиптической кривой, возвращает ноль по ошибке

addmod(uint x, uint y, uint k) returns (uint): вычислить, где сложение выполняется с помощью произвольной точности и не оборачивается в точке . Подтвердите, что начиная с версии 0.5.0.(x + y) % k2**256k != 0

mulmod(uint x, uint y, uint k) returns (uint): вычислить, где выполняется умножение с произвольной точностью и не оборачивается в точке . Подтвердите, что начиная с версии 0.5.0.(x * y) % k2**256k != 0

Договорные связи
this (тип текущего контракта): текущий контракт, явно конвертируемый в или addressaddress payable

super: контракт на один уровень выше в иерархии наследования

selfdestruct(address payable recipient): отправить все средства на указанный адрес и (только на EVM до Канкуна или при вызове в рамках транзакции, создающей контракт) уничтожить контракт.

Информация о типе
type(C).name (string): наименование договора

type(C).creationCode (bytes memory): создание байт-кода данного контракта, см. Информация о типе.

type(C).runtimeCode (bytes memory): байт-код времени выполнения данного контракта, см. Информация о типе.

type(I).interfaceId (bytes4): значение, содержащее идентификатор интерфейса EIP-165 данного интерфейса, см. Информация о типе.

type(T).min (T): минимальное значение, представляемое целочисленным типом, см. Информация о типе.T

type(T).max (T): максимальное значение, представляемое целочисленным типом, см. Информация о типе.T

Спецификаторы видимости функций
открыть в Remix

function myFunction() <visibility specifier> returns (bool) {
    return true;
}
public: видимый внешне и внутренне (создает функцию getter для переменных хранилища/состояния)

private: отображается только в текущем контракте

external: видимый только извне (только для функций) - т.е. может быть вызван только через сообщение (через this.func)

internal: виден только внутри

Модификаторы
pure для функций: Запрещает изменение или доступ к состоянию.

view для функций: Запрещает изменение состояния.

payable для функций: Позволяет им принимать эфир вместе с вызовом.

constant для переменных состояния: Запрещает присваивание (кроме инициализации), не занимает место для хранения.

immutable для переменных состояния: Разрешает назначение во время создания и является постоянным при развертывании. Хранится в коде.

anonymous для событий: Не сохраняет сигнатуру события в качестве темы.

indexed для параметров события: сохраняет параметр как topic.

virtual для функций и модификаторов: Разрешает функцию или модификатор Поведение, которое должно быть изменено в производных контрактах.

override: Указывает, что эта функция, модификатор или общедоступная переменная состояния изменяется Поведение функции или модификатора в базовом контракте.

vm.deal(address, amount): Начисляем тестовые ETH на указанный адрес.
vm.addr(privateKey): Получаем адрес из приватного ключа.
address owner = vm.addr(PRIVATE_KEY);
2. Манипуляция отправителем транзакции
vm.prank(address): Подменяет адрес отправителя (msg.sender) для следующей транзакции.
vm.prank(user); // Следующая транзакция будет отправлена от имени `user`

vm.startPrank(address) / vm.stopPrank(): Начинает и завершает серию транзакций от имени указанного адреса.
vm.startPrank(user);
store.toPay{value: 1 ether}();
store.someOtherFunction();
vm.stopPrank();
3. Ожидание ошибок
vm.expectRevert(): Ожидает, что следующая транзакция завершится с ошибкой.
 Copyvm.expectRevert(); // Ожидаем ошибку
store.widthDraft(); // Эта транзакция должна завершиться с ошибкой
vm.expectRevert(bytes memory reason): Ожидает, что транзакция завершится с конкретной причиной ошибки.
 Copyvm.expectRevert("Not owner".abiEncode());
store.widthDraft();
4. Управление состоянием блокчейна
vm.roll(blockNumber): Откатывает состояние блокчейна к указанному блоку.
vm.roll(100); // Откатываемся к блоку 100
vm.warp(timestamp): Устанавливает текущее время блокчейна.
vm.warp(block.timestamp + 1 days); // Перемещаемся на 1 день вперёд
vm.etch(deployer, bytecode): Развёртывает контракт с указанным байткодом.
 Copyvm.etch(address(this), type(Store).creationCode);


5. Работа с событиями
vm.expectEmit(): Ожидает, что следующая транзакция вызовет определённое событие.
vm.expectEmit(true, true, true, true);
emit store.PaymentReceived(1 ether);

6. Манипуляция газом
vm.recordLogs(): Записывает логи транзакций для последующего анализа.
vm.getGasUsed(): Возвращает количество газа, потраченного на последнюю транзакцию.

7. Симуляция вызовов между контрактами
vm.mockCall(address, data, returnData): Мокает (подменяет) результат вызова контракта.
vm.mockCall(
    address(someContract),
    abi.encodeWithSignature("someFunction()"),
    abi.encode(true) // Возвращаемое значение
);
9. Симуляция форков сети
vm.createFork(rpcUrl): Создаёт форк реальной сети (например, Mainnet) для тестирования.
vm.createFork("https://mainnet.infura.io/v3/YOUR_INFURA_KEY");

vm.selectFork(forkId): Переключается на указанный форк.
vm.selectFork(1);

console.log(address(admin).balance);
````

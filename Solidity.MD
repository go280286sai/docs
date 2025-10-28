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
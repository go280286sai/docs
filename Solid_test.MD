````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "forge-std/Test.sol";
import "./Atak.sol";
import "./Shop.sol";

contract ShopTest is Test {
    Shop shop;
    Atak atak;
    
    address owner;
    address user1;
    address user2;
    address attacker;
    
    // События для тестирования
    event ItemPurchased(address buyer, uint256 itemId, uint256 price);
    event OwnerChanged(address oldOwner, address newOwner);
    
    function setUp() public {
        owner = address(this);
        user1 = makeAddr("user1");
        user2 = makeAddr("user2");
        attacker = makeAddr("attacker");
        
        shop = new Shop();
        atak = new Atak(address(shop));
        
        // Даём тестовым адресам ETH
        vm.deal(user1, 100 ether);
        vm.deal(user2, 100 ether);
        vm.deal(attacker, 100 ether);
    }
    
    receive() external payable {}
    fallback() external {}
    
    // ============================================
    // ТЕСТЫ ДЕПЛОЯ И ИНИЦИАЛИЗАЦИИ
    // ============================================
    
    function testDeployment() public {
        assertEq(address(shop).code.length > 0, true, "Shop should be deployed");
        assertEq(address(atak).code.length > 0, true, "Atak should be deployed");
    }
    
    function testInitialOwner() public view {
        // Проверяем владельца, если есть функция owner()
        // assertEq(shop.owner(), owner);
    }
    
    function testInitialBalance() public view {
        assertEq(address(shop).balance, 0, "Initial balance should be 0");
    }
    
    // ============================================
    // ТЕСТЫ БАЗОВОЙ ФУНКЦИОНАЛЬНОСТИ
    // ============================================
    
    function testBuyItem() public {
        vm.startPrank(user1);
        // Покупка предмета
        // shop.buy{value: 1 ether}(itemId);
        vm.stopPrank();
    }
    
    function testBuyMultipleItems() public {
        vm.startPrank(user1);
        // shop.buy{value: 1 ether}(1);
        // shop.buy{value: 2 ether}(2);
        vm.stopPrank();
    }
    
    function testGetPrice() public view {
        // uint256 price = shop.price();
        // assertEq(price, 1 ether);
    }
    
    function testIsSold() public {
        // bool sold = shop.isSold();
        // assertFalse(sold);
    }
    
    // ============================================
    // ТЕСТЫ С vm.PRANK (ИМИТАЦИЯ ВЫЗОВОВ)
    // ============================================
    
    function testPrankAsDifferentUser() public {
        vm.prank(user1);
        // shop.someFunction();
        
        vm.prank(user2);
        // shop.someFunction();
    }
    
    function testStartStopPrank() public {
        vm.startPrank(user1);
        // Все вызовы от имени user1
        // shop.function1();
        // shop.function2();
        vm.stopPrank();
        
        // Теперь вызовы от текущего контракта
        // shop.function3();
    }
    
    // ============================================
    // ТЕСТЫ С vm.DEAL (УПРАВЛЕНИЕ БАЛАНСАМИ)
    // ============================================
    
    function testDealEther() public {
        vm.deal(user1, 50 ether);
        assertEq(user1.balance, 50 ether);
    }
    
    function testBuyWithInsufficientFunds() public {
        vm.deal(user1, 0.1 ether);
        vm.startPrank(user1);
        vm.expectRevert();
        // shop.buy{value: 0.1 ether}();
        vm.stopPrank();
    }
    
    // ============================================
    // ТЕСТЫ СОБЫТИЙ (vm.expectEmit)
    // ============================================
    
    function testEmitEvent() public {
        vm.expectEmit(true, true, false, true);
        emit ItemPurchased(user1, 1, 1 ether);
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}(1);
    }
    
    function testMultipleEvents() public {
        // Проверка нескольких событий
        vm.expectEmit(true, true, false, true);
        emit ItemPurchased(user1, 1, 1 ether);
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}(1);
    }
    
    // ============================================
    // ТЕСТЫ ОШИБОК (vm.expectRevert)
    // ============================================
    
    function testRevertOnZeroValue() public {
        vm.expectRevert();
        // shop.buy{value: 0}();
    }
    
    function testRevertWithMessage() public {
        vm.expectRevert("Insufficient payment");
        vm.prank(user1);
        // shop.buy{value: 0.5 ether}();
    }
    
    function testRevertWithCustomError() public {
        vm.expectRevert(Shop.InsufficientPayment.selector);
        // shop.buy{value: 0}();
    }
    
    function testRevertOnlyOwner() public {
        vm.prank(user1);
        vm.expectRevert();
        // shop.withdraw();
    }
    
    // ============================================
    // ТЕСТЫ ИЗМЕНЕНИЯ СОСТОЯНИЯ
    // ============================================
    
    function testBalanceChangeAfterPurchase() public {
        uint256 initialBalance = address(shop).balance;
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}();
        
        assertEq(address(shop).balance, initialBalance + 1 ether);
    }
    
    function testBuyerBalanceDecrease() public {
        uint256 initialBalance = user1.balance;
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}();
        
        assertLt(user1.balance, initialBalance);
    }
    
    // ============================================
    // ТЕСТЫ ВРЕМЕННЫХ МЕТОК (vm.warp)
    // ============================================
    
    function testTimeBasedFunction() public {
        uint256 futureTime = block.timestamp + 1 days;
        vm.warp(futureTime);
        
        assertEq(block.timestamp, futureTime);
        // shop.executeAfterDelay();
    }
    
    function testSaleExpiration() public {
        // Пропускаем 7 дней
        vm.warp(block.timestamp + 7 days);
        
        vm.expectRevert("Sale ended");
        // shop.buy{value: 1 ether}();
    }
    
    // ============================================
    // ТЕСТЫ БЛОКОВ (vm.roll)
    // ============================================
    
    function testBlockNumber() public {
        uint256 targetBlock = block.number + 100;
        vm.roll(targetBlock);
        
        assertEq(block.number, targetBlock);
    }
    
    function testBlockBasedLock() public {
        // shop.lockForBlocks(100);
        
        vm.roll(block.number + 99);
        vm.expectRevert("Still locked");
        // shop.unlock();
        
        vm.roll(block.number + 1);
        // shop.unlock();
    }
    
    // ============================================
    // ТЕСТЫ ЗАПИСИ В STORAGE (vm.store)
    // ============================================
    
    function testManipulateStorage() public {
        // Прямая запись в storage слот 0
        bytes32 value = bytes32(uint256(1234));
        vm.store(address(shop), bytes32(uint256(0)), value);
        
        // Проверяем изменение
        bytes32 stored = vm.load(address(shop), bytes32(uint256(0)));
        assertEq(stored, value);
    }
    
    // ============================================
    // ТЕСТЫ ЧТЕНИЯ STORAGE (vm.load)
    // ============================================
    
    function testReadStorage() public view {
        // Читаем значение из слота 0
        bytes32 value = vm.load(address(shop), bytes32(uint256(0)));
        // Проверяем значение
    }
    
    // ============================================
    // ТЕСТЫ ВЫЗОВОВ К ДРУГИМ КОНТРАКТАМ
    // ============================================
    
    function testAtakContract() public {
        vm.prank(attacker);
        // atak.exploit();
    }
    
    function testContractInteraction() public {
        // Проверка взаимодействия между контрактами
        address(atak).call("");
    }
    
    // ============================================
    // ТЕСТЫ REENTRANCY
    // ============================================
    
    function testReentrancyAttack() public {
        vm.prank(attacker);
        vm.expectRevert();
        // atak.reentrancyAttack();
    }
    
    // ============================================
    // ТЕСТЫ СНИМКА СОСТОЯНИЯ (vm.snapshot / vm.revertTo)
    // ============================================
    
    function testSnapshot() public {
        // Сохраняем состояние
        uint256 snapshot = vm.snapshot();
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}();
        assertEq(address(shop).balance, 1 ether);
        
        // Откатываем состояние
        vm.revertTo(snapshot);
        assertEq(address(shop).balance, 0);
    }
    
    // ============================================
    // ТЕСТЫ ЛОГОВ (vm.recordLogs / vm.getRecordedLogs)
    // ============================================
    
    function testRecordLogs() public {
        vm.recordLogs();
        
        vm.prank(user1);
        // shop.buy{value: 1 ether}();
        
        Vm.Log[] memory logs = vm.getRecordedLogs();
        assertGt(logs.length, 0);
    }
    
    // ============================================
    // ТЕСТЫ ОЖИДАНИЯ ВЫЗОВОВ (vm.expectCall)
    // ============================================
    
    function testExpectCall() public {
        vm.expectCall(
            address(shop),
            abi.encodeWithSignature("buy()")
        );
        
        // shop.buy{value: 1 ether}();
    }
    
    // ============================================
    // ТЕСТЫ MOCK КОНТРАКТОВ (vm.mockCall)
    // ============================================
    
    function testMockCall() public {
        // Мокаем вызов
        vm.mockCall(
            address(shop),
            abi.encodeWithSignature("price()"),
            abi.encode(5 ether)
        );
        
        // uint256 price = shop.price();
        // assertEq(price, 5 ether);
        
        vm.clearMockedCalls();
    }
    
    // ============================================
    // FUZZ ТЕСТЫ (СЛУЧАЙНЫЕ ВХОДНЫЕ ДАННЫЕ)
    // ============================================
    
    function testFuzzBuyAmount(uint256 amount) public {
        vm.assume(amount > 0 && amount < 100 ether);
        
        vm.deal(user1, amount);
        vm.prank(user1);
        // shop.buy{value: amount}();
    }
    
    function testFuzzMultipleUsers(address randomUser) public {
        vm.assume(randomUser != address(0));
        vm.assume(randomUser != address(shop));
        
        vm.deal(randomUser, 10 ether);
        vm.prank(randomUser);
        // shop.buy{value: 1 ether}();
    }
    
    function testFuzzPriceRange(uint256 price) public {
        price = bound(price, 0.1 ether, 10 ether);
        // shop.setPrice(price);
        // assertEq(shop.price(), price);
    }
    
    // ============================================
    // INVARIANT ТЕСТЫ (ИНВАРИАНТЫ)
    // ============================================
    
    function invariant_BalanceNeverNegative() public view {
        assertGe(address(shop).balance, 0);
    }
    
    function invariant_TotalSupplyConsistent() public view {
        // Проверяем, что сумма балансов == totalSupply
    }
    
    // ============================================
    // ТЕСТЫ ПРАВ ДОСТУПА
    // ============================================
    
    function testOnlyOwnerCanWithdraw() public {
        vm.deal(address(shop), 10 ether);
        
        vm.prank(user1);
        vm.expectRevert();
        // shop.withdraw();
        
        // shop.withdraw(); // Должно пройти от owner
    }
    
    function testAccessControl() public {
        vm.prank(user1);
        vm.expectRevert();
        // shop.adminFunction();
    }
    
    // ============================================
    // ТЕСТЫ ЛИМИТОВ GAS
    // ============================================
    
    function testGasLimit() public {
        uint256 gasBefore = gasleft();
        
        // shop.expensiveFunction();
        
        uint256 gasUsed = gasBefore - gasleft();
        assertLt(gasUsed, 1000000); // Меньше 1М gas
    }
    
    // ============================================
    // ТЕСТЫ ETH ПЕРЕВОДОВ
    // ============================================
    
    function testReceiveEther() public {
        uint256 initialBalance = address(shop).balance;
        
        (bool success,) = address(shop).call{value: 1 ether}("");
        assertTrue(success);
        
        assertEq(address(shop).balance, initialBalance + 1 ether);
    }
    
    function testFallbackFunction() public {
        (bool success,) = address(shop).call{value: 1 ether}("randomData");
        assertTrue(success);
    }
    
    // ============================================
    // ТЕСТЫ КОМПЛЕКСНЫХ СЦЕНАРИЕВ
    // ============================================
    
    function testCompleteUserJourney() public {
        // 1. Пользователь покупает предмет
        vm.startPrank(user1);
        // shop.buy{value: 1 ether}();
        vm.stopPrank();
        
        // 2. Проверяем владение
        // assertTrue(shop.owns(user1));
        
        // 3. Пользователь продает предмет
        vm.startPrank(user1);
        // shop.sell();
        vm.stopPrank();
        
        // 4. Проверяем, что больше не владеет
        // assertFalse(shop.owns(user1));
    }
    
    function testMultipleUsersPurchase() public {
        vm.prank(user1);
        // shop.buy{value: 1 ether}();
        
        vm.prank(user2);
        // shop.buy{value: 1 ether}();
        
        assertEq(address(shop).balance, 2 ether);
    }
    
    // ============================================
    // ТЕСТЫ ГРАНИЧНЫХ УСЛОВИЙ
    // ============================================
    
    function testZeroAddress() public {
        vm.expectRevert();
        // shop.transfer(address(0));
    }
    
    function testMaxUint256() public {
        vm.expectRevert();
        // shop.setPrice(type(uint256).max);
    }
    
    function testOverflow() public {
        // Тест на переполнение
        vm.expectRevert();
        // shop.add(type(uint256).max, 1);
    }
    
    // ============================================
    // ТЕСТЫ CHEATCODES
    // ============================================
    
    function testSign() public {
        uint256 privateKey = 0xA11CE;
        address alice = vm.addr(privateKey);
        
        bytes32 hash = keccak256("message");
        (uint8 v, bytes32 r, bytes32 s) = vm.sign(privateKey, hash);
        
        address signer = ecrecover(hash, v, r, s);
        assertEq(signer, alice);
    }
    
    function testLabel() public {
        vm.label(user1, "Alice");
        vm.label(user2, "Bob");
        vm.label(address(shop), "Shop Contract");
    }
    
    // ============================================
    // ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ
    // ============================================
    
    function _buyAsUser(address user, uint256 value) internal {
        vm.prank(user);
        // shop.buy{value: value}();
    }
    
    function _assertBalances(address user, uint256 expected) internal view {
        assertEq(user.balance, expected);
    }
}
````

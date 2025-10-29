#  Полноценная рабочая блокчейн сеть на Python!
## 📚 Компоненты системы:

### 1. Класс Block (Блок)

* Содержит индекс, время, транзакции, proof и хеш предыдущего блока
* Метод compute_hash() - вычисляет SHA-256 хеш блока

### 2. Класс Blockchain (Основной)

* Транзакции - создание и хранение
* Proof of Work - алгоритм майнинга (ищет хеш с 4 нулями в начале)
* Валидация - проверка целостности цепи
* Консенсус - разрешение конфликтов между узлами
* Балансы - подсчёт баланса адресов

### 3. Flask API (Веб-интерфейс)

* GET /mine - майнинг блока
* POST /transactions/new - создать транзакцию
* GET /chain - получить цепь
* POST /nodes/register - добавить узлы
* GET /nodes/resolve - запустить консенсус
* GET /balance/<address> - баланс адреса

## 🚀 Как запустить:
````
python blockchain.py demo
````
Запуск узла сети:
````
# Узел 1 на порту 5000
python blockchain.py 5000

# Узел 2 на порту 5001
python blockchain.py 5001

# Узел 3 на порту 5002
python blockchain.py 5002
````
## Примеры использования API:

### 1. Создать транзакцию:
````bash
curl -X POST http://localhost:5000/transactions/new \
  -H "Content-Type: application/json" \
  -d '{"sender":"Alice","recipient":"Bob","amount":5.0}'
````

### 2. Майнинг блока:
````bash
curl http://localhost:5000/mine
````

### 3. Получить цепь:
````bash
curl http://localhost:5000/chain
````

### 4. Зарегистрировать узлы:
````bash
curl -X POST http://localhost:5000/nodes/register \
  -H "Content-Type: application/json" \
  -d '{"nodes":["http://localhost:5001","http://localhost:5002"]}'
````

### 5. Консенсус (синхронизация):
````bash
curl http://localhost:5000/nodes/resolve
````

### 6. Проверить баланс:
````bash
curl http://localhost:5000/balance/Alice
````
## 🔐 Ключевые концепции:

* Proof of Work - вычислительная задача для защиты от спама
* Иммутабельность - изменение блока ломает всю цепь
* Консенсус - самая длинная валидная цепь побеждает
* Децентрализация - каждый узел хранит полную копию
````python
import hashlib
import json
import time
from typing import List, Dict, Optional
from datetime import datetime
import requests
from urllib.parse import urlparse
from flask import Flask, jsonify, request


# ============================================
# БЛОК - основная единица блокчейна
# ============================================
class Block:
    """
    Блок содержит:
    - index: порядковый номер блока
    - timestamp: время создания
    - transactions: список транзакций
    - proof: proof of work (nonce)
    - previous_hash: хеш предыдущего блока
    """
    def __init__(self, index: int, timestamp: float, transactions: List[Dict], 
                 proof: int, previous_hash: str):
        self.index = index
        self.timestamp = timestamp
        self.transactions = transactions
        self.proof = proof
        self.previous_hash = previous_hash
    
    def compute_hash(self) -> str:
        """
        Вычисляет SHA-256 хеш блока
        Используется для связывания блоков в цепь
        """
        block_string = json.dumps(self.__dict__, sort_keys=True)
        return hashlib.sha256(block_string.encode()).hexdigest()
    
    def to_dict(self) -> Dict:
        """Конвертирует блок в словарь для JSON сериализации"""
        return {
            'index': self.index,
            'timestamp': self.timestamp,
            'transactions': self.transactions,
            'proof': proof,
            'previous_hash': self.previous_hash
        }


# ============================================
# BLOCKCHAIN - главный класс цепи блоков
# ============================================
class Blockchain:
    """
    Основной класс блокчейна
    Управляет цепью блоков, транзакциями и консенсусом
    """
    
    def __init__(self):
        self.chain: List[Block] = []
        self.current_transactions: List[Dict] = []
        self.nodes: set = set()  # Узлы сети
        
        # Создаём genesis блок (первый блок)
        self.create_genesis_block()
    
    def create_genesis_block(self):
        """
        Genesis блок - первый блок в цепи
        Не имеет предыдущего блока, поэтому previous_hash = "0"
        """
        genesis_block = Block(
            index=0,
            timestamp=time.time(),
            transactions=[],
            proof=100,  # Произвольное начальное значение
            previous_hash="0"
        )
        self.chain.append(genesis_block)
        print("🎉 Genesis блок создан!")
    
    def get_last_block(self) -> Block:
        """Возвращает последний блок в цепи"""
        return self.chain[-1]
    
    # ============================================
    # ТРАНЗАКЦИИ
    # ============================================
    
    def new_transaction(self, sender: str, recipient: str, amount: float) -> int:
        """
        Создаёт новую транзакцию
        
        Args:
            sender: адрес отправителя
            recipient: адрес получателя
            amount: сумма
        
        Returns:
            index блока, в который будет включена транзакция
        """
        transaction = {
            'sender': sender,
            'recipient': recipient,
            'amount': amount,
            'timestamp': time.time()
        }
        
        self.current_transactions.append(transaction)
        
        # Возвращаем индекс следующего блока
        return self.get_last_block().index + 1
    
    # ============================================
    # PROOF OF WORK - алгоритм консенсуса
    # ============================================
    
    def proof_of_work(self, last_proof: int) -> int:
        """
        Простой алгоритм Proof of Work:
        - Найти число p', такое что hash(pp') содержит 4 нуля в начале
        - p - предыдущий proof, p' - новый proof
        
        Это создаёт вычислительную сложность для майнинга
        """
        proof = 0
        while not self.valid_proof(last_proof, proof):
            proof += 1
        
        print(f"⛏️  Майнинг завершён! Найден proof: {proof}")
        return proof
    
    @staticmethod
    def valid_proof(last_proof: int, proof: int, difficulty: int = 4) -> bool:
        """
        Проверяет proof: hash(last_proof, proof) начинается с N нулей?
        
        Args:
            last_proof: предыдущий proof
            proof: текущий proof
            difficulty: количество нулей в начале (сложность)
        """
        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:difficulty] == "0" * difficulty
    
    # ============================================
    # СОЗДАНИЕ НОВЫХ БЛОКОВ
    # ============================================
    
    def mine_block(self, miner_address: str) -> Block:
        """
        Майнинг нового блока:
        1. Вычисляем proof of work
        2. Добавляем вознаграждение майнеру
        3. Создаём новый блок
        4. Добавляем в цепь
        """
        last_block = self.get_last_block()
        last_proof = last_block.proof
        
        # Вычисляем proof of work (это занимает время!)
        print("⛏️  Начинаем майнинг...")
        proof = self.proof_of_work(last_proof)
        
        # Награда майнеру (от "системы")
        self.new_transaction(
            sender="0",  # 0 означает новые монеты
            recipient=miner_address,
            amount=1.0  # Награда за блок
        )
        
        # Создаём новый блок
        previous_hash = last_block.compute_hash()
        new_block = Block(
            index=len(self.chain),
            timestamp=time.time(),
            transactions=self.current_transactions,
            proof=proof,
            previous_hash=previous_hash
        )
        
        # Очищаем список транзакций
        self.current_transactions = []
        
        # Добавляем блок в цепь
        self.chain.append(new_block)
        
        print(f"✅ Блок #{new_block.index} добавлен в цепь!")
        return new_block
    
    # ============================================
    # ВАЛИДАЦИЯ ЦЕПИ
    # ============================================
    
    def is_valid_chain(self, chain: List[Block]) -> bool:
        """
        Проверяет валидность цепи:
        1. Проверяет хеши блоков
        2. Проверяет proof of work
        """
        previous_block = chain[0]
        current_index = 1
        
        while current_index < len(chain):
            block = chain[current_index]
            
            # Проверка 1: Хеш предыдущего блока совпадает?
            if block.previous_hash != previous_block.compute_hash():
                print(f"❌ Цепь невалидна: хеш не совпадает на блоке {current_index}")
                return False
            
            # Проверка 2: Proof of work валиден?
            if not self.valid_proof(previous_block.proof, block.proof):
                print(f"❌ Цепь невалидна: proof неверен на блоке {current_index}")
                return False
            
            previous_block = block
            current_index += 1
        
        return True
    
    # ============================================
    # КОНСЕНСУС - разрешение конфликтов
    # ============================================
    
    def register_node(self, address: str):
        """
        Добавляет новый узел в сеть
        
        Args:
            address: адрес узла, например 'http://192.168.0.5:5000'
        """
        parsed_url = urlparse(address)
        self.nodes.add(parsed_url.netloc)
        print(f"🔗 Узел {parsed_url.netloc} добавлен в сеть")
    
    def resolve_conflicts(self) -> bool:
        """
        Алгоритм консенсуса:
        Заменяет нашу цепь самой длинной валидной цепью в сети
        
        Returns:
            True если цепь была заменена, False если нет
        """
        neighbours = self.nodes
        new_chain = None
        
        # Ищем цепи длиннее нашей
        max_length = len(self.chain)
        
        # Проверяем все узлы в сети
        for node in neighbours:
            try:
                response = requests.get(f'http://{node}/chain')
                
                if response.status_code == 200:
                    length = response.json()['length']
                    chain_data = response.json()['chain']
                    
                    # Конвертируем JSON в объекты Block
                    chain = []
                    for block_data in chain_data:
                        block = Block(
                            index=block_data['index'],
                            timestamp=block_data['timestamp'],
                            transactions=block_data['transactions'],
                            proof=block_data['proof'],
                            previous_hash=block_data['previous_hash']
                        )
                        chain.append(block)
                    
                    # Проверяем: цепь длиннее и валидна?
                    if length > max_length and self.is_valid_chain(chain):
                        max_length = length
                        new_chain = chain
            except:
                continue
        
        # Заменяем цепь если нашли лучшую
        if new_chain:
            self.chain = new_chain
            print("🔄 Цепь заменена более длинной версией!")
            return True
        
        print("✅ Наша цепь авторитетна")
        return False
    
    # ============================================
    # ПОЛУЧЕНИЕ БАЛАНСА
    # ============================================
    
    def get_balance(self, address: str) -> float:
        """
        Вычисляет баланс адреса путём сканирования всей цепи
        """
        balance = 0.0
        
        for block in self.chain:
            for transaction in block.transactions:
                if transaction['recipient'] == address:
                    balance += transaction['amount']
                if transaction['sender'] == address:
                    balance -= transaction['amount']
        
        return balance
    
    # ============================================
    # ВСПОМОГАТЕЛЬНЫЕ МЕТОДЫ
    # ============================================
    
    def to_dict(self) -> Dict:
        """Конвертирует всю цепь в словарь"""
        chain_data = []
        for block in self.chain:
            chain_data.append({
                'index': block.index,
                'timestamp': block.timestamp,
                'transactions': block.transactions,
                'proof': block.proof,
                'previous_hash': block.previous_hash,
                'hash': block.compute_hash()
            })
        return chain_data
    
    def print_chain(self):
        """Красиво выводит всю цепь"""
        print("\n" + "="*60)
        print("📊 BLOCKCHAIN")
        print("="*60)
        
        for block in self.chain:
            print(f"\n🔗 Блок #{block.index}")
            print(f"   Время: {datetime.fromtimestamp(block.timestamp)}")
            print(f"   Транзакций: {len(block.transactions)}")
            print(f"   Proof: {block.proof}")
            print(f"   Предыдущий хеш: {block.previous_hash[:16]}...")
            print(f"   Хеш: {block.compute_hash()[:16]}...")
            
            if block.transactions:
                print("   Транзакции:")
                for tx in block.transactions:
                    print(f"      {tx['sender'][:8]} → {tx['recipient'][:8]}: {tx['amount']} BTC")
        
        print("\n" + "="*60 + "\n")


# ============================================
# FLASK API - веб-интерфейс для блокчейна
# ============================================

# Создаём Flask приложение
app = Flask(__name__)

# Создаём уникальный адрес для этого узла
import uuid
node_identifier = str(uuid.uuid4()).replace('-', '')

# Инициализируем блокчейн
blockchain = Blockchain()


@app.route('/mine', methods=['GET'])
def mine():
    """Эндпоинт для майнинга нового блока"""
    # Майним новый блок
    block = blockchain.mine_block(node_identifier)
    
    response = {
        'message': 'Новый блок добавлен!',
        'index': block.index,
        'transactions': block.transactions,
        'proof': block.proof,
        'previous_hash': block.previous_hash,
        'hash': block.compute_hash()
    }
    
    return jsonify(response), 200


@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    """Создание новой транзакции"""
    values = request.get_json()
    
    # Проверяем наличие необходимых полей
    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return 'Отсутствуют необходимые поля', 400
    
    # Создаём транзакцию
    index = blockchain.new_transaction(
        values['sender'],
        values['recipient'],
        values['amount']
    )
    
    response = {'message': f'Транзакция будет добавлена в блок {index}'}
    return jsonify(response), 201


@app.route('/chain', methods=['GET'])
def full_chain():
    """Возвращает полную цепь"""
    response = {
        'chain': blockchain.to_dict(),
        'length': len(blockchain.chain)
    }
    return jsonify(response), 200


@app.route('/nodes/register', methods=['POST'])
def register_nodes():
    """Регистрирует новые узлы"""
    values = request.get_json()
    nodes = values.get('nodes')
    
    if nodes is None:
        return "Ошибка: список узлов пуст", 400
    
    for node in nodes:
        blockchain.register_node(node)
    
    response = {
        'message': 'Новые узлы добавлены',
        'total_nodes': list(blockchain.nodes)
    }
    return jsonify(response), 201


@app.route('/nodes/resolve', methods=['GET'])
def consensus():
    """Запускает алгоритм консенсуса"""
    replaced = blockchain.resolve_conflicts()
    
    if replaced:
        response = {
            'message': 'Наша цепь заменена',
            'new_chain': blockchain.to_dict()
        }
    else:
        response = {
            'message': 'Наша цепь авторитетна',
            'chain': blockchain.to_dict()
        }
    
    return jsonify(response), 200


@app.route('/balance/<address>', methods=['GET'])
def get_balance(address):
    """Получает баланс адреса"""
    balance = blockchain.get_balance(address)
    
    response = {
        'address': address,
        'balance': balance
    }
    return jsonify(response), 200


@app.route('/', methods=['GET'])
def index():
    """Главная страница с информацией"""
    return jsonify({
        'node_id': node_identifier,
        'endpoints': {
            'mine': '/mine - Майнинг нового блока',
            'new_transaction': '/transactions/new - Создать транзакцию',
            'chain': '/chain - Получить всю цепь',
            'register': '/nodes/register - Зарегистрировать узлы',
            'consensus': '/nodes/resolve - Запустить консенсус',
            'balance': '/balance/<address> - Получить баланс'
        }
    }), 200


# ============================================
# ДЕМОНСТРАЦИЯ РАБОТЫ
# ============================================

def demo():
    """Демонстрация работы блокчейна"""
    print("\n" + "🚀 ДЕМОНСТРАЦИЯ BLOCKCHAIN " + "\n")
    
    # Создаём блокчейн
    bc = Blockchain()
    
    print("\n1️⃣ Создаём транзакции...")
    bc.new_transaction("Alice", "Bob", 5.0)
    bc.new_transaction("Bob", "Charlie", 2.0)
    
    print("\n2️⃣ Майним блок 1...")
    bc.mine_block("Miner1")
    
    print("\n3️⃣ Создаём ещё транзакции...")
    bc.new_transaction("Charlie", "Alice", 1.0)
    bc.new_transaction("Alice", "Bob", 3.0)
    
    print("\n4️⃣ Майним блок 2...")
    bc.mine_block("Miner2")
    
    print("\n5️⃣ Проверяем балансы...")
    print(f"Баланс Alice: {bc.get_balance('Alice')} BTC")
    print(f"Баланс Bob: {bc.get_balance('Bob')} BTC")
    print(f"Баланс Charlie: {bc.get_balance('Charlie')} BTC")
    print(f"Баланс Miner1: {bc.get_balance('Miner1')} BTC")
    print(f"Баланс Miner2: {bc.get_balance('Miner2')} BTC")
    
    print("\n6️⃣ Проверяем валидность цепи...")
    is_valid = bc.is_valid_chain(bc.chain)
    print(f"Цепь валидна: {is_valid}")
    
    print("\n7️⃣ Выводим всю цепь...")
    bc.print_chain()
    
    print("\n8️⃣ Попытка подделки блока...")
    # Пытаемся изменить транзакцию в блоке 1
    bc.chain[1].transactions[0]['amount'] = 1000.0
    print("Изменили сумму первой транзакции на 1000 BTC")
    
    print("\n9️⃣ Проверяем валидность после подделки...")
    is_valid = bc.is_valid_chain(bc.chain)
    print(f"Цепь валидна: {is_valid}")
    print("❌ Цепь стала невалидной! Подделка обнаружена!")


# ============================================
# ЗАПУСК
# ============================================

if __name__ == '__main__':
    import sys
    
    if len(sys.argv) > 1 and sys.argv[1] == 'demo':
        # Запуск демонстрации
        demo()
    else:
        # Запуск API сервера
        port = 5000
        if len(sys.argv) > 1:
            port = int(sys.argv[1])
        
        print(f"\n🚀 Запуск Blockchain узла на порту {port}")
        print(f"📍 Node ID: {node_identifier}\n")
        app.run(host='0.0.0.0', port=port)
````
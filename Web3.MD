# Web3 разработки: Python & JavaScript

# Web3.py (Python)

````
# ========== УСТАНОВКА ==========
"""
pip install web3
pip install python-dotenv
pip install eth-account
"""

from web3 import Web3
from web3.middleware import geth_poa_middleware
from eth_account import Account
import json
import os
from dotenv import load_dotenv

# Загрузка переменных окружения
load_dotenv()

# ========== ПОДКЛЮЧЕНИЕ К БЛОКЧЕЙНУ ==========

class BlockchainConnector:
    def __init__(self, provider_url):
        """
        Инициализация подключения к блокчейну
        
        provider_url примеры:
        - HTTP: "http://127.0.0.1:8545" (локальный)
        - Infura: "https://mainnet.infura.io/v3/YOUR-PROJECT-ID"
        - Alchemy: "https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY"
        - WebSocket: "wss://mainnet.infura.io/ws/v3/YOUR-PROJECT-ID"
        """
        self.w3 = Web3(Web3.HTTPProvider(provider_url))
        
        # Для PoA сетей (Polygon, BSC и т.д.)
        self.w3.middleware_onion.inject(geth_poa_middleware, layer=0)
        
        # Проверка подключения
        if self.w3.is_connected():
            print(f"✅ Подключено к блокчейну")
            print(f"Chain ID: {self.w3.eth.chain_id}")
        else:
            raise Exception("❌ Не удалось подключиться к блокчейну")
    
    def get_connection(self):
        return self.w3

# ========== БАЗОВЫЕ ОПЕРАЦИИ ==========

def basic_operations():
    # Подключение
    w3 = Web3(Web3.HTTPProvider('https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY'))
    
    # 1. ПРОВЕРКА ПОДКЛЮЧЕНИЯ
    print(f"Connected: {w3.is_connected()}")
    
    # 2. ПОЛУЧИТЬ ПОСЛЕДНИЙ БЛОК
    latest_block = w3.eth.block_number
    print(f"Latest block: {latest_block}")
    
    # 3. ИНФОРМАЦИЯ О БЛОКЕ
    block = w3.eth.get_block('latest')
    print(f"Block timestamp: {block['timestamp']}")
    print(f"Block transactions: {len(block['transactions'])}")
    
    # 4. ПОЛУЧИТЬ БАЛАНС АДРЕСА
    address = '0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045'  # Vitalik's address
    balance_wei = w3.eth.get_balance(address)
    balance_eth = w3.from_wei(balance_wei, 'ether')
    print(f"Balance: {balance_eth} ETH")
    
    # 5. ЦЕНА ГАЗА
    gas_price = w3.eth.gas_price
    gas_price_gwei = w3.from_wei(gas_price, 'gwei')
    print(f"Gas price: {gas_price_gwei} Gwei")
    
    # 6. ИНФОРМАЦИЯ О ТРАНЗАКЦИИ
    tx_hash = '0x...'  # hash транзакции
    # tx = w3.eth.get_transaction(tx_hash)
    # print(f"From: {tx['from']}")
    # print(f"To: {tx['to']}")
    # print(f"Value: {w3.from_wei(tx['value'], 'ether')} ETH")
    
    # 7. RECEIPT ТРАНЗАКЦИИ
    # receipt = w3.eth.get_transaction_receipt(tx_hash)
    # print(f"Status: {'Success' if receipt['status'] == 1 else 'Failed'}")
    # print(f"Gas used: {receipt['gasUsed']}")

# ========== РАБОТА С АККАУНТАМИ ==========

class WalletManager:
    def __init__(self, w3):
        self.w3 = w3
    
    def create_account(self):
        """Создать новый аккаунт"""
        account = Account.create()
        return {
            'address': account.address,
            'private_key': account.key.hex()
        }
    
    def import_account(self, private_key):
        """Импортировать аккаунт по приватному ключу"""
        account = Account.from_key(private_key)
        return account
    
    def get_balance(self, address):
        """Получить баланс в ETH"""
        balance_wei = self.w3.eth.get_balance(address)
        return self.w3.from_wei(balance_wei, 'ether')
    
    def get_nonce(self, address):
        """Получить nonce для транзакции"""
        return self.w3.eth.get_transaction_count(address)

# Пример использования
def wallet_example():
    w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))
    wallet = WalletManager(w3)
    
    # Создать новый кошелек
    new_account = wallet.create_account()
    print(f"New Address: {new_account['address']}")
    print(f"Private Key: {new_account['private_key']}")
    
    # ВАЖНО: Храните приватный ключ в .env файле!
    # Никогда не коммитьте его в git!

# ========== ОТПРАВКА ТРАНЗАКЦИЙ ==========

class TransactionManager:
    def __init__(self, w3, private_key):
        self.w3 = w3
        self.account = Account.from_key(private_key)
        self.address = self.account.address
    
    def send_eth(self, to_address, amount_eth):
        """
        Отправить ETH
        
        Args:
            to_address: адрес получателя
            amount_eth: количество ETH
        """
        # Конвертируем ETH в Wei
        amount_wei = self.w3.to_wei(amount_eth, 'ether')
        
        # Получаем nonce
        nonce = self.w3.eth.get_transaction_count(self.address)
        
        # Создаем транзакцию
        transaction = {
            'nonce': nonce,
            'to': to_address,
            'value': amount_wei,
            'gas': 21000,  # стандартный gas для ETH перевода
            'gasPrice': self.w3.eth.gas_price,
            'chainId': self.w3.eth.chain_id
        }
        
        # Подписываем транзакцию
        signed_txn = self.w3.eth.account.sign_transaction(
            transaction, 
            self.account.key
        )
        
        # Отправляем транзакцию
        tx_hash = self.w3.eth.send_raw_transaction(signed_txn.rawTransaction)
        
        print(f"Transaction sent! Hash: {tx_hash.hex()}")
        
        # Ждем подтверждения
        tx_receipt = self.w3.eth.wait_for_transaction_receipt(tx_hash)
        
        if tx_receipt['status'] == 1:
            print("✅ Transaction successful!")
        else:
            print("❌ Transaction failed!")
        
        return tx_hash.hex()
    
    def send_eip1559_transaction(self, to_address, amount_eth):
        """
        Отправить транзакцию с EIP-1559 (London fork)
        Более эффективное использование газа
        """
        amount_wei = self.w3.to_wei(amount_eth, 'ether')
        nonce = self.w3.eth.get_transaction_count(self.address)
        
        # Получаем базовую комиссию и priority fee
        latest_block = self.w3.eth.get_block('latest')
        base_fee = latest_block['baseFeePerGas']
        max_priority_fee = self.w3.to_wei(2, 'gwei')  # tip для майнера
        max_fee = base_fee * 2 + max_priority_fee  # максимальная готовность платить
        
        transaction = {
            'nonce': nonce,
            'to': to_address,
            'value': amount_wei,
            'gas': 21000,
            'maxFeePerGas': max_fee,
            'maxPriorityFeePerGas': max_priority_fee,
            'chainId': self.w3.eth.chain_id,
            'type': 2  # EIP-1559
        }
        
        signed_txn = self.w3.eth.account.sign_transaction(
            transaction, 
            self.account.key
        )
        
        tx_hash = self.w3.eth.send_raw_transaction(signed_txn.rawTransaction)
        print(f"EIP-1559 Transaction sent! Hash: {tx_hash.hex()}")
        
        return tx_hash.hex()

# ========== РАБОТА СО СМАРТ-КОНТРАКТАМИ ==========

class ContractManager:
    def __init__(self, w3, contract_address, abi):
        """
        Инициализация контракта
        
        Args:
            w3: Web3 instance
            contract_address: адрес контракта
            abi: ABI контракта (JSON)
        """
        self.w3 = w3
        self.contract = w3.eth.contract(
            address=Web3.to_checksum_address(contract_address),
            abi=abi
        )
        self.address = contract_address
    
    def call_view_function(self, function_name, *args):
        """
        Вызвать view/pure функцию (не требует газа)
        
        Example:
            balance = contract.call_view_function('balanceOf', user_address)
        """
        function = getattr(self.contract.functions, function_name)
        return function(*args).call()
    
    def send_transaction(self, private_key, function_name, *args, value=0):
        """
        Отправить транзакцию (изменяет состояние)
        
        Example:
            tx_hash = contract.send_transaction(
                private_key,
                'transfer',
                recipient,
                amount
            )
        """
        account = Account.from_key(private_key)
        
        # Получаем функцию
        function = getattr(self.contract.functions, function_name)
        
        # Строим транзакцию
        transaction = function(*args).build_transaction({
            'from': account.address,
            'nonce': self.w3.eth.get_transaction_count(account.address),
            'gas': 200000,  # можно оценить через estimate_gas
            'gasPrice': self.w3.eth.gas_price,
            'value': value,
            'chainId': self.w3.eth.chain_id
        })
        
        # Подписываем
        signed_txn = self.w3.eth.account.sign_transaction(
            transaction,
            private_key
        )
        
        # Отправляем
        tx_hash = self.w3.eth.send_raw_transaction(signed_txn.rawTransaction)
        
        # Ждем подтверждения
        receipt = self.w3.eth.wait_for_transaction_receipt(tx_hash)
        
        return {
            'tx_hash': tx_hash.hex(),
            'status': receipt['status'],
            'gas_used': receipt['gasUsed']
        }
    
    def estimate_gas(self, function_name, *args, from_address=None):
        """Оценить необходимый газ"""
        function = getattr(self.contract.functions, function_name)
        return function(*args).estimate_gas({'from': from_address})

# ========== ПРИМЕР: РАБОТА С ERC-20 ТОКЕНОМ ==========

def erc20_example():
    # ABI для ERC-20 (упрощенный)
    ERC20_ABI = json.loads('''
    [
        {
            "constant": true,
            "inputs": [],
            "name": "name",
            "outputs": [{"name": "", "type": "string"}],
            "type": "function"
        },
        {
            "constant": true,
            "inputs": [],
            "name": "symbol",
            "outputs": [{"name": "", "type": "string"}],
            "type": "function"
        },
        {
            "constant": true,
            "inputs": [],
            "name": "totalSupply",
            "outputs": [{"name": "", "type": "uint256"}],
            "type": "function"
        },
        {
            "constant": true,
            "inputs": [{"name": "account", "type": "address"}],
            "name": "balanceOf",
            "outputs": [{"name": "", "type": "uint256"}],
            "type": "function"
        },
        {
            "constant": false,
            "inputs": [
                {"name": "recipient", "type": "address"},
                {"name": "amount", "type": "uint256"}
            ],
            "name": "transfer",
            "outputs": [{"name": "", "type": "bool"}],
            "type": "function"
        },
        {
            "constant": false,
            "inputs": [
                {"name": "spender", "type": "address"},
                {"name": "amount", "type": "uint256"}
            ],
            "name": "approve",
            "outputs": [{"name": "", "type": "bool"}],
            "type": "function"
        }
    ]
    ''')
    
    # Подключение
    w3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR-KEY'))
    
    # USDT контракт на Ethereum
    usdt_address = '0xdAC17F958D2ee523a2206206994597C13D831ec7'
    
    # Создаем контракт
    usdt = ContractManager(w3, usdt_address, ERC20_ABI)
    
    # Читаем данные (не требует газа)
    name = usdt.call_view_function('name')
    symbol = usdt.call_view_function('symbol')
    total_supply = usdt.call_view_function('totalSupply')
    
    print(f"Token: {name} ({symbol})")
    print(f"Total Supply: {total_supply / 10**6} USDT")  # USDT has 6 decimals
    
    # Проверить баланс
    user_address = '0x...'
    balance = usdt.call_view_function('balanceOf', user_address)
    print(f"Balance: {balance / 10**6} USDT")
    
    # Отправить токены (требует газ и приватный ключ)
    # private_key = os.getenv('PRIVATE_KEY')
    # recipient = '0x...'
    # amount = 100 * 10**6  # 100 USDT
    # 
    # result = usdt.send_transaction(
    #     private_key,
    #     'transfer',
    #     recipient,
    #     amount
    # )
    # print(f"Transaction: {result['tx_hash']}")

# ========== СЛУШАТЬ СОБЫТИЯ (EVENTS) ==========

class EventListener:
    def __init__(self, w3, contract_address, abi):
        self.w3 = w3
        self.contract = w3.eth.contract(
            address=Web3.to_checksum_address(contract_address),
            abi=abi
        )
    
    def get_past_events(self, event_name, from_block=0, to_block='latest'):
        """
        Получить прошлые события
        
        Example:
            transfers = listener.get_past_events(
                'Transfer',
                from_block=15000000,
                to_block=15001000
            )
        """
        event = getattr(self.contract.events, event_name)
        return event.get_logs(fromBlock=from_block, toBlock=to_block)
    
    def listen_for_events(self, event_name, callback):
        """
        Слушать новые события в реальном времени
        Требует WebSocket подключение
        """
        event = getattr(self.contract.events, event_name)
        event_filter = event.create_filter(fromBlock='latest')
        
        while True:
            for event in event_filter.get_new_entries():
                callback(event)

# Пример использования
def event_example():
    w3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR-KEY'))
    
    # ERC-20 контракт
    contract_address = '0x...'
    abi = [...]  # ABI с событием Transfer
    
    listener = EventListener(w3, contract_address, abi)
    
    # Получить прошлые трансферы
    transfers = listener.get_past_events('Transfer', from_block=18000000, to_block=18001000)
    
    for transfer in transfers:
        print(f"From: {transfer['args']['from']}")
        print(f"To: {transfer['args']['to']}")
        print(f"Amount: {transfer['args']['value']}")
        print(f"Block: {transfer['blockNumber']}")
        print("---")

# ========== ДЕПЛОЙ КОНТРАКТА ==========

def deploy_contract(w3, private_key, bytecode, abi, constructor_args=None):
    """
    Задеплоить смарт-контракт
    
    Args:
        w3: Web3 instance
        private_key: приватный ключ деплоера
        bytecode: скомпилированный bytecode контракта
        abi: ABI контракта
        constructor_args: аргументы конструктора
    """
    account = Account.from_key(private_key)
    
    # Создаем контракт
    Contract = w3.eth.contract(abi=abi, bytecode=bytecode)
    
    # Строим транзакцию деплоя
    if constructor_args:
        transaction = Contract.constructor(*constructor_args).build_transaction({
            'from': account.address,
            'nonce': w3.eth.get_transaction_count(account.address),
            'gas': 3000000,
            'gasPrice': w3.eth.gas_price,
            'chainId': w3.eth.chain_id
        })
    else:
        transaction = Contract.constructor().build_transaction({
            'from': account.address,
            'nonce': w3.eth.get_transaction_count(account.address),
            'gas': 3000000,
            'gasPrice': w3.eth.gas_price,
            'chainId': w3.eth.chain_id
        })
    
    # Подписываем
    signed_txn = w3.eth.account.sign_transaction(transaction, private_key)
    
    # Отправляем
    tx_hash = w3.eth.send_raw_transaction(signed_txn.rawTransaction)
    print(f"Deploy transaction sent: {tx_hash.hex()}")
    
    # Ждем подтверждения
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    
    contract_address = receipt['contractAddress']
    print(f"✅ Contract deployed at: {contract_address}")
    
    return contract_address

# ========== ПОЛНЫЙ ПРИМЕР: ERC-20 BOT ==========

class ERC20Bot:
    """Бот для мониторинга и взаимодействия с ERC-20 токеном"""
    
    def __init__(self, provider_url, contract_address, abi, private_key=None):
        self.w3 = Web3(Web3.HTTPProvider(provider_url))
        self.contract = ContractManager(self.w3, contract_address, abi)
        self.private_key = private_key
        
        if private_key:
            self.account = Account.from_key(private_key)
            self.address = self.account.address
    
    def get_token_info(self):
        """Получить информацию о токене"""
        return {
            'name': self.contract.call_view_function('name'),
            'symbol': self.contract.call_view_function('symbol'),
            'decimals': self.contract.call_view_function('decimals'),
            'total_supply': self.contract.call_view_function('totalSupply')
        }
    
    def get_balance(self, address=None):
        """Получить баланс"""
        if address is None:
            address = self.address
        
        balance = self.contract.call_view_function('balanceOf', address)
        decimals = self.contract.call_view_function('decimals')
        
        return balance / (10 ** decimals)
    
    def transfer(self, to_address, amount):
        """Перевести токены"""
        if not self.private_key:
            raise Exception("Private key required for transactions")
        
        decimals = self.contract.call_view_function('decimals')
        amount_wei = int(amount * (10 ** decimals))
        
        result = self.contract.send_transaction(
            self.private_key,
            'transfer',
            to_address,
            amount_wei
        )
        
        return result

if __name__ == "__main__":
    # Пример использования
    print("Web3.py Examples")
    print("=" * 50)
    
    # basic_operations()
    # wallet_example()
    # erc20_example()
````

## Web3.js & Ethers.js (JavaScript)

````
// ========== УСТАНОВКА ==========
/*
npm install web3
npm install ethers
npm install dotenv
*/

// ========== ETHERS.JS (РЕКОМЕНДУЕТСЯ) ==========

const { ethers } = require('ethers');
require('dotenv').config();

// ========== ПОДКЛЮЧЕНИЕ К БЛОКЧЕЙНУ ==========

class BlockchainConnector {
    constructor(providerUrl, chainId = 1) {
        // Способ 1: JSON-RPC Provider
        this.provider = new ethers.JsonRpcProvider(providerUrl, chainId);
        
        // Способ 2: Infura/Alchemy Provider
        // this.provider = new ethers.InfuraProvider('mainnet', 'YOUR-API-KEY');
        // this.provider = new ethers.AlchemyProvider('mainnet', 'YOUR-API-KEY');
        
        // Способ 3: Подключение к MetaMask (в браузере)
        // this.provider = new ethers.BrowserProvider(window.ethereum);
    }
    
    async checkConnection() {
        try {
            const network = await this.provider.getNetwork();
            console.log('✅ Connected to:', network.name);
            console.log('Chain ID:', network.chainId);
            return true;
        } catch (error) {
            console.error('❌ Connection failed:', error);
            return false;
        }
    }
}

// ========== БАЗОВЫЕ ОПЕРАЦИИ ==========

async function basicOperations() {
    // Подключение
    const provider = new ethers.JsonRpcProvider('https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY');
    
    // 1. ПОЛУЧИТЬ ПОСЛЕДНИЙ БЛОК
    const blockNumber = await provider.getBlockNumber();
    console.log('Latest block:', blockNumber);
    
    // 2. ИНФОРМАЦИЯ О БЛОКЕ
    const block = await provider.getBlock('latest');
    console.log('Block timestamp:', block.timestamp);
    console.log('Block hash:', block.hash);
    console.log('Transactions count:', block.transactions.length);
    
    // 3. ПОЛУЧИТЬ БАЛАНС
    const address = '0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045'; // Vitalik
    const balance = await provider.getBalance(address);
    const balanceEth = ethers.formatEther(balance);
    console.log(`Balance: ${balanceEth} ETH`);
    
    // 4. ЦЕНА ГАЗА
    const feeData = await provider.getFeeData();
    console.log('Gas Price:', ethers.formatUnits(feeData.gasPrice, 'gwei'), 'Gwei');
    console.log('Max Fee:', ethers.formatUnits(feeData.maxFeePerGas, 'gwei'), 'Gwei');
    
    // 5. ИНФОРМАЦИЯ О ТРАНЗАКЦИИ
    const txHash = '0x...';
    // const tx = await provider.getTransaction(txHash);
    // console.log('From:', tx.from);
    // console.log('To:', tx.to);
    // console.log('Value:', ethers.formatEther(tx.value), 'ETH');
    
    // 6. RECEIPT ТРАНЗАКЦИИ
    // const receipt = await provider.getTransactionReceipt(txHash);
    // console.log('Status:', receipt.status === 1 ? 'Success' : 'Failed');
    // console.log('Gas used:', receipt.gasUsed.toString());
}

// ========== РАБОТА С КОШЕЛЬКАМИ ==========

class WalletManager {
    // 1. СОЗДАТЬ НОВЫЙ КОШЕЛЕК
    static createWallet() {
        const wallet = ethers.Wallet.createRandom();
        return {
            address: wallet.address,
            privateKey: wallet.privateKey,
            mnemonic: wallet.mnemonic.phrase
        };
    }
    
    // 2. ИМПОРТ ПО ПРИВАТНОМУ КЛЮЧУ
    static fromPrivateKey(privateKey, provider) {
        return new ethers.Wallet(privateKey, provider);
    }
    
    // 3. ИМПОРТ ПО МНЕМОНИКЕ
    static fromMnemonic(mnemonic, provider) {
        return ethers.Wallet.fromPhrase(mnemonic, provider);
    }
    
    // 4. ПОДКЛЮЧЕНИЕ К METAMASK (в браузере)
    static async connectMetaMask() {
        if (typeof window.ethereum === 'undefined') {
            throw new Error('MetaMask not installed');
        }
        
        const provider = new ethers.BrowserProvider(window.ethereum);
        await provider.send("eth_requestAccounts", []);
        const signer = await provider.getSigner();
        
        return {
            provider,
            signer,
            address: await signer.getAddress()
        };
    }
}

// Пример использования
async function walletExample() {
    // Создать новый кошелек
    const newWallet = WalletManager.createWallet();
    console.log('New Address:', newWallet.address);
    console.log('Private Key:', newWallet.privateKey);
    console.log('Mnemonic:', newWallet.mnemonic);
    
    // ВАЖНО: Храните приватные ключи в .env файле!
    
    // Импорт существующего
    const provider = new ethers.JsonRpcProvider('http://127.0.0.1:8545');
    const wallet = WalletManager.fromPrivateKey(process.env.PRIVATE_KEY, provider);
    
    const balance = await provider.getBalance(wallet.address);
    console.log('Balance:', ethers.formatEther(balance), 'ETH');
}

// ========== ОТПРАВКА ТРАНЗАКЦИЙ ==========

class TransactionManager {
    constructor(wallet) {
        this.wallet = wallet;
        this.provider = wallet.provider;
    }
    
    // Отправить ETH
    async sendEth(toAddress, amountEth) {
        const tx = {
            to: toAddress,
            value: ethers.parseEther(amountEth.toString())
        };
        
        console.log('Sending transaction...');
        const txResponse = await this.wallet.sendTransaction(tx);
        console.log('Transaction sent! Hash:', txResponse.hash);
        
        // Ждем подтверждения
        const receipt = await txResponse.wait();
        
        if (receipt.status === 1) {
            console.log('✅ Transaction successful!');
            console.log('Block:', receipt.blockNumber);
            console.log('Gas used:', receipt.gasUsed.toString());
        } else {
            console.log('❌ Transaction failed!');
        }
        
        return receipt;
    }
    
    // Отправить с кастомным газом
    async sendEthWithGasOptions(toAddress, amountEth) {
        const feeData = await this.provider.getFeeData();
        
        const tx = {
            to: toAddress,
            value: ethers.parseEther(amountEth.toString()),
            gasLimit: 21000,
            maxFeePerGas: feeData.maxFeePerGas,
            maxPriorityFeePerGas: feeData.maxPriorityFeePerGas
        };
        
        const txResponse = await this.wallet.sendTransaction(tx);
        return await txResponse.wait();
    }
    
    // Оценить газ
    async estimateGas(toAddress, amountEth) {
        const tx = {
            to: toAddress,
            value: ethers.parseEther(amountEth.toString())
        };
        
        const gasEstimate = await this.provider.estimateGas(tx);
        console.log('Estimated gas:', gasEstimate.toString());
        
        const feeData = await this.provider.getFeeData();
        const gasCost = gasEstimate * feeData.gasPrice;
        console.log('Estimated cost:', ethers.formatEther(gasCost), 'ETH');
        
        return gasEstimate;
    }
}

// ========== РАБОТА СО СМАРТ-КОНТРАКТАМИ ==========

class ContractManager {
    constructor(address, abi, signerOrProvider) {
        this.contract = new ethers.Contract(address, abi, signerOrProvider);
        this.address = address;
    }
    
    // Вызвать view функцию (не требует газа)
    async callViewFunction(functionName, ...args) {
        try {
            const result = await this.contract[functionName](...args);
            return result;
        } catch (error) {
            console.error('Error calling function:', error);
            throw error;
        }
    }
    
    // Отправить транзакцию (изменяет состояние)
    async sendTransaction(functionName, ...args) {
        try {
            const tx = await this.contract[functionName](...args);
            console.log('Transaction sent:', tx.hash);
            
            const receipt = await tx.wait();
            console.log('Transaction confirmed in block:', receipt.blockNumber);
            
            return {
                hash: tx.hash,
                receipt: receipt
            };
        } catch (error) {
            console.error('Transaction failed:', error);
            throw error;
        }
    }
    
    // Отправить с кастомными параметрами
    async sendTransactionWithOptions(functionName, args, options = {}) {
        const tx = await this.contract[functionName](...args, options);
        return await tx.wait();
    }
    
    // Оценить газ
    async estimateGas(functionName, ...args) {
        const gasEstimate = await this.contract[functionName].estimateGas(...args);
        return gasEstimate;
    }
}

// ========== ERC-20 ПРИМЕР ==========

const ERC20_ABI = [
    "function name() view returns (string)",
    "function symbol() view returns (string)",
    "function decimals() view returns (uint8)",
    "function totalSupply() view returns (uint256)",
    "function balanceOf(address) view returns (uint256)",
    "function transfer(address to, uint256 amount) returns (bool)",
    "function approve(address spender, uint256 amount) returns (bool)",
    "function allowance(address owner, address spender) view returns (uint256)",
    "function transferFrom(address from, address to, uint256 amount) returns (bool)",
    "event Transfer(address indexed from, address indexed to, uint256 value)",
    "event Approval(address indexed owner, address indexed spender, uint256 value)"
];

async function erc20Example() {
    const provider = new ethers.JsonRpcProvider('https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY');
    
    // USDT контракт
    const usdtAddress = '0xdAC17F958D2ee523a2206206994597C13D831ec7';
    const usdt = new ContractManager(usdtAddress, ERC20_ABI, provider);
    
    // Читаем данные
    const name = await usdt.callViewFunction('name');
    const symbol = await usdt.callViewFunction('symbol');
    const decimals = await usdt.callViewFunction('decimals');
    const totalSupply = await usdt.callViewFunction('totalSupply');
    
    console.log(`Token: ${name} (${symbol})`);
    console.log(`Decimals: ${decimals}`);
    console.log(`Total Supply: ${ethers.formatUnits(totalSupply, decimals)} ${symbol}`);
    
    // Проверить баланс
    const userAddress = '0x...';
    const balance = await usdt.callViewFunction('balanceOf', userAddress);
    console.log(`Balance: ${ethers.formatUnits(balance, decimals)} ${symbol}`);
    
    // Отправить токены (требует signer)
    // const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
    // const usdtWithSigner = new ContractManager(usdtAddress, ERC20_ABI, wallet);
    // 
    // const recipient = '0x...';
    // const amount = ethers.parseUnits('100', decimals); // 100 USDT
    // 
    // await usdtWithSigner.sendTransaction('transfer', recipient, amount);
}

// ========== СЛУШАТЬ СОБЫТИЯ ==========

class EventListener {
    constructor(contract) {
        this.contract = contract;
    }
    
    // Получить прошлые события
    async getPastEvents(eventName, fromBlock = 0, toBlock = 'latest') {
        const filter = this.contract.filters[eventName]();
        const events = await this.contract.queryFilter(filter, fromBlock, toBlock);
        return events;
    }
    
    // Слушать новые события
    listenForEvents(eventName, callback) {
        const filter = this.contract.filters[eventName]();
        
        this.contract.on(filter, (...args) => {
            const event = args[args.length - 1];
            callback(event);
        });
    }
    
    // Слушать конкретное событие с фильтром
    listenWithFilter(eventName, filterParams, callback) {
        const filter = this.contract.filters[eventName](...filterParams);
        
        this.contract.on(filter, (...args) => {
            const event = args[args.length - 1];
            callback(event);
        });
    }
    
    // Остановить прослушивание
    stopListening(eventName) {
        this.contract.removeAllListeners(eventName);
    }
}

// Пример прослушивания Transfer событий
async function eventExample() {
    const provider = new ethers.JsonRpcProvider('wss://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY');
    
    const usdtAddress = '0xdAC17F958D2ee523a2206206994597C13D831ec7';
    const usdt = new ethers.Contract(usdtAddress, ERC20_ABI, provider);
    
    const listener = new EventListener(usdt);
    
    // Получить прошлые трансферы
    const pastTransfers = await listener.getPastEvents('Transfer', 18000000, 18001000);
    
    console.log(`Found ${pastTransfers.length} transfers`);
    pastTransfers.slice(0, 5).forEach(event => {
        console.log('---');
        console.log('From:', event.args.from);
        console.log('To:', event.args.to);
        console.log('Amount:', ethers.formatUnits(event.args.value, 6));
        console.log('Block:', event.blockNumber);
    });
    
    // Слушать новые трансферы
    console.log('\nListening for new transfers...');
    listener.listenForEvents('Transfer', (event) => {
        console.log('New Transfer!');
        console.log('From:', event.args.from);
        console.log('To:', event.args.to);
        console.log('Amount:', ethers.formatUnits(event.args.value, 6));
    });
}

// ========== ДЕПЛОЙ КОНТРАКТА ==========

async function deployContract(wallet, bytecode, abi, constructorArgs = []) {
    const factory = new ethers.ContractFactory(abi, bytecode, wallet);
    
    console.log('Deploying contract...');
    const contract = await factory.deploy(...constructorArgs);
    
    console.log('Deploy transaction sent:', contract.deploymentTransaction().hash);
    console.log('Waiting for confirmation...');
    
    await contract.waitForDeployment();
    
    const address = await contract.getAddress();
    console.log('✅ Contract deployed at:', address);
    
    return contract;
}

// ========== ПОДПИСАНИЕ СООБЩЕНИЙ ==========

class MessageSigner {
    constructor(wallet) {
        this.wallet = wallet;
    }
    
    // Подписать сообщение
    async signMessage(message) {
        const signature = await this.wallet.signMessage(message);
        return signature;
    }
    
    // Подписать типизированные данные (EIP-712)
    async signTypedData(domain, types, value) {
        const signature = await this.wallet.signTypedData(domain, types, value);
        return signature;
    }
    
    // Проверить подпись
    static verifyMessage(message, signature) {
        const address = ethers.verifyMessage(message, signature);
        return address;
    }
    
    // Проверить типизированную подпись
    static verifyTypedData(domain, types, value, signature) {
        const address = ethers.verifyTypedData(domain, types, value, signature);
        return address;
    }
}

// Пример подписания
async function signingExample() {
    const wallet = ethers.Wallet.createRandom();
    const signer = new MessageSigner(wallet);
    
    // Подписать простое сообщение
    const message = "Hello, Blockchain!";
    const signature = await signer.signMessage(message);
    console.log('Signature:', signature);
    
    // Проверить подпись
    const recoveredAddress = MessageSigner.verifyMessage(message, signature);
    console.log('Signer address:', recoveredAddress);
    console.log('Match:', recoveredAddress === wallet.address);
    
    // EIP-712 типизированная подпись
    const domain = {
        name: 'MyDApp',
        version: '1',
        chainId: 1,
        verifyingContract: '0x...'
    };
    
    const types = {
        Person: [
            { name: 'name', type: 'string' },
            { name: 'wallet', type: 'address' }
        ]
    };
    
    const value = {
        name: 'Alice',
        wallet: '0x...'
    };
    
    const typedSignature = await signer.signTypedData(domain, types, value);
    console.log('Typed Signature:', typedSignature);
}

// ========== MULTICALL (BATCH ЗАПРОСЫ) ==========

async function multicallExample() {
    const provider = new ethers.JsonRpcProvider('https://eth-mainnet.g.alchemy.com/v2/YOUR-API-KEY');
    
    // Массив контрактов
    const tokens = [
        '0xdAC17F958D2ee523a2206206994597C13D831ec7', // USDT
        '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48', // USDC
        '0x6B175474E89094C44Da98b954EedeAC495271d0F'  // DAI
    ];
    
    // Параллельные запросы
    const balances = await Promise.all(
        tokens.map(async (tokenAddress) => {
            const token = new ethers.Contract(tokenAddress, ERC20_ABI, provider);
            const symbol = await token.symbol();
            const balance = await token.balanceOf('0x...');
            return { symbol, balance };
        })
    );
    
    balances.forEach(({ symbol, balance }) => {
        console.log(`${symbol}: ${ethers.formatUnits(balance, 18)}`);
    });
}

// ========== УТИЛИТЫ ==========

class Utils {
    // Конвертация единиц
    static toWei(amount, unit = 'ether') {
        return ethers.parseUnits(amount.toString(), unit);
    }
    
    static fromWei(amount, unit = 'ether') {
        return ethers.formatUnits(amount, unit);
    }
    
    // Хеширование
    static keccak256(data) {
        return ethers.keccak256(ethers.toUtf8Bytes(data));
    }
    
    // Проверка адреса
    static isAddress(address) {
        return ethers.isAddress(address);
    }
    
    // Checksum адрес
    static getAddress(address) {
        return ethers.getAddress(address);
    }
    
    // Создать случайные bytes
    static randomBytes(length) {
        return ethers.randomBytes(length);
    }
    
    // Конкатенация bytes
    static concat(arrays) {
        return ethers.concat(arrays);
    }
}

// ========== REACT ИНТЕГРАЦИЯ ==========

// Пример React хука для Web3
/*
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

function useWeb3() {
    const [provider, setProvider] = useState(null);
    const [signer, setSigner] = useState(null);
    const [address, setAddress] = useState(null);
    const [connected, setConnected] = useState(false);
    
    const connect = async () => {
        if (typeof window.ethereum === 'undefined') {
            alert('Please install MetaMask!');
            return;
        }
        
        try {
            const provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            const signer = await provider.getSigner();
            const address = await signer.getAddress();
            
            setProvider(provider);
            setSigner(signer);
            setAddress(address);
            setConnected(true);
        } catch (error) {
            console.error('Connection failed:', error);
        }
    };
    
    const disconnect = () => {
        setProvider(null);
        setSigner(null);
        setAddress(null);
        setConnected(false);
    };
    
    return { provider, signer, address, connected, connect, disconnect };
}

// Использование в компоненте
function App() {
    const { address, connected, connect, disconnect } = useWeb3();
    
    return (
        <div>
            {!connected ? (
                <button onClick={connect}>Connect Wallet</button>
            ) : (
                <div>
                    <p>Connected: {address}</p>
                    <button onClick={disconnect}>Disconnect</button>
                </div>
            )}
        </div>
    );
}
*/

// ========== ЭКСПОРТ ==========

module.exports = {
    BlockchainConnector,
    WalletManager,
    TransactionManager,
    ContractManager,
    EventListener,
    MessageSigner,
    Utils
};

// Примеры запуска
if (require.main === module) {
    console.log('Ethers.js Examples');
    console.log('=' + '='.repeat(50));
    
    // basicOperations();
    // walletExample();
    // erc20Example();
    // eventExample();
}
````

## DApp с фронтендом и бэкендом

````
# 🚀 Полноценный DApp: NFT Marketplace

## Архитектура проекта

```
nft-marketplace/
├── contracts/          # Solidity смарт-контракты
│   ├── NFT.sol
│   └── Marketplace.sol
├── backend/           # Python/Node.js backend
│   ├── server.py
│   └── indexer.py
├── frontend/          # React frontend
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── utils/
│   └── package.json
├── scripts/           # Скрипты деплоя
│   ├── deploy.py
│   └── deploy.js
└── tests/            # Тесты
    ├── test_contracts.py
    └── test_contracts.js
```

---

## 📝 1. Backend API (Python + Flask)

```python
# backend/server.py

from flask import Flask, jsonify, request
from flask_cors import CORS
from web3 import Web3
from eth_account import Account
import json
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
CORS(app)

# Подключение к блокчейну
w3 = Web3(Web3.HTTPProvider(os.getenv('RPC_URL')))

# Загрузка ABI контрактов
with open('abi/NFT.json') as f:
    NFT_ABI = json.load(f)

with open('abi/Marketplace.json') as f:
    MARKETPLACE_ABI = json.load(f)

# Адреса контрактов
NFT_ADDRESS = os.getenv('NFT_CONTRACT_ADDRESS')
MARKETPLACE_ADDRESS = os.getenv('MARKETPLACE_CONTRACT_ADDRESS')

# Контракты
nft_contract = w3.eth.contract(address=NFT_ADDRESS, abi=NFT_ABI)
marketplace_contract = w3.eth.contract(address=MARKETPLACE_ADDRESS, abi=MARKETPLACE_ABI)

# ========== API ENDPOINTS ==========

@app.route('/api/nfts', methods=['GET'])
def get_all_nfts():
    """Получить все NFT"""
    try:
        total_supply = nft_contract.functions.totalSupply().call()
        nfts = []
        
        for token_id in range(total_supply):
            owner = nft_contract.functions.ownerOf(token_id).call()
            token_uri = nft_contract.functions.tokenURI(token_id).call()
            
            nfts.append({
                'tokenId': token_id,
                'owner': owner,
                'tokenURI': token_uri
            })
        
        return jsonify({'success': True, 'nfts': nfts})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/api/nft/<int:token_id>', methods=['GET'])
def get_nft(token_id):
    """Получить информацию об NFT"""
    try:
        owner = nft_contract.functions.ownerOf(token_id).call()
        token_uri = nft_contract.functions.tokenURI(token_id).call()
        
        # Проверить, выставлен ли на продажу
        listing = marketplace_contract.functions.listings(token_id).call()
        
        return jsonify({
            'success': True,
            'tokenId': token_id,
            'owner': owner,
            'tokenURI': token_uri,
            'listing': {
                'price': listing[0],
                'seller': listing[1],
                'active': listing[2]
            }
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/api/marketplace/listings', methods=['GET'])
def get_listings():
    """Получить все активные листинги"""
    try:
        # Получаем события Listing
        event_filter = marketplace_contract.events.Listed.create_filter(
            fromBlock=0,
            toBlock='latest'
        )
        
        events = event_filter.get_all_entries()
        active_listings = []
        
        for event in events:
            token_id = event['args']['tokenId']
            listing = marketplace_contract.functions.listings(token_id).call()
            
            if listing[2]:  # если active
                nft_uri = nft_contract.functions.tokenURI(token_id).call()
                active_listings.append({
                    'tokenId': token_id,
                    'price': listing[0],
                    'seller': listing[1],
                    'tokenURI': nft_uri
                })
        
        return jsonify({'success': True, 'listings': active_listings})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/api/user/<address>/nfts', methods=['GET'])
def get_user_nfts(address):
    """Получить NFT пользователя"""
    try:
        balance = nft_contract.functions.balanceOf(address).call()
        user_nfts = []
        
        total_supply = nft_contract.functions.totalSupply().call()
        
        for token_id in range(total_supply):
            owner = nft_contract.functions.ownerOf(token_id).call()
            if owner.lower() == address.lower():
                token_uri = nft_contract.functions.tokenURI(token_id).call()
                user_nfts.append({
                    'tokenId': token_id,
                    'tokenURI': token_uri
                })
        
        return jsonify({'success': True, 'nfts': user_nfts})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

@app.route('/api/stats', methods=['GET'])
def get_stats():
    """Получить статистику маркетплейса"""
    try:
        total_supply = nft_contract.functions.totalSupply().call()
        
        # Подсчет активных листингов
        active_count = 0
        total_volume = 0
        
        # Получаем события Sold для подсчета объема
        sold_events = marketplace_contract.events.Sold.create_filter(
            fromBlock=0,
            toBlock='latest'
        ).get_all_entries()
        
        for event in sold_events:
            total_volume += event['args']['price']
        
        return jsonify({
            'success': True,
            'stats': {
                'totalNFTs': total_supply,
                'totalVolume': total_volume,
                'activeListings': active_count
            }
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

---

## 🎨 2. Frontend (React + Ethers.js)

```javascript
// frontend/src/App.jsx

import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import NFTCard from './components/NFTCard';
import Navbar from './components/Navbar';
import './App.css';

// Import ABIs
import NFT_ABI from './contracts/NFT.json';
import MARKETPLACE_ABI from './contracts/Marketplace.json';

const NFT_ADDRESS = process.env.REACT_APP_NFT_ADDRESS;
const MARKETPLACE_ADDRESS = process.env.REACT_APP_MARKETPLACE_ADDRESS;

function App() {
    const [provider, setProvider] = useState(null);
    const [signer, setSigner] = useState(null);
    const [account, setAccount] = useState(null);
    const [nftContract, setNftContract] = useState(null);
    const [marketplaceContract, setMarketplaceContract] = useState(null);
    const [listings, setListings] = useState([]);
    const [userNFTs, setUserNFTs] = useState([]);
    const [loading, setLoading] = useState(false);

    // Подключение кошелька
    const connectWallet = async () => {
        try {
            if (typeof window.ethereum === 'undefined') {
                alert('Please install MetaMask!');
                return;
            }

            const provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            const signer = await provider.getSigner();
            const address = await signer.getAddress();

            setProvider(provider);
            setSigner(signer);
            setAccount(address);

            // Инициализация контрактов
            const nft = new ethers.Contract(NFT_ADDRESS, NFT_ABI, signer);
            const marketplace = new ethers.Contract(MARKETPLACE_ADDRESS, MARKETPLACE_ABI, signer);

            setNftContract(nft);
            setMarketplaceContract(marketplace);

            console.log('Connected:', address);
        } catch (error) {
            console.error('Connection failed:', error);
        }
    };

    // Загрузка листингов
    const loadListings = async () => {
        if (!marketplaceContract) return;

        setLoading(true);
        try {
            const response = await fetch('http://localhost:5000/api/marketplace/listings');
            const data = await response.json();

            if (data.success) {
                setListings(data.listings);
            }
        } catch (error) {
            console.error('Error loading listings:', error);
        }
        setLoading(false);
    };

    // Загрузка NFT пользователя
    const loadUserNFTs = async () => {
        if (!account) return;

        try {
            const response = await fetch(`http://localhost:5000/api/user/${account}/nfts`);
            const data = await response.json();

            if (data.success) {
                setUserNFTs(data.nfts);
            }
        } catch (error) {
            console.error('Error loading user NFTs:', error);
        }
    };

    // Минт NFT
    const mintNFT = async (tokenURI) => {
        if (!nftContract) return;

        try {
            setLoading(true);
            const tx = await nftContract.mint(account, tokenURI);
            await tx.wait();
            alert('NFT minted successfully!');
            loadUserNFTs();
        } catch (error) {
            console.error('Mint failed:', error);
            alert('Mint failed: ' + error.message);
        }
        setLoading(false);
    };

    // Выставить на продажу
    const listNFT = async (tokenId, price) => {
        if (!marketplaceContract || !nftContract) return;

        try {
            setLoading(true);

            // 1. Approve marketplace
            const approveTx = await nftContract.approve(MARKETPLACE_ADDRESS, tokenId);
            await approveTx.wait();

            // 2. List на marketplace
            const priceWei = ethers.parseEther(price.toString());
            const listTx = await marketplaceContract.listNFT(NFT_ADDRESS, tokenId, priceWei);
            await listTx.wait();

            alert('NFT listed successfully!');
            loadListings();
            loadUserNFTs();
        } catch (error) {
            console.error('Listing failed:', error);
            alert('Listing failed: ' + error.message);
        }
        setLoading(false);
    };

    // Купить NFT
    const buyNFT = async (tokenId, price) => {
        if (!marketplaceContract) return;

        try {
            setLoading(true);
            const tx = await marketplaceContract.buyNFT(tokenId, { value: price });
            await tx.wait();
            alert('NFT purchased successfully!');
            loadListings();
            loadUserNFTs();
        } catch (error) {
            console.error('Purchase failed:', error);
            alert('Purchase failed: ' + error.message);
        }
        setLoading(false);
    };

    useEffect(() => {
        if (marketplaceContract) {
            loadListings();
        }
    }, [marketplaceContract]);

    useEffect(() => {
        if (account) {
            loadUserNFTs();
        }
    }, [account]);

    return (
        <div className="App">
            <Navbar 
                account={account} 
                connectWallet={connectWallet}
            />

            <div className="container">
                {/* Секция минта */}
                <section className="mint-section">
                    <h2>Mint NFT</h2>
                    <button 
                        onClick={() => mintNFT('ipfs://...')}
                        disabled={loading || !account}
                    >
                        {loading ? 'Minting...' : 'Mint NFT'}
                    </button>
                </section>

                {/* Мои NFT */}
                <section className="my-nfts">
                    <h2>My NFTs</h2>
                    <div className="nft-grid">
                        {userNFTs.map(nft => (
                            <NFTCard
                                key={nft.tokenId}
                                nft={nft}
                                onList={listNFT}
                                owned={true}
                            />
                        ))}
                    </div>
                </section>

                {/* Marketplace листинги */}
                <section className="marketplace">
                    <h2>Marketplace</h2>
                    <div className="nft-grid">
                        {listings.map(listing => (
                            <NFTCard
                                key={listing.tokenId}
                                nft={listing}
                                onBuy={buyNFT}
                                owned={false}
                            />
                        ))}
                    </div>
                </section>
            </div>
        </div>
    );
}

export default App;
```

---

## 🔧 3. Custom React Hooks

```javascript
// frontend/src/hooks/useWeb3.js

import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

export const useWeb3 = () => {
    const [provider, setProvider] = useState(null);
    const [signer, setSigner] = useState(null);
    const [account, setAccount] = useState(null);
    const [chainId, setChainId] = useState(null);
    const [balance, setBalance] = useState('0');

    const connect = async () => {
        if (typeof window.ethereum === 'undefined') {
            throw new Error('MetaMask not installed');
        }

        const provider = new ethers.BrowserProvider(window.ethereum);
        await provider.send("eth_requestAccounts", []);
        const signer = await provider.getSigner();
        const address = await signer.getAddress();
        const network = await provider.getNetwork();
        const balance = await provider.getBalance(address);

        setProvider(provider);
        setSigner(signer);
        setAccount(address);
        setChainId(Number(network.chainId));
        setBalance(ethers.formatEther(balance));
    };

    const disconnect = () => {
        setProvider(null);
        setSigner(null);
        setAccount(null);
        setChainId(null);
        setBalance('0');
    };

    // Слушать смену аккаунта
    useEffect(() => {
        if (window.ethereum) {
            window.ethereum.on('accountsChanged', (accounts) => {
                if (accounts.length > 0) {
                    connect();
                } else {
                    disconnect();
                }
            });

            window.ethereum.on('chainChanged', () => {
                window.location.reload();
            });
        }

        return () => {
            if (window.ethereum) {
                window.ethereum.removeAllListeners('accountsChanged');
                window.ethereum.removeAllListeners('chainChanged');
            }
        };
    }, []);

    return {
        provider,
        signer,
        account,
        chainId,
        balance,
        connect,
        disconnect
    };
};
```

```javascript
// frontend/src/hooks/useContract.js

import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

export const useContract = (address, abi, signer) => {
    const [contract, setContract] = useState(null);

    useEffect(() => {
        if (address && abi && signer) {
            const contract = new ethers.Contract(address, abi, signer);
            setContract(contract);
        }
    }, [address, abi, signer]);

    return contract;
};
```

---

## 🧪 4. Скрипты деплоя

```python
# scripts/deploy.py (Brownie/Ape)

from ape import accounts, project

def deploy():
    # Получить аккаунт
    deployer = accounts.load("deployer")
    
    # Деплой NFT контракта
    nft = project.NFT.deploy(
        "MyNFT",
        "MNFT",
        sender=deployer
    )
    print(f"NFT deployed at: {nft.address}")
    
    # Деплой Marketplace контракта
    marketplace = project.Marketplace.deploy(sender=deployer)
    print(f"Marketplace deployed at: {marketplace.address}")
    
    return nft, marketplace

if __name__ == "__main__":
    deploy()
```

```javascript
// scripts/deploy.js (Hardhat)

const hre = require("hardhat");

async function main() {
    const [deployer] = await hre.ethers.getSigners();
    console.log("Deploying with:", deployer.address);

    // Deploy NFT
    const NFT = await hre.ethers.getContractFactory("NFT");
    const nft = await NFT.deploy("MyNFT", "MNFT");
    await nft.waitForDeployment();
    console.log("NFT deployed to:", await nft.getAddress());

    // Deploy Marketplace
    const Marketplace = await hre.ethers.getContractFactory("Marketplace");
    const marketplace = await Marketplace.deploy();
    await marketplace.waitForDeployment();
    console.log("Marketplace deployed to:", await marketplace.getAddress());
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

---

## 📋 Резюме

**Стек технологий:**
- **Смарт-контракты:** Solidity
- **Backend:** Python (Flask) / Node.js (Express)
- **Frontend:** React + Ethers.js
- **Database:** PostgreSQL / MongoDB (опционально)
- **IPFS:** для хранения metadata
- **Deployment:** Hardhat / Foundry

**Ключевые концепции:**
1. ✅ Подключение к блокчейну (Web3.py, Ethers.js)
2. ✅ Работа с кошельками (создание, импорт)
3. ✅ Отправка транзакций
4. ✅ Взаимодействие со смарт-контрактами
5. ✅ Прослушивание событий
6. ✅ React hooks для Web3
7. ✅ Backend API для индексации
8. ✅ Полноценный DApp

Теперь у вас есть полный набор для создания Web3 приложений!
````

# 🎯 Web3 Development: Best Practices & Чеклист

## 📚 Сравнение библиотек

````
| Функция           | Web3.py (Python) | Web3.js  | Ethers.js     |
|-------------------|------------------|----------|---------------|
| **Популярность**  | ⭐⭐⭐          | ⭐⭐⭐⭐| ⭐⭐⭐⭐⭐   |
| **Документация**  | Хорошая          | Средняя  | Отличная      |
| **Размер**        | Средний          | Большой  | Маленький     |
| **TypeScript**    | ❌               | Частично | ✅             |
| **Использование** | Backend/Scripts  | Legacy   | Рекомендуется |
| **ENS Support**   | ✅               | ✅       | ✅             |
| **Подписание**    | ✅               | ✅       | ✅             |

**Рекомендация:** 
- **Frontend**: Ethers.js (v6)
- **Backend**: Web3.py (Python) или Ethers.js (Node.js)
- **Scripts**: Любая

---

## 🔐 Безопасность

### 1. Хранение приватных ключей

```bash
# ❌ НИКОГДА НЕ ДЕЛАЙТЕ ТАК
const privateKey = "0x1234..."; // в коде

# ✅ ПРАВИЛЬНО: .env файл
PRIVATE_KEY=0x1234...
RPC_URL=https://...
CONTRACT_ADDRESS=0x...
```

```python
# Python
from dotenv import load_dotenv
import os

load_dotenv()
private_key = os.getenv('PRIVATE_KEY')
```

```javascript
// JavaScript
require('dotenv').config();
const privateKey = process.env.PRIVATE_KEY;
```

### 2. Проверка входных данных

```javascript
// ❌ ПЛОХО
async function transfer(to, amount) {
    await token.transfer(to, amount);
}

// ✅ ХОРОШО
async function transfer(to, amount) {
    if (!ethers.isAddress(to)) {
        throw new Error('Invalid address');
    }
    if (amount <= 0) {
        throw new Error('Amount must be positive');
    }
    try {
        const tx = await token.transfer(to, amount);
        return await tx.wait();
    } catch (error) {
        console.error('Transfer failed:', error);
        throw error;
    }
}
```

### 3. Rate Limiting для RPC

```python
import time
from functools import wraps

def rate_limit(calls_per_second=5):
    min_interval = 1.0 / calls_per_second
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                time.sleep(min_interval - elapsed)
            result = func(*args, **kwargs)
            last_called[0] = time.time()
            return result
        return wrapper
    return decorator

@rate_limit(calls_per_second=5)
def get_balance(address):
    return w3.eth.get_balance(address)
```

---

## ⚡ Оптимизация производительности

### 1. Батчинг запросов

```javascript
// ❌ ПЛОХО: Последовательные запросы
async function getBalances(addresses) {
    const balances = [];
    for (const addr of addresses) {
        const balance = await provider.getBalance(addr);
        balances.push(balance);
    }
    return balances;
}

// ✅ ХОРОШО: Параллельные запросы
async function getBalances(addresses) {
    return await Promise.all(
        addresses.map(addr => provider.getBalance(addr))
    );
}
```

### 2. Кеширование

```python
from functools import lru_cache
import time

@lru_cache(maxsize=128)
def get_token_info(token_address):
    """Кеширование метаданных токена"""
    contract = w3.eth.contract(address=token_address, abi=ERC20_ABI)
    return {
        'name': contract.functions.name().call(),
        'symbol': contract.functions.symbol().call(),
        'decimals': contract.functions.decimals().call()
    }

# С временным кешем
class TimedCache:
    def __init__(self, ttl_seconds=60):
        self.cache = {}
        self.ttl = ttl_seconds
    
    def get(self, key):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if time.time() - timestamp < self.ttl:
                return value
            del self.cache[key]
        return None
    
    def set(self, key, value):
        self.cache[key] = (value, time.time())
```

### 3. Pagination для событий

```javascript
// ❌ ПЛОХО: Все события сразу
const events = await contract.queryFilter('Transfer', 0, 'latest');

// ✅ ХОРОШО: По блокам
async function getEventsInChunks(contract, eventName, fromBlock, toBlock, chunkSize = 10000) {
    const events = [];
    
    for (let i = fromBlock; i <= toBlock; i += chunkSize) {
        const endBlock = Math.min(i + chunkSize - 1, toBlock);
        const chunk = await contract.queryFilter(eventName, i, endBlock);
        events.push(...chunk);
    }
    
    return events;
}
```

---

## 🔄 Обработка ошибок

### 1. Повторные попытки

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay * (2 ** attempt))  # Exponential backoff
        return wrapper
    return decorator

@retry(max_attempts=3, delay=1)
def send_transaction(tx):
    return w3.eth.send_raw_transaction(tx)
```

### 2. Graceful degradation

```javascript
async function getGasPrice() {
    try {
        // Попытка получить от оракула
        const gasPrice = await gasOracle.getGasPrice();
        return gasPrice;
    } catch (error) {
        console.warn('Gas oracle failed, using provider gas price');
        try {
            // Fallback на провайдер
            return await provider.getFeeData();
        } catch (error2) {
            console.error('All gas price sources failed');
            // Hardcoded fallback
            return {
                gasPrice: ethers.parseUnits('50', 'gwei'),
                maxFeePerGas: ethers.parseUnits('100', 'gwei'),
                maxPriorityFeePerGas: ethers.parseUnits('2', 'gwei')
            };
        }
    }
}
```

---

## 📊 Мониторинг и логирование

### 1. Структурированное логирование

```python
import logging
import json

class Web3Logger:
    def __init__(self):
        self.logger = logging.getLogger('web3_app')
        handler = logging.StreamHandler()
        handler.setFormatter(
            logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
        )
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.INFO)
    
    def log_transaction(self, tx_hash, status, gas_used):
        self.logger.info(json.dumps({
            'event': 'transaction',
            'tx_hash': tx_hash,
            'status': status,
            'gas_used': gas_used
        }))
    
    def log_error(self, error, context):
        self.logger.error(json.dumps({
            'event': 'error',
            'error': str(error),
            'context': context
        }))
```

### 2. Метрики производительности

```javascript
class PerformanceMonitor {
    constructor() {
        this.metrics = {};
    }
    
    async measure(name, fn) {
        const start = Date.now();
        try {
            const result = await fn();
            const duration = Date.now() - start;
            
            if (!this.metrics[name]) {
                this.metrics[name] = {
                    count: 0,
                    totalTime: 0,
                    avgTime: 0
                };
            }
            
            this.metrics[name].count++;
            this.metrics[name].totalTime += duration;
            this.metrics[name].avgTime = 
                this.metrics[name].totalTime / this.metrics[name].count;
            
            return result;
        } catch (error) {
            console.error(`Error in ${name}:`, error);
            throw error;
        }
    }
    
    getMetrics() {
        return this.metrics;
    }
}

// Использование
const monitor = new PerformanceMonitor();

const balance = await monitor.measure('getBalance', async () => {
    return await provider.getBalance(address);
});

console.log(monitor.getMetrics());
```

---

## 🧪 Тестирование

### 1. Unit тесты (Python)

```python
import pytest
from web3 import Web3

@pytest.fixture
def w3():
    return Web3(Web3.HTTPProvider('http://127.0.0.1:8545'))

@pytest.fixture
def accounts(w3):
    return w3.eth.accounts

def test_send_transaction(w3, accounts):
    tx = {
        'from': accounts[0],
        'to': accounts[1],
        'value': w3.to_wei(1, 'ether'),
        'gas': 21000,
        'gasPrice': w3.eth.gas_price
    }
    
    tx_hash = w3.eth.send_transaction(tx)
    receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
    
    assert receipt['status'] == 1
```

### 2. Integration тесты (JavaScript)

```javascript
const { expect } = require('chai');
const { ethers } = require('hardhat');

describe('Token', function() {
    let token;
    let owner;
    let addr1;
    
    beforeEach(async function() {
        [owner, addr1] = await ethers.getSigners();
        const Token = await ethers.getContractFactory('Token');
        token = await Token.deploy(1000000);
    });
    
    it('Should transfer tokens', async function() {
        await token.transfer(addr1.address, 100);
        expect(await token.balanceOf(addr1.address)).to.equal(100);
    });
    
    it('Should fail if sender has insufficient balance', async function() {
        await expect(
            token.connect(addr1).transfer(owner.address, 100)
        ).to.be.revertedWith('Insufficient balance');
    });
});
```

---

## 📝 Документация кода

```javascript
/**
 * Transfers tokens to a recipient
 * 
 * @param {string} recipient - The recipient's address
 * @param {number|string} amount - Amount of tokens to transfer
 * @returns {Promise<TransactionReceipt>} Transaction receipt
 * @throws {Error} If address is invalid or amount is negative
 * 
 * @example
 * const receipt = await transferTokens('0x...', '100');
 * console.log('Transfer successful:', receipt.hash);
 */
async function transferTokens(recipient, amount) {
    if (!ethers.isAddress(recipient)) {
        throw new Error('Invalid recipient address');
    }
    
    const tx = await token.transfer(recipient, ethers.parseEther(amount));
    return await tx.wait();
}
```

---

## ✅ Чеклист перед продакшеном

### Смарт-контракты
- [ ] Аудит кода
- [ ] Тесты (минимум 80% покрытие)
- [ ] Проверка на известные уязвимости
- [ ] Тестирование на testnet
- [ ] Verify контракта на Etherscan
- [ ] Документация для пользователей

### Backend/Frontend
- [ ] Обработка всех ошибок
- [ ] Rate limiting для API
- [ ] Валидация входных данных
- [ ] Безопасное хранение ключей
- [ ] Логирование и мониторинг
- [ ] Резервные RPC провайдеры
- [ ] CORS правильно настроен
- [ ] HTTPS для продакшена

### Безопасность
- [ ] Приватные ключи в .env
- [ ] .gitignore для чувствительных файлов
- [ ] Multisig для критичных операций
- [ ] Timelock для важных изменений
- [ ] Проверка всех внешних вводов
- [ ] Защита от CSRF/XSS

### Производительность
- [ ] Кеширование данных
- [ ] Батчинг запросов
- [ ] Pagination для больших списков
- [ ] Оптимизация газа
- [ ] CDN для статики
- [ ] Compression для API

### UX
- [ ] Loading индикаторы
- [ ] Понятные error сообщения
- [ ] Подтверждения транзакций
- [ ] Mobile responsive
- [ ] Поддержка разных кошельков
- [ ] Transaction history

---

## 🛠️ Полезные инструменты

**Development:**
- Hardhat / Foundry - разработка
- Remix - быстрое тестирование
- Tenderly - debugging
- Etherscan - verify контрактов

**Testing:**
- Ganache - локальный блокчейн
- Hardhat Network - встроенная сеть
- Fork Mainnet - тестирование с реальными данными

**Monitoring:**
- Etherscan API - данные блокчейна
- The Graph - индексация событий
- Alchemy/Infura Dashboard - метрики
- Sentry - error tracking

**Security:**
- Slither - статический анализ
- Mythril - security scanner
- OpenZeppelin Defender - автоматизация
- Certora - formal verification

---

## 📚 Дополнительные ресурсы

**Документация:**
- https://docs.ethers.org/
- https://web3py.readthedocs.io/
- https://docs.openzeppelin.com/

**Tutorials:**
- https://ethereum.org/en/developers/
- https://hardhat.org/tutorial
- https://docs.alchemy.com/

**Communities:**
- Ethereum Stack Exchange
- Reddit r/ethdev
- Discord серверы проектов

---

## 🎯 Roadmap дальнейшего изучения

1. **Углубленно:**
   - Layer 2 (Optimism, Arbitrum, zkSync)
   - MEV и Flashbots
   - Account Abstraction (ERC-4337)
   - Cross-chain bridges

2. **Специализация:**
   - DeFi protocols (Uniswap, Aave, Compound)
   - NFT platforms
   - DAO governance
   - Gaming (Unity + Web3)

3. **Инфраструктура:**
   - Running Ethereum node
   - Indexing с The Graph
   - IPFS integration
   - Oracle development

Теперь у вас есть все для создания профессиональных Web3 приложений! 🚀
````
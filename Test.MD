### Pytest
````
    pip install pytest
````
#### Пример: Мок функции
````
    from unittest.mock import Mock
    
    # Создаем мок-объект
    mock_function = Mock()
    
    # Задаем возвращаемое значение
    mock_function.return_value = 42
    
    # Вызываем мок
    result = mock_function()
    
    print(result)  # Вывод: 42
    print(mock_function.called)  # Вывод: True (функция была вызвана)
````
#### Использование patch для подмены объектов
Пример: Мок внешнего API
````
    import requests
    
    def get_user_data(user_id):
        response = requests.get(f"https://api.example.com/users/{user_id}")
        return response.json()
````
Чтобы протестировать эту функцию без реального запроса, используйте patch:
````
    from unittest.mock import patch
    import pytest
    
    @patch('requests.get')  # Подменяем requests.get
    def test_get_user_data(mock_get):
        # Настраиваем мок
        mock_get.return_value.json.return_value = {'id': 1, 'name': 'Alice'}
    
        # Вызываем функцию
        result = get_user_data(1)
    
        # Проверяем результат
        assert result == {'id': 1, 'name': 'Alice'}
        mock_get.assert_called_once_with('https://api.example.com/users/1')
````
#### Работа с MagicMock
````
    from unittest.mock import MagicMock
    
    # Создаем MagicMock
    mock = MagicMock()
    
    # Настраиваем поведение магического метода
    mock.__str__.return_value = "Mocked object"
    mock.__len__.return_value = 5
    
    print(str(mock))  # Вывод: Mocked object
    print(len(mock))  # Вывод: 5
````
#### Проверка вызовов
````
    from unittest.mock import Mock
    
    mock = Mock()
    mock(1, 2, x=3)
    
    mock.assert_called_with(1, 2, x=3)
    print(mock.call_count)  # Вывод: 1
````
````
    from unittest.mock import Mock
    
    mock = Mock()
    mock(1, 2, x=3)
    
    mock.assert_called_with(1, 2, x=3)
    print(mock.call_count)  # Вывод: 1
````
#### Мок методов класса
````
    from unittest.mock import patch
    
    class MyClass:
        def my_method(self):
            return "Original"
    
    @patch('__main__.MyClass.my_method')
    def test_my_method(mock_method):
        mock_method.return_value = "Mocked"
        obj = MyClass()
        result = obj.my_method()
        assert result == "Mocked"
    
    test_my_method()
````

---
name: python-unit-tests
description: Write comprehensive unit tests for Python applications using pytest with async support. Use this skill when the user wants to test Python functions, classes, Django views/models, pandas operations, or set up Python testing infrastructure. Trigger for requests like "write tests for my Python code", "test this Django model", "pytest for this function", "set up Python testing", "TDD for Python", or any mention of testing Python code, pytest, Django tests, or pandas testing.
---

# Python Unit Tests with pytest

This skill helps you write maintainable, comprehensive unit tests for Python applications using pytest, with support for async operations, Django, and pandas testing, optimized for TDD workflows.

## Core Testing Philosophy

1. **Test behavior, not implementation** - Focus on inputs and outputs
2. **Isolated units** - Mock external dependencies (databases, APIs, file systems)
3. **Clear failure messages** - Descriptive test names and assertions
4. **Edge cases first** - Cover error conditions, boundary values, empty inputs
5. **Fast execution** - Use fixtures and mocks to avoid slow operations

## Setup & Configuration

### Install Dependencies

```bash
# Core testing
pip install pytest pytest-cov pytest-asyncio

# Django testing (if using Django)
pip install pytest-django

# Additional useful plugins
pip install pytest-mock pytest-xdist  # mocking and parallel execution
```

### pytest Configuration (`pytest.ini` or `pyproject.toml`)

**pytest.ini:**
```ini
[tool:pytest]
testpaths = tests
python_files = test_*.py *_test.py
python_classes = Test*
python_functions = test_*
asyncio_mode = auto
addopts = 
    --strict-markers
    --cov=src
    --cov-report=html
    --cov-report=term
    --cov-report=xml
    -v
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
    unit: marks tests as unit tests
```

**pyproject.toml:**
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
asyncio_mode = "auto"
addopts = [
    "--strict-markers",
    "--cov=src",
    "--cov-report=html",
    "--cov-report=term",
    "--cov-report=xml",
    "-v"
]
markers = [
    "slow: marks tests as slow",
    "integration: marks tests as integration tests",
    "unit: marks tests as unit tests"
]

[tool.coverage.run]
omit = [
    "*/tests/*",
    "*/migrations/*",
    "*/__init__.py",
    "*/settings/*",
]
```

### Django Configuration (`conftest.py` in tests directory)

```python
import pytest
from django.conf import settings

pytest_plugins = ['pytest_django']

@pytest.fixture(scope='session')
def django_db_setup():
    settings.DATABASES['default'] = {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',
    }
```

### Directory Structure

```
project/
├── src/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   └── utils.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Shared fixtures
│   ├── test_models.py
│   ├── test_views.py
│   └── test_utils.py
├── pytest.ini
└── requirements.txt
```

## Test Structure & Patterns

### Test File Template

```python
import pytest
from unittest.mock import Mock, patch, MagicMock
from src.module import function_to_test

class TestFunctionName:
    """Test suite for function_name"""
    
    def test_basic_functionality(self):
        """Should return expected output for valid input"""
        result = function_to_test("valid_input")
        assert result == "expected_output"
    
    def test_edge_case_empty_input(self):
        """Should handle empty input gracefully"""
        result = function_to_test("")
        assert result == ""
    
    def test_raises_error_on_invalid_input(self):
        """Should raise ValueError for invalid input"""
        with pytest.raises(ValueError, match="Invalid input"):
            function_to_test(None)
    
    @pytest.mark.parametrize("input_val,expected", [
        ("test1", "output1"),
        ("test2", "output2"),
        ("test3", "output3"),
    ])
    def test_multiple_inputs(self, input_val, expected):
        """Should handle various inputs correctly"""
        assert function_to_test(input_val) == expected
```

## Common Testing Patterns

### 1. Basic Function Testing

```python
import pytest
from src.utils import calculate_total, format_price

class TestCalculateTotal:
    """Test suite for calculate_total function"""
    
    def test_calculates_total_correctly(self):
        """Should sum all item prices"""
        items = [10.0, 20.0, 30.0]
        assert calculate_total(items) == 60.0
    
    def test_handles_empty_list(self):
        """Should return 0 for empty list"""
        assert calculate_total([]) == 0.0
    
    def test_handles_single_item(self):
        """Should return item price for single item"""
        assert calculate_total([42.0]) == 42.0
    
    def test_handles_negative_prices(self):
        """Should handle negative prices (refunds)"""
        assert calculate_total([10.0, -5.0, 20.0]) == 25.0
    
    def test_raises_error_for_none(self):
        """Should raise TypeError for None input"""
        with pytest.raises(TypeError):
            calculate_total(None)
    
    @pytest.mark.parametrize("prices,expected", [
        ([1.0, 2.0, 3.0], 6.0),
        ([0.0], 0.0),
        ([100.0, 200.0], 300.0),
        ([], 0.0),
    ])
    def test_various_price_lists(self, prices, expected):
        """Should calculate total for various price lists"""
        assert calculate_total(prices) == expected


class TestFormatPrice:
    """Test suite for format_price function"""
    
    def test_formats_price_with_two_decimals(self):
        """Should format price with exactly 2 decimal places"""
        assert format_price(42) == "$42.00"
        assert format_price(42.5) == "$42.50"
        assert format_price(42.555) == "$42.56"  # rounds up
    
    def test_handles_zero(self):
        """Should format zero correctly"""
        assert format_price(0) == "$0.00"
    
    def test_handles_large_numbers(self):
        """Should format large numbers with commas"""
        assert format_price(1000) == "$1,000.00"
        assert format_price(1234567.89) == "$1,234,567.89"
    
    def test_raises_error_for_negative_price(self):
        """Should raise ValueError for negative prices"""
        with pytest.raises(ValueError, match="Price cannot be negative"):
            format_price(-10)
```

### 2. Testing Classes

```python
import pytest
from src.models import ShoppingCart, Product

class TestShoppingCart:
    """Test suite for ShoppingCart class"""
    
    @pytest.fixture
    def cart(self):
        """Fixture providing a fresh shopping cart for each test"""
        return ShoppingCart()
    
    @pytest.fixture
    def sample_product(self):
        """Fixture providing a sample product"""
        return Product(id=1, name="Widget", price=19.99)
    
    def test_cart_starts_empty(self, cart):
        """Should initialize with zero items"""
        assert len(cart) == 0
        assert cart.total == 0.0
    
    def test_add_item_increases_count(self, cart, sample_product):
        """Should increase item count when adding product"""
        cart.add_item(sample_product)
        assert len(cart) == 1
    
    def test_add_item_updates_total(self, cart, sample_product):
        """Should update total price when adding product"""
        cart.add_item(sample_product)
        assert cart.total == 19.99
    
    def test_add_multiple_same_items(self, cart, sample_product):
        """Should handle adding same item multiple times"""
        cart.add_item(sample_product, quantity=3)
        assert len(cart) == 3
        assert cart.total == pytest.approx(59.97)
    
    def test_remove_item_decreases_count(self, cart, sample_product):
        """Should decrease item count when removing product"""
        cart.add_item(sample_product)
        cart.remove_item(sample_product)
        assert len(cart) == 0
    
    def test_remove_nonexistent_item_raises_error(self, cart, sample_product):
        """Should raise ValueError when removing item not in cart"""
        with pytest.raises(ValueError, match="Item not in cart"):
            cart.remove_item(sample_product)
    
    def test_clear_empties_cart(self, cart, sample_product):
        """Should remove all items when clearing cart"""
        cart.add_item(sample_product, quantity=5)
        cart.clear()
        assert len(cart) == 0
        assert cart.total == 0.0
    
    def test_apply_discount_reduces_total(self, cart, sample_product):
        """Should apply percentage discount correctly"""
        cart.add_item(sample_product)
        cart.apply_discount(10)  # 10% discount
        assert cart.total == pytest.approx(17.99)
    
    def test_discount_cannot_exceed_100_percent(self, cart):
        """Should raise ValueError for discount over 100%"""
        with pytest.raises(ValueError, match="Discount cannot exceed 100%"):
            cart.apply_discount(150)
```

### 3. Mocking External Dependencies

```python
import pytest
from unittest.mock import Mock, patch, call
from src.api_client import WeatherClient
from src.email_service import EmailService

class TestWeatherClient:
    """Test suite for WeatherClient"""
    
    @patch('src.api_client.requests.get')
    def test_fetch_weather_success(self, mock_get):
        """Should return weather data for valid city"""
        # Arrange
        mock_response = Mock()
        mock_response.json.return_value = {
            'temperature': 72,
            'condition': 'sunny'
        }
        mock_response.status_code = 200
        mock_get.return_value = mock_response
        
        client = WeatherClient(api_key="test_key")
        
        # Act
        weather = client.fetch_weather("New York")
        
        # Assert
        assert weather['temperature'] == 72
        assert weather['condition'] == 'sunny'
        mock_get.assert_called_once_with(
            'https://api.weather.com/data',
            params={'city': 'New York', 'api_key': 'test_key'}
        )
    
    @patch('src.api_client.requests.get')
    def test_fetch_weather_api_error(self, mock_get):
        """Should raise exception when API returns error"""
        mock_response = Mock()
        mock_response.status_code = 500
        mock_get.return_value = mock_response
        
        client = WeatherClient(api_key="test_key")
        
        with pytest.raises(Exception, match="API Error"):
            client.fetch_weather("Invalid City")
    
    @patch('src.api_client.requests.get')
    def test_fetch_weather_network_error(self, mock_get):
        """Should handle network errors gracefully"""
        mock_get.side_effect = ConnectionError("Network unreachable")
        
        client = WeatherClient(api_key="test_key")
        
        with pytest.raises(ConnectionError):
            client.fetch_weather("New York")


class TestEmailService:
    """Test suite for EmailService"""
    
    @patch('src.email_service.smtplib.SMTP')
    def test_send_email_success(self, mock_smtp):
        """Should send email successfully"""
        # Arrange
        mock_server = Mock()
        mock_smtp.return_value.__enter__.return_value = mock_server
        
        service = EmailService(host="smtp.example.com", port=587)
        
        # Act
        service.send_email(
            to="user@example.com",
            subject="Test",
            body="Test message"
        )
        
        # Assert
        mock_server.send_message.assert_called_once()
        call_args = mock_server.send_message.call_args[0][0]
        assert call_args['To'] == "user@example.com"
        assert call_args['Subject'] == "Test"
    
    @patch('src.email_service.smtplib.SMTP')
    def test_send_email_authentication_failure(self, mock_smtp):
        """Should raise error on authentication failure"""
        mock_server = Mock()
        mock_server.login.side_effect = Exception("Authentication failed")
        mock_smtp.return_value.__enter__.return_value = mock_server
        
        service = EmailService(host="smtp.example.com", port=587)
        
        with pytest.raises(Exception, match="Authentication failed"):
            service.send_email(
                to="user@example.com",
                subject="Test",
                body="Test"
            )
```

### 4. Testing Async Functions

```python
import pytest
import asyncio
from src.async_service import fetch_user, process_batch

class TestAsyncFunctions:
    """Test suite for async functions"""
    
    @pytest.mark.asyncio
    async def test_fetch_user_returns_user_data(self):
        """Should return user data for valid user ID"""
        user = await fetch_user(1)
        
        assert user['id'] == 1
        assert 'name' in user
        assert 'email' in user
    
    @pytest.mark.asyncio
    async def test_fetch_user_handles_not_found(self):
        """Should raise exception for non-existent user"""
        with pytest.raises(ValueError, match="User not found"):
            await fetch_user(99999)
    
    @pytest.mark.asyncio
    async def test_process_batch_handles_multiple_items(self):
        """Should process all items in batch"""
        items = [1, 2, 3, 4, 5]
        results = await process_batch(items)
        
        assert len(results) == 5
        assert all(result is not None for result in results)
    
    @pytest.mark.asyncio
    async def test_process_batch_handles_empty_list(self):
        """Should return empty list for empty input"""
        results = await process_batch([])
        assert results == []
    
    @pytest.mark.asyncio
    async def test_concurrent_requests(self):
        """Should handle concurrent async requests"""
        tasks = [fetch_user(i) for i in range(1, 4)]
        results = await asyncio.gather(*tasks)
        
        assert len(results) == 3
        assert results[0]['id'] == 1
        assert results[1]['id'] == 2
        assert results[2]['id'] == 3


# Testing async context managers
class TestAsyncContextManager:
    """Test suite for async context managers"""
    
    @pytest.mark.asyncio
    async def test_async_database_connection(self):
        """Should properly handle async database connection"""
        from src.database import AsyncDatabase
        
        async with AsyncDatabase() as db:
            result = await db.query("SELECT 1")
            assert result is not None
    
    @pytest.mark.asyncio
    async def test_connection_cleanup_on_error(self):
        """Should cleanup connection even on error"""
        from src.database import AsyncDatabase
        
        with pytest.raises(ValueError):
            async with AsyncDatabase() as db:
                raise ValueError("Test error")
        
        # Verify connection was closed (implementation-specific)
```

### 5. Testing Django Models

```python
import pytest
from django.core.exceptions import ValidationError
from myapp.models import Article, User

@pytest.mark.django_db
class TestArticleModel:
    """Test suite for Article model"""
    
    @pytest.fixture
    def user(self):
        """Fixture providing a test user"""
        return User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    @pytest.fixture
    def article(self, user):
        """Fixture providing a test article"""
        return Article.objects.create(
            title='Test Article',
            content='Test content',
            author=user,
            status='draft'
        )
    
    def test_create_article(self, user):
        """Should create article with valid data"""
        article = Article.objects.create(
            title='New Article',
            content='Article content',
            author=user
        )
        
        assert article.id is not None
        assert article.title == 'New Article'
        assert article.author == user
        assert article.status == 'draft'  # default status
    
    def test_article_str_representation(self, article):
        """Should return title as string representation"""
        assert str(article) == 'Test Article'
    
    def test_article_requires_title(self, user):
        """Should raise error when title is missing"""
        with pytest.raises(ValidationError):
            article = Article(content='Content', author=user)
            article.full_clean()
    
    def test_article_requires_author(self):
        """Should raise error when author is missing"""
        with pytest.raises(ValidationError):
            article = Article(title='Title', content='Content')
            article.full_clean()
    
    def test_publish_article_updates_status(self, article):
        """Should update status to published"""
        article.publish()
        
        assert article.status == 'published'
        assert article.published_at is not None
    
    def test_cannot_publish_empty_article(self, user):
        """Should not allow publishing article without content"""
        article = Article.objects.create(
            title='Empty',
            content='',
            author=user
        )
        
        with pytest.raises(ValidationError, match="Cannot publish empty article"):
            article.publish()
    
    def test_get_published_articles_queryset(self, user):
        """Should return only published articles"""
        # Create mix of published and draft articles
        Article.objects.create(title='Draft 1', content='Content', author=user, status='draft')
        published = Article.objects.create(title='Published 1', content='Content', author=user)
        published.status = 'published'
        published.save()
        
        published_articles = Article.objects.published()
        
        assert published_articles.count() == 1
        assert published_articles.first().title == 'Published 1'
    
    def test_article_word_count(self, article):
        """Should calculate word count correctly"""
        article.content = "This is a test content with ten words here"
        assert article.word_count() == 10
    
    def test_article_word_count_empty(self, user):
        """Should return 0 for empty content"""
        article = Article.objects.create(title='Empty', content='', author=user)
        assert article.word_count() == 0
```

### 6. Testing Django Views

```python
import pytest
from django.urls import reverse
from django.test import Client
from myapp.models import Article, User

@pytest.mark.django_db
class TestArticleListView:
    """Test suite for article list view"""
    
    @pytest.fixture
    def client(self):
        """Fixture providing Django test client"""
        return Client()
    
    @pytest.fixture
    def user(self):
        """Fixture providing test user"""
        return User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    @pytest.fixture
    def articles(self, user):
        """Fixture providing multiple test articles"""
        return [
            Article.objects.create(
                title=f'Article {i}',
                content=f'Content {i}',
                author=user,
                status='published'
            )
            for i in range(5)
        ]
    
    def test_article_list_view_status_code(self, client):
        """Should return 200 status code"""
        url = reverse('article-list')
        response = client.get(url)
        
        assert response.status_code == 200
    
    def test_article_list_view_uses_correct_template(self, client):
        """Should use article_list.html template"""
        url = reverse('article-list')
        response = client.get(url)
        
        assert 'article_list.html' in [t.name for t in response.templates]
    
    def test_article_list_displays_articles(self, client, articles):
        """Should display all published articles"""
        url = reverse('article-list')
        response = client.get(url)
        
        assert response.status_code == 200
        assert len(response.context['articles']) == 5
        for article in articles:
            assert article.title.encode() in response.content
    
    def test_article_list_pagination(self, client, user):
        """Should paginate articles correctly"""
        # Create 25 articles
        for i in range(25):
            Article.objects.create(
                title=f'Article {i}',
                content='Content',
                author=user,
                status='published'
            )
        
        url = reverse('article-list')
        response = client.get(url)
        
        assert len(response.context['articles']) == 10  # page size
        assert response.context['is_paginated'] is True
    
    def test_article_list_filters_by_status(self, client, user):
        """Should only show published articles"""
        Article.objects.create(title='Published', content='Content', author=user, status='published')
        Article.objects.create(title='Draft', content='Content', author=user, status='draft')
        
        url = reverse('article-list')
        response = client.get(url)
        
        assert len(response.context['articles']) == 1
        assert response.context['articles'][0].title == 'Published'


@pytest.mark.django_db
class TestArticleCreateView:
    """Test suite for article create view"""
    
    @pytest.fixture
    def client(self):
        return Client()
    
    @pytest.fixture
    def user(self):
        return User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    def test_create_view_requires_authentication(self, client):
        """Should redirect to login for unauthenticated users"""
        url = reverse('article-create')
        response = client.get(url)
        
        assert response.status_code == 302
        assert '/login/' in response.url
    
    def test_create_view_displays_form(self, client, user):
        """Should display article creation form for authenticated users"""
        client.force_login(user)
        url = reverse('article-create')
        response = client.get(url)
        
        assert response.status_code == 200
        assert 'form' in response.context
    
    def test_create_article_with_valid_data(self, client, user):
        """Should create article with valid form data"""
        client.force_login(user)
        url = reverse('article-create')
        data = {
            'title': 'New Article',
            'content': 'Article content goes here'
        }
        
        response = client.post(url, data)
        
        assert response.status_code == 302  # redirect after success
        assert Article.objects.filter(title='New Article').exists()
        article = Article.objects.get(title='New Article')
        assert article.author == user
    
    def test_create_article_with_invalid_data(self, client, user):
        """Should show errors for invalid form data"""
        client.force_login(user)
        url = reverse('article-create')
        data = {'title': '', 'content': ''}  # empty required fields
        
        response = client.post(url, data)
        
        assert response.status_code == 200  # stays on form
        assert 'form' in response.context
        assert response.context['form'].errors
        assert not Article.objects.filter(title='').exists()
```

### 7. Testing with Pandas

```python
import pytest
import pandas as pd
import numpy as np
from src.data_processor import clean_data, aggregate_sales, detect_outliers

class TestCleanData:
    """Test suite for data cleaning functions"""
    
    @pytest.fixture
    def sample_dataframe(self):
        """Fixture providing sample DataFrame"""
        return pd.DataFrame({
            'id': [1, 2, 3, 4, 5],
            'name': ['Alice', 'Bob', None, 'David', 'Eve'],
            'age': [25, 30, np.nan, 35, 28],
            'salary': [50000, 60000, 55000, None, 52000]
        })
    
    def test_clean_data_removes_null_rows(self, sample_dataframe):
        """Should remove rows with any null values"""
        result = clean_data(sample_dataframe, strategy='drop')
        
        assert len(result) == 2  # Only rows without nulls
        assert result['name'].isna().sum() == 0
        assert result['age'].isna().sum() == 0
    
    def test_clean_data_fills_null_values(self, sample_dataframe):
        """Should fill null values with specified value"""
        result = clean_data(sample_dataframe, strategy='fill', fill_value=0)
        
        assert len(result) == 5  # All rows preserved
        assert result['age'].isna().sum() == 0
        assert result.loc[2, 'age'] == 0
    
    def test_clean_data_preserves_datatypes(self, sample_dataframe):
        """Should maintain column datatypes after cleaning"""
        result = clean_data(sample_dataframe, strategy='drop')
        
        assert result['id'].dtype == sample_dataframe['id'].dtype
        assert result['age'].dtype == sample_dataframe['age'].dtype
    
    def test_clean_data_handles_empty_dataframe(self):
        """Should handle empty DataFrame gracefully"""
        empty_df = pd.DataFrame()
        result = clean_data(empty_df)
        
        assert len(result) == 0
        assert isinstance(result, pd.DataFrame)


class TestAggregateSales:
    """Test suite for sales aggregation"""
    
    @pytest.fixture
    def sales_data(self):
        """Fixture providing sales DataFrame"""
        return pd.DataFrame({
            'date': pd.date_range('2024-01-01', periods=10, freq='D'),
            'product': ['A', 'B', 'A', 'B', 'A'] * 2,
            'quantity': [10, 20, 15, 25, 12, 18, 22, 14, 28, 16],
            'price': [100, 150, 100, 150, 100, 100, 150, 100, 150, 100]
        })
    
    def test_aggregate_by_product(self, sales_data):
        """Should aggregate sales by product correctly"""
        result = aggregate_sales(sales_data, group_by='product')
        
        assert 'A' in result.index
        assert 'B' in result.index
        assert result.loc['A', 'total_quantity'] == 10 + 15 + 12 + 14 + 16
        assert result.loc['B', 'total_quantity'] == 20 + 25 + 18 + 22 + 28
    
    def test_aggregate_calculates_revenue(self, sales_data):
        """Should calculate total revenue correctly"""
        result = aggregate_sales(sales_data, group_by='product')
        
        expected_revenue_a = (10 + 15 + 12 + 14 + 16) * 100
        expected_revenue_b = (20 + 25 + 18 + 22 + 28) * 150
        
        assert result.loc['A', 'total_revenue'] == expected_revenue_a
        assert result.loc['B', 'total_revenue'] == expected_revenue_b
    
    def test_aggregate_handles_empty_dataframe(self):
        """Should handle empty DataFrame without errors"""
        empty_df = pd.DataFrame(columns=['product', 'quantity', 'price'])
        result = aggregate_sales(empty_df, group_by='product')
        
        assert len(result) == 0


class TestDetectOutliers:
    """Test suite for outlier detection"""
    
    def test_detect_outliers_zscore_method(self):
        """Should detect outliers using z-score method"""
        data = pd.Series([1, 2, 3, 4, 5, 100])  # 100 is clear outlier
        outliers = detect_outliers(data, method='zscore', threshold=2)
        
        assert len(outliers) == 1
        assert 100 in outliers.values
    
    def test_detect_outliers_iqr_method(self):
        """Should detect outliers using IQR method"""
        data = pd.Series([1, 2, 3, 4, 5, 100])
        outliers = detect_outliers(data, method='iqr')
        
        assert len(outliers) >= 1
        assert 100 in outliers.values
    
    def test_detect_outliers_no_outliers(self):
        """Should return empty series when no outliers exist"""
        data = pd.Series([1, 2, 3, 4, 5])  # Normal distribution
        outliers = detect_outliers(data, method='zscore', threshold=3)
        
        assert len(outliers) == 0
    
    @pytest.mark.parametrize("method,threshold", [
        ('zscore', 2),
        ('zscore', 3),
        ('iqr', None),
    ])
    def test_detect_outliers_various_parameters(self, method, threshold):
        """Should work with different methods and thresholds"""
        data = pd.Series(list(range(1, 100)) + [1000])
        outliers = detect_outliers(data, method=method, threshold=threshold)
        
        assert isinstance(outliers, pd.Series)
```

## Fixtures Best Practices

### Fixture Scopes

```python
# Function scope (default) - runs before each test
@pytest.fixture
def temp_data():
    return {'key': 'value'}

# Class scope - runs once per test class
@pytest.fixture(scope='class')
def database_connection():
    db = Database()
    yield db
    db.close()

# Module scope - runs once per module
@pytest.fixture(scope='module')
def app_config():
    return load_config()

# Session scope - runs once per test session
@pytest.fixture(scope='session')
def test_database():
    db = create_test_db()
    yield db
    drop_test_db(db)
```

### Fixture Cleanup with yield

```python
@pytest.fixture
def temp_file():
    """Create temp file and clean up after test"""
    file_path = '/tmp/test_file.txt'
    with open(file_path, 'w') as f:
        f.write('test data')
    
    yield file_path  # Test runs here
    
    # Cleanup
    import os
    if os.path.exists(file_path):
        os.remove(file_path)
```

## Parametrized Tests

```python
@pytest.mark.parametrize("input_value,expected", [
    (0, 0),
    (1, 1),
    (2, 4),
    (3, 9),
    (-2, 4),
])
def test_square_function(input_value, expected):
    """Should calculate square correctly"""
    assert square(input_value) == expected


# Multiple parameters
@pytest.mark.parametrize("width,height,expected_area", [
    (3, 4, 12),
    (5, 5, 25),
    (0, 10, 0),
])
def test_rectangle_area(width, height, expected_area):
    """Should calculate rectangle area correctly"""
    assert calculate_area(width, height) == expected_area


# Parametrize with IDs for better test names
@pytest.mark.parametrize("input,expected", [
    pytest.param("hello", "HELLO", id="lowercase"),
    pytest.param("HELLO", "HELLO", id="uppercase"),
    pytest.param("Hello", "HELLO", id="mixed_case"),
])
def test_to_uppercase(input, expected):
    """Should convert string to uppercase"""
    assert to_uppercase(input) == expected
```

## Edge Cases to Always Test

1. **Empty/None Inputs**
   - Empty lists, dicts, strings
   - None values
   - Zero values

2. **Boundary Conditions**
   - Minimum/maximum values
   - Zero/negative numbers
   - Very large numbers

3. **Error Conditions**
   - Invalid input types
   - Missing required parameters
   - Network/database failures

4. **Data Type Handling**
   - Different numeric types
   - Unicode strings
   - Date/time edge cases

## Assertions Best Practices

```python
# Use specific assertions
assert value == expected  # Basic equality
assert value is None  # Identity check
assert value in collection  # Membership
assert len(collection) == 5  # Length

# Use pytest.approx for floats
assert result == pytest.approx(3.14159, rel=1e-5)

# Use context managers for exceptions
with pytest.raises(ValueError):
    function_that_raises()

with pytest.raises(ValueError, match="specific error message"):
    function_that_raises()

# Multiple assertions (use when logical)
def test_user_creation():
    """Should create user with all attributes"""
    user = create_user("John", "john@example.com")
    
    assert user.name == "John"
    assert user.email == "john@example.com"
    assert user.id is not None
    assert user.created_at is not None
```

## Test Naming Convention

```python
# ✅ GOOD - Descriptive and specific
def test_calculate_total_with_valid_prices_returns_sum():
def test_create_user_with_duplicate_email_raises_error():
def test_empty_shopping_cart_has_zero_total():

# ❌ BAD - Vague or unclear
def test_function():
def test_case_1():
def test_it_works():
```

## TDD Workflow

1. **Write failing test** that describes desired behavior
2. **Run test** to confirm it fails
3. **Write minimum code** to make test pass
4. **Run test** to confirm it passes
5. **Refactor** code while keeping test green
6. **Repeat** for next feature

## Common Pitfalls to Avoid

1. **Don't test implementation details**
   ```python
   # ❌ BAD
   assert obj._internal_variable == 5
   
   # ✅ GOOD
   assert obj.get_value() == 5
   ```

2. **Don't have test dependencies**
   ```python
   # ❌ BAD - tests depend on order
   def test_1_create_user():
       global user
       user = create_user()
   
   def test_2_update_user():
       user.update()  # Depends on test_1
   
   # ✅ GOOD - independent tests
   @pytest.fixture
   def user():
       return create_user()
   
   def test_create_user(user):
       assert user is not None
   
   def test_update_user(user):
       user.update()
       assert user.updated
   ```

3. **Clean up mocks between tests**
   ```python
   @pytest.fixture(autouse=True)
   def reset_mocks():
       yield
       # Reset all mocks after each test
   ```

## Output Guidelines

When writing tests:

1. **Understand the code** - Ask about:
   - Function/class purpose
   - Input/output types
   - External dependencies
   - Edge cases and error conditions

2. **Generate comprehensive test file** with:
   - Clear test class organization
   - Fixtures for common setup
   - All edge cases covered
   - Descriptive test names

3. **Include setup** if needed:
   - pytest.ini or pyproject.toml config
   - conftest.py for shared fixtures
   - Mock setup for external dependencies

4. **Explain briefly** what's being tested and why

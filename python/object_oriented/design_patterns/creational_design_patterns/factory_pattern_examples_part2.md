# Factory Method Pattern - Part 2: Real-World Examples with Actual Value

These examples demonstrate Factory Method when it truly encapsulates complexity and adds real value—not just string-to-class mapping.

---

## Example 1: Configuration-Based API Client Factory

The factory handles complex initialization based on environment/config, including authentication, retry policies, and default headers.

```python
from abc import ABC, abstractmethod
import os
from typing import Dict, Any

# Product Interface
class APIClient(ABC):
    @abstractmethod
    def get(self, endpoint):
        pass
    
    @abstractmethod
    def post(self, endpoint, data):
        pass

# Concrete Products
class ProductionAPIClient(APIClient):
    def __init__(self, api_key, timeout, retry_count, base_url):
        self.api_key = api_key
        self.timeout = timeout
        self.retry_count = retry_count
        self.base_url = base_url
    
    def get(self, endpoint):
        return f"GET {self.base_url}{endpoint} with API key (timeout: {self.timeout}s, retries: {self.retry_count})"
    
    def post(self, endpoint, data):
        return f"POST {self.base_url}{endpoint} with {data} (authenticated)"

class DevelopmentAPIClient(APIClient):
    def __init__(self, mock_data=None, timeout=5, base_url="http://localhost"):
        self.mock_data = mock_data or {}
        self.timeout = timeout
        self.base_url = base_url
    
    def get(self, endpoint):
        if endpoint in self.mock_data:
            return f"[MOCK] GET {endpoint}: {self.mock_data[endpoint]}"
        return f"[DEV] GET {self.base_url}{endpoint}"
    
    def post(self, endpoint, data):
        return f"[DEV] POST {endpoint} with {data} (no authentication needed)"

class MockAPIClient(APIClient):
    def __init__(self, mock_responses):
        self.mock_responses = mock_responses
    
    def get(self, endpoint):
        return self.mock_responses.get(endpoint, "No mock data for endpoint")
    
    def post(self, endpoint, data):
        return f"[MOCK] POST successful: {data}"

# Creator - Where the real logic happens
class APIClientFactory:
    @staticmethod
    def create_client(environment=None) -> APIClient:
        """
        Factory method with actual business logic:
        - Reads from environment/config
        - Handles authentication setup
        - Configures retry policies and timeouts
        - Validates required credentials
        """
        environment = environment or os.getenv("APP_ENV", "development")
        
        if environment == "production":
            api_key = os.getenv("API_KEY")
            if not api_key:
                raise ValueError("API_KEY environment variable required for production")
            
            return ProductionAPIClient(
                api_key=api_key,
                timeout=int(os.getenv("API_TIMEOUT", 30)),
                retry_count=int(os.getenv("API_RETRIES", 3)),
                base_url=os.getenv("API_BASE_URL", "https://api.example.com")
            )
        
        elif environment == "testing":
            # For testing, return mock client with predefined responses
            mock_data = {
                "/users": [{"id": 1, "name": "John"}],
                "/products": [{"id": 1, "price": 99.99}]
            }
            return MockAPIClient(mock_data)
        
        else:  # development
            return DevelopmentAPIClient(
                timeout=int(os.getenv("DEV_TIMEOUT", 5)),
                base_url=os.getenv("DEV_BASE_URL", "http://localhost:8000")
            )

# Usage
# In production
os.environ["APP_ENV"] = "production"
os.environ["API_KEY"] = "secret-key-12345"
prod_client = APIClientFactory.create_client()
print(prod_client.get("/users"))  # Uses real API with auth

# In testing
test_client = APIClientFactory.create_client("testing")
print(test_client.get("/users"))  # Returns mock data

# In development
dev_client = APIClientFactory.create_client("development")
print(dev_client.get("/users"))  # Calls local server
```

---

## Example 2: Database Connection Pool Factory

The factory manages connection pooling, connection validation, and failover logic—not just creating database objects.

```python
from abc import ABC, abstractmethod
from typing import List
import time

# Product Interface
class DatabaseConnection(ABC):
    @abstractmethod
    def execute(self, query):
        pass
    
    @abstractmethod
    def close(self):
        pass

# Concrete Products
class PostgreSQLConnection(DatabaseConnection):
    def __init__(self, host, port, username, password):
        self.host = host
        self.port = port
        self.username = username
        self.connection_time = time.time()
    
    def execute(self, query):
        return f"PostgreSQL: Executing on {self.host}:{self.port} - {query}"
    
    def close(self):
        return "PostgreSQL connection closed"

class MySQLConnection(DatabaseConnection):
    def __init__(self, host, port, username, password):
        self.host = host
        self.port = port
        self.username = username
        self.connection_time = time.time()
    
    def execute(self, query):
        return f"MySQL: Executing on {self.host}:{self.port} - {query}"
    
    def close(self):
        return "MySQL connection closed"

# Creator - Manages connection pooling and failover
class DatabaseConnectionFactory:
    _pools = {}
    
    @staticmethod
    def create_connection_pool(db_type, primary_config, replica_configs=None, pool_size=5):
        """
        Factory with real complexity:
        - Creates connection pools, not just single connections
        - Implements failover logic
        - Manages connection health checks
        - Returns connections with automatic retry
        """
        pool_key = f"{db_type}_{primary_config['host']}"
        
        if pool_key in DatabaseConnectionFactory._pools:
            return DatabaseConnectionFactory._pools[pool_key]
        
        # Determine which connection class to use
        if db_type == "postgresql":
            ConnectionClass = PostgreSQLConnection
        elif db_type == "mysql":
            ConnectionClass = MySQLConnection
        else:
            raise ValueError(f"Unknown database type: {db_type}")
        
        # Create pool with primary and replicas
        pool = ConnectionPool(
            ConnectionClass=ConnectionClass,
            primary_config=primary_config,
            replica_configs=replica_configs or [],
            pool_size=pool_size
        )
        
        DatabaseConnectionFactory._pools[pool_key] = pool
        return pool

class ConnectionPool:
    def __init__(self, ConnectionClass, primary_config, replica_configs, pool_size):
        self.ConnectionClass = ConnectionClass
        self.primary_config = primary_config
        self.replica_configs = replica_configs
        self.pool_size = pool_size
        self.connections = []
        self.available_connections = []
        
        # Initialize pool
        for i in range(pool_size):
            try:
                # Try primary first
                conn = self._create_connection(primary_config)
                self.connections.append(conn)
                self.available_connections.append(conn)
            except Exception as e:
                print(f"Failed to create connection: {e}")
    
    def _create_connection(self, config):
        """Encapsulated connection creation with validation"""
        conn = self.ConnectionClass(**config)
        # Validate connection
        conn.execute("SELECT 1")
        return conn
    
    def get_connection(self):
        """Get a connection from pool with failover"""
        if not self.available_connections:
            raise RuntimeError("No connections available in pool")
        
        conn = self.available_connections.pop(0)
        return conn
    
    def return_connection(self, conn):
        """Return connection to pool"""
        self.available_connections.append(conn)

# Usage
pool = DatabaseConnectionFactory.create_connection_pool(
    db_type="postgresql",
    primary_config={"host": "prod-db.example.com", "port": 5432, "username": "app", "password": "secret"},
    replica_configs=[
        {"host": "replica1.example.com", "port": 5432, "username": "app", "password": "secret"}
    ],
    pool_size=5
)

conn = pool.get_connection()
print(conn.execute("SELECT * FROM users"))
pool.return_connection(conn)
```

---

## Example 3: Plugin System Factory

The factory loads plugins dynamically, manages dependencies, and validates plugin interface contracts.

```python
from abc import ABC, abstractmethod
import importlib
import sys
from pathlib import Path

# Product Interface
class Plugin(ABC):
    def __init__(self):
        self.name = self.__class__.__name__
        self.version = "1.0.0"
        self.dependencies = []
    
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def validate(self):
        """Validates plugin can run in current environment"""
        pass

# Concrete Products
class DataProcessingPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.dependencies = ["numpy", "pandas"]
    
    def execute(self):
        return "Data processing plugin executed"
    
    def validate(self):
        return all(self._check_dependency(dep) for dep in self.dependencies)
    
    def _check_dependency(self, dep):
        try:
            importlib.import_module(dep)
            return True
        except ImportError:
            return False

class ReportingPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.dependencies = ["jinja2"]
    
    def execute(self):
        return "Report generation plugin executed"
    
    def validate(self):
        return all(self._check_dependency(dep) for dep in self.dependencies)
    
    def _check_dependency(self, dep):
        try:
            importlib.import_module(dep)
            return True
        except ImportError:
            return False

# Creator - Complex plugin loading logic
class PluginFactory:
    _loaded_plugins = {}
    
    @staticmethod
    def load_plugin(plugin_name, plugin_path="./plugins"):
        """
        Factory with real complexity:
        - Dynamically imports modules
        - Validates plugin interface
        - Checks dependencies
        - Caches loaded plugins
        - Handles errors gracefully
        """
        if plugin_name in PluginFactory._loaded_plugins:
            return PluginFactory._loaded_plugins[plugin_name]
        
        try:
            # Dynamically load module
            module_path = Path(plugin_path) / f"{plugin_name}.py"
            
            if not module_path.exists():
                raise FileNotFoundError(f"Plugin not found: {module_path}")
            
            # Import module dynamically
            spec = importlib.util.spec_from_file_location(plugin_name, module_path)
            module = importlib.util.module_from_spec(spec)
            sys.modules[plugin_name] = module
            spec.loader.exec_module(module)
            
            # Find Plugin subclass in module
            plugin_class = None
            for item_name in dir(module):
                item = getattr(module, item_name)
                if isinstance(item, type) and issubclass(item, Plugin) and item != Plugin:
                    plugin_class = item
                    break
            
            if not plugin_class:
                raise ValueError(f"No Plugin subclass found in {plugin_name}")
            
            # Instantiate and validate
            plugin = plugin_class()
            
            if not plugin.validate():
                raise RuntimeError(f"Plugin {plugin_name} validation failed - missing dependencies")
            
            PluginFactory._loaded_plugins[plugin_name] = plugin
            return plugin
        
        except Exception as e:
            print(f"Error loading plugin {plugin_name}: {e}")
            raise

# Usage (simulated)
# In real code, plugins would be in separate files
# plugins = PluginFactory.load_plugin("data_processing")
# plugins = PluginFactory.load_plugin("reporting")

# For demo, create directly
plugin = DataProcessingPlugin()
if plugin.validate():
    print(plugin.execute())
```

---

## Example 4: Caching Strategy Factory

The factory creates appropriate cache implementation based on data size, access patterns, and resource constraints.

```python
from abc import ABC, abstractmethod
from typing import Any
import time

# Product Interface
class CacheStrategy(ABC):
    @abstractmethod
    def get(self, key):
        pass
    
    @abstractmethod
    def set(self, key, value, ttl=None):
        pass
    
    @abstractmethod
    def clear(self):
        pass

# Concrete Products
class InMemoryCache(CacheStrategy):
    def __init__(self, max_size=1000):
        self.cache = {}
        self.max_size = max_size
        self.access_count = {}
    
    def get(self, key):
        self.access_count[key] = self.access_count.get(key, 0) + 1
        return self.cache.get(key)
    
    def set(self, key, value, ttl=None):
        if len(self.cache) >= self.max_size:
            # LRU eviction
            least_used = min(self.access_count, key=self.access_count.get)
            del self.cache[least_used]
            del self.access_count[least_used]
        self.cache[key] = value
    
    def clear(self):
        self.cache.clear()
        self.access_count.clear()

class RedisLikeCache(CacheStrategy):
    def __init__(self, ttl_default=3600):
        self.cache = {}
        self.ttl_default = ttl_default
        self.expiry = {}
    
    def get(self, key):
        if key in self.expiry and time.time() > self.expiry[key]:
            del self.cache[key]
            del self.expiry[key]
            return None
        return self.cache.get(key)
    
    def set(self, key, value, ttl=None):
        self.cache[key] = value
        ttl = ttl or self.ttl_default
        self.expiry[key] = time.time() + ttl
    
    def clear(self):
        self.cache.clear()
        self.expiry.clear()

class NoCache(CacheStrategy):
    def get(self, key):
        return None
    
    def set(self, key, value, ttl=None):
        pass
    
    def clear(self):
        pass

# Creator - Intelligent selection based on requirements
class CacheFactory:
    @staticmethod
    def create_cache(
        data_size_estimate: int,
        access_frequency: str,
        memory_available: int,
        persistence_needed: bool = False
    ) -> CacheStrategy:
        """
        Factory with decision logic:
        - Analyzes data characteristics
        - Considers system resources
        - Selects optimal strategy
        - Falls back if insufficient resources
        """
        
        # If no memory available, don't cache
        if memory_available < 10:  # MB
            return NoCache()
        
        # For very large datasets with low access frequency, don't cache
        if data_size_estimate > memory_available * 10:
            if not persistence_needed:
                return NoCache()
            # Could return disk cache here
        
        # High access frequency = in-memory cache
        if access_frequency == "high":
            return InMemoryCache(max_size=data_size_estimate)
        
        # Medium/low frequency with TTL = Redis-like
        if access_frequency in ["medium", "low"]:
            return RedisLikeCache(ttl_default=3600)
        
        return NoCache()

# Usage
# Small, frequently accessed data
cache = CacheFactory.create_cache(
    data_size_estimate=1000,
    access_frequency="high",
    memory_available=512
)
print(type(cache).__name__)  # InMemoryCache

# Large data, infrequent access, limited memory
cache = CacheFactory.create_cache(
    data_size_estimate=100000,
    access_frequency="low",
    memory_available=50
)
print(type(cache).__name__)  # NoCache or RedisLikeCache
```

---

## Example 5: Report Generator Factory

The factory orchestrates complex report generation with format selection, styling, and export options.

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Any

# Product Interface
class ReportGenerator(ABC):
    @abstractmethod
    def add_data(self, data):
        pass
    
    @abstractmethod
    def generate(self):
        pass
    
    @abstractmethod
    def export(self, filename):
        pass

# Concrete Products
class PDFReportGenerator(ReportGenerator):
    def __init__(self, title, style="default"):
        self.title = title
        self.style = style
        self.data = []
    
    def add_data(self, data):
        self.data.append(data)
    
    def generate(self):
        return f"PDF Report: {self.title} with {len(self.data)} sections (style: {self.style})"
    
    def export(self, filename):
        return f"PDF exported to {filename}.pdf with embedded fonts"

class ExcelReportGenerator(ReportGenerator):
    def __init__(self, title, include_formulas=True):
        self.title = title
        self.include_formulas = include_formulas
        self.data = []
    
    def add_data(self, data):
        self.data.append(data)
    
    def generate(self):
        formulas = "with formulas" if self.include_formulas else "data only"
        return f"Excel Report: {self.title} with {len(self.data)} sheets ({formulas})"
    
    def export(self, filename):
        return f"Excel exported to {filename}.xlsx with auto-formatting"

class HTMLReportGenerator(ReportGenerator):
    def __init__(self, title, responsive=True):
        self.title = title
        self.responsive = responsive
        self.data = []
    
    def add_data(self, data):
        self.data.append(data)
    
    def generate(self):
        responsive_info = "responsive" if self.responsive else "fixed-width"
        return f"HTML Report: {self.title} ({responsive_info}) with {len(self.data)} sections"
    
    def export(self, filename):
        return f"HTML exported to {filename}.html with embedded CSS"

# Creator - Manages report configuration and optimization
class ReportFactory:
    @staticmethod
    def create_report(
        report_type: str,
        title: str,
        data_size: int = None,
        distribution_method: str = None,
        **options
    ) -> ReportGenerator:
        """
        Factory with business logic:
        - Validates report feasibility
        - Selects format based on use case
        - Configures for distribution method
        - Optimizes for data size
        """
        
        # Validate inputs
        if not title:
            raise ValueError("Report title is required")
        
        # Select format based on use case and constraints
        if report_type == "financial":
            # Financial reports need calculations = Excel
            return ExcelReportGenerator(
                title=title,
                include_formulas=True
            )
        
        elif report_type == "presentation":
            # Presentations need styling = PDF
            return PDFReportGenerator(
                title=title,
                style=options.get("style", "professional")
            )
        
        elif report_type == "interactive":
            # Interactive reports = HTML
            return HTMLReportGenerator(
                title=title,
                responsive=True
            )
        
        elif report_type == "data_export":
            # Exporting large datasets = Excel
            if data_size and data_size > 100000:
                # For very large datasets, optimize
                return ExcelReportGenerator(
                    title=title,
                    include_formulas=False  # Disable formulas for performance
                )
            return ExcelReportGenerator(title=title)
        
        else:
            # Default = HTML for web distribution
            if distribution_method == "email":
                return PDFReportGenerator(title=title, style="email-friendly")
            return HTMLReportGenerator(title=title)

# Usage
financial_report = ReportFactory.create_report("financial", "Q3 Financial Report")
print(financial_report.generate())

presentation = ReportFactory.create_report("presentation", "Sales Overview", style="corporate")
print(presentation.generate())

data_export = ReportFactory.create_report("data_export", "Customer Export", data_size=500000)
print(data_export.generate())
```

---

## Key Differences from Part 1

| Aspect | Part 1 (Weak) | Part 2 (Strong) |
|--------|---------------|-----------------|
| **Factory Logic** | Simple string-to-class mapping | Complex decision-making and configuration |
| **Encapsulation** | None - just instantiation | Significant business logic hidden from client |
| **Resource Management** | Creates single objects | Manages pools, caches, lifecycles |
| **Validation** | None | Comprehensive validation and error handling |
| **Configuration** | Passed directly to constructors | Resolved from environment/context/parameters |
| **Value Add** | Minimal - adds indirection | High - hides complexity, manages resources |
| **Real-World Match** | Oversimplified | Matches actual application needs |

---

## When Factory Method Adds Real Value

✅ **Use it when:**
- Creation involves reading config/environment variables
- You need to validate prerequisites or dependencies
- Resource management is needed (pooling, caching, lifecycle)
- Multiple related products need coordination
- Different implementations have different setup requirements
- Business logic determines which implementation to use

❌ **Skip it when:**
- Creation is just `ClassName(**kwargs)`
- No configuration or validation needed
- No shared state or resource management
- You're adding it "just in case"

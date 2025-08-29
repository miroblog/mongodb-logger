# MongoDB Logging Installation Guide

Simple setup guide to add intelligent MongoDB logging to any Python project.

## Prerequisites

- Python 3.7+
- MongoDB database (local or hosted)
- Package manager (pip, uv, poetry, etc.)

## Installation Steps

### 1. Copy Files

Copy these files to your project:
```
mongo_logger.py    # MongoDB logging handler
```

### 2. Install Dependencies

Add to your project dependencies:

**Using pip:**
```bash
pip install pymongo python-dotenv
```

**Using uv (pyproject.toml):**
```toml
dependencies = [
    "pymongo>=4.0.0",
    "python-dotenv>=1.0.0",
]
```

**Using poetry:**
```bash
poetry add pymongo python-dotenv
```

### 3. Environment Configuration

Create `.env` file in your project root:
```env
MONGODB_URI="mongodb://user:password@host:port"
LOG_FROM="your_app_name"
```

**Example:**
```env
MONGODB_URI="mongodb://username:password@localhost:27017/mydatabase"
LOG_FROM="my_app"
```

### 4. Setup Logging in Your Code

**Method 1: Application-wide (Recommended)**
```python
# At the top of your main.py or app.py
import logging
from mongo_logger import setup_mongodb_logging

# Set up MongoDB logging (call once at startup)
setup_mongodb_logging()

# Use logging anywhere in your app
logging.info("Application started")
logging.info({"event": "user_login", "user_id": 123})
logging.error("Something went wrong")
```

**Method 2: Module-specific**
```python
# In each module where you need logging
import logging
from mongo_logger import setup_mongodb_logging

# Call setup once per module
setup_mongodb_logging()

# Use logging normally
logger = logging.getLogger(__name__)
logger.info("Module loaded")
```

### 5. Verify Installation

Test your setup:
```python
import logging
from mongo_logger import setup_mongodb_logging

setup_mongodb_logging()

# Test different log types
logging.info("Simple message")
logging.info({"event": "test", "data": [1, 2, 3]})
logging.warning("Test warning")
logging.error("Test error")
```

## Features

### ✅ What You Get

**Automatic Data Type Handling:**
- Strings stay as strings
- Dictionaries preserved as structured MongoDB objects  
- Arrays/lists preserved as arrays
- Tuples/sets converted to arrays
- Robust error handling for any object type

**Rich Metadata (automatically added):**
```python
{
    "_id": ObjectId(),
    "from": "your_app_name",
    "timestamp": "2024-08-28T10:30:00Z",
    "level": "INFO",
    "logger": "root",
    "module": "main.py",
    "line_number": 42,
    "function": "my_function",
    "message": "your log data here",
    "formatted_args": ["arg1", "arg2"]  # When using string formatting
}
```

**Dual Output:**
- Console: Immediate visible logging
- MongoDB: Structured persistent storage

**Thread-Safe:**
- Uses QueueHandler/QueueListener for high-performance logging
- Safe for multi-threaded applications

### 📋 Supported Logging Patterns

All these work seamlessly:
```python
# Simple strings
logging.info("User logged in")

# Formatted strings  
logging.info("User %s logged in at %s", username, timestamp)

# Structured data (preserved in MongoDB)
logging.info({
    "event": "user_login",
    "user_id": 123,
    "metadata": {"ip": "192.168.1.1", "device": "mobile"}
})

# Arrays
logging.info(["item1", "item2", "item3"])

# Different log levels
logging.warning({"alert": "disk_space_low", "usage": 85})
logging.error({"error": "db_connection_failed", "attempts": 3})
```

## Configuration Options

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `MONGODB_URI` | ✅ | MongoDB connection string |
| `LOG_FROM` | ✅ | App identifier (becomes collection name) |

### MongoDB Collections

- **Database**: `logs` (hardcoded, can be modified in `mongo_logger.py`)
- **Collection**: Uses `LOG_FROM` value (e.g., "my_app" → collection "my_app")

## Troubleshooting

### Common Issues

**1. "pymongo not available"**
```bash
pip install pymongo
```

**2. "MongoDB connection failed"**
- Check `MONGODB_URI` format
- Verify network connectivity  
- Confirm MongoDB credentials

**3. "Only first log appears in MongoDB"**
- Add delay before program exit: `time.sleep(2)`
- Ensure QueueListener has time to process logs

**4. "No console output"**
- Console logging is enabled by default
- Check if other logging configuration is interfering

### Testing Connection

Quick connection test:
```python
from pymongo import MongoClient
import os
from dotenv import load_dotenv

load_dotenv()
client = MongoClient(os.getenv("MONGODB_URI"))
print("Connected:", client.admin.command('ping'))
```

## Production Considerations

### Performance
- Uses async queue processing (no blocking)
- Automatic connection retry with fallback
- Handles connection timeouts gracefully

### Security
- Never log sensitive data in production
- Use connection strings with authentication
- Consider MongoDB access controls

### Monitoring
- Monitor MongoDB collection size
- Set up log rotation if needed
- Consider TTL indexes for automatic cleanup

## Examples

### Flask Application
```python
from flask import Flask
import logging
from mongo_logger import setup_mongodb_logging

app = Flask(__name__)

# Set up logging once at startup
setup_mongodb_logging()

@app.route('/')
def home():
    logging.info({
        "event": "page_view",
        "page": "/",
        "user_agent": request.headers.get('User-Agent')
    })
    return "Hello World"
```

### FastAPI Application
```python
from fastapi import FastAPI
import logging
from mongo_logger import setup_mongodb_logging

# Set up logging at startup
setup_mongodb_logging()

app = FastAPI()

@app.get("/")
def read_root():
    logging.info({"event": "api_call", "endpoint": "/"})
    return {"message": "Hello World"}
```

### Django Application
```python
# In settings.py or apps.py
import logging
from mongo_logger import setup_mongodb_logging

# Set up once during Django startup
setup_mongodb_logging()

# Use anywhere in your Django app
def my_view(request):
    logging.info({
        "event": "view_accessed",
        "view": "my_view",
        "user": str(request.user)
    })
```

## Support

- **File Issues**: Check `mongo_logger.py` for error handling
- **Connection Problems**: Verify MongoDB URI and network access
- **Performance**: Adjust queue settings if needed for high-volume logging
- **Custom Fields**: Modify `_insert_log()` method to add custom metadata

---

**That's it!** Your application now has intelligent MongoDB logging with zero configuration beyond the initial setup. All logging calls automatically preserve data structure and include comprehensive metadata.
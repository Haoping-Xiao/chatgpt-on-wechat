# Plugin Development Guide

## Overview

The ChatGPT-on-WeChat framework provides a powerful plugin system that allows developers to extend the bot's functionality. This guide covers everything you need to know to develop, deploy, and manage plugins.

## Plugin Architecture

### Core Components

1. **Plugin Base Class**: All plugins inherit from the `Plugin` class
2. **Event System**: Plugins respond to events using handlers
3. **Configuration Management**: Each plugin can have its own configuration
4. **Plugin Manager**: Manages plugin lifecycle and event distribution

### Plugin Lifecycle

1. **Discovery**: Plugin manager scans the plugins directory
2. **Loading**: Plugin classes are instantiated
3. **Registration**: Event handlers are registered
4. **Execution**: Events trigger registered handlers
5. **Cleanup**: Resources are released when plugin is disabled

## Creating Your First Plugin

### Basic Plugin Structure

```python
# plugins/my_plugin/main.py
import os
from plugins.plugin import Plugin
from plugins.event import Event, EventContext, EventAction
from bridge.reply import Reply, ReplyType
from bridge.context import ContextType
from common.log import logger

class MyPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context,
            Event.ON_DECORATE_REPLY: self.on_decorate_reply,
        }
        
        # Load plugin configuration
        self.config = self.load_config()
        if not self.config:
            self.config = self._load_default_config()
            self.save_config(self.config)
        
        logger.info(f"[{self.__class__.__name__}] Plugin initialized")
    
    def _load_default_config(self):
        return {
            "enabled": True,
            "trigger_words": ["hello", "hi"],
            "response": "Hello! I'm a custom plugin."
        }
    
    def on_handle_context(self, e_context: EventContext):
        """Handle incoming messages"""
        if not self.config.get("enabled", False):
            return
        
        context = e_context['context']
        content = context.content.lower().strip()
        
        # Check if message contains trigger words
        if any(word in content for word in self.config.get("trigger_words", [])):
            reply = Reply(ReplyType.TEXT, self.config.get("response"))
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS  # Stop processing other plugins
    
    def on_decorate_reply(self, e_context: EventContext):
        """Modify replies before they are sent"""
        reply = e_context['reply']
        if reply.type == ReplyType.TEXT:
            # Add plugin signature to text replies
            reply.content += "\n\n[Powered by MyPlugin]"
    
    def get_help_text(self, **kwargs):
        help_text = "MyPlugin Help:\n"
        help_text += f"Trigger words: {', '.join(self.config.get('trigger_words', []))}\n"
        help_text += "Type any trigger word to get a custom response."
        return help_text
```

### Plugin Configuration

Create a `config.json` file in your plugin directory:

```json
{
    "enabled": true,
    "trigger_words": ["hello", "hi", "greetings"],
    "response": "Hello! How can I assist you today?",
    "debug": false
}
```

### Plugin Metadata

Create a `plugin.json` file for metadata:

```json
{
    "name": "my_plugin",
    "version": "1.0.0",
    "author": "Your Name",
    "description": "A sample plugin for demonstration",
    "requires": ["bridge", "plugins"],
    "permissions": ["read_messages", "send_replies"]
}
```

## Event System

### Available Events

#### `Event.ON_HANDLE_CONTEXT`
Triggered when processing incoming messages.

```python
def on_handle_context(self, e_context: EventContext):
    context = e_context['context']
    # Access message content
    message = context.content
    # Access context type (personal/group)
    ctype = context.type
    # Access user information
    user_id = context.get('from_user_id')
```

#### `Event.ON_DECORATE_REPLY`
Triggered before sending replies.

```python
def on_decorate_reply(self, e_context: EventContext):
    reply = e_context['reply']
    context = e_context['context']
    # Modify reply content
    if reply.type == ReplyType.TEXT:
        reply.content = f"🤖 {reply.content}"
```

#### `Event.ON_SEND_REPLY`
Triggered when sending replies.

```python
def on_send_reply(self, e_context: EventContext):
    reply = e_context['reply']
    context = e_context['context']
    # Log sent messages
    logger.info(f"Sent reply: {reply.content}")
```

#### `Event.ON_VOICE_RECOGNIZE`
Triggered during voice recognition.

```python
def on_voice_recognize(self, e_context: EventContext):
    voice_file = e_context['voice_file']
    # Custom voice processing
    pass
```

### Event Actions

Control event flow using actions:

```python
from plugins.event import EventAction

# Continue to next plugin
e_context.action = EventAction.CONTINUE

# Stop processing (break)
e_context.action = EventAction.BREAK

# Break and pass to next handler
e_context.action = EventAction.BREAK_PASS
```

## Advanced Plugin Examples

### 1. Weather Plugin

```python
import requests
from plugins.plugin import Plugin
from plugins.event import Event, EventContext, EventAction
from bridge.reply import Reply, ReplyType

class WeatherPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
        self.config = self.load_config()
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        content = context.content.strip()
        
        if content.startswith("天气"):
            city = content.replace("天气", "").strip()
            if city:
                weather_info = self.get_weather(city)
                reply = Reply(ReplyType.TEXT, weather_info)
                e_context['reply'] = reply
                e_context.action = EventAction.BREAK_PASS
    
    def get_weather(self, city):
        try:
            api_key = self.config.get("api_key")
            url = f"http://api.openweathermap.org/data/2.5/weather"
            params = {"q": city, "appid": api_key, "units": "metric", "lang": "zh"}
            response = requests.get(url, params=params)
            data = response.json()
            
            if response.status_code == 200:
                weather = data['weather'][0]['description']
                temp = data['main']['temp']
                return f"{city}当前天气：{weather}，温度：{temp}°C"
            else:
                return "抱歉，无法获取天气信息"
        except Exception as e:
            return f"天气查询出错：{str(e)}"
```

### 2. Database Integration Plugin

```python
import sqlite3
from plugins.plugin import Plugin
from plugins.event import Event, EventContext, EventAction
from bridge.reply import Reply, ReplyType

class DatabasePlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
        self.db_path = os.path.join(self.path, "plugin_data.db")
        self.init_database()
    
    def init_database(self):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS user_notes (
                user_id TEXT,
                note TEXT,
                timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        conn.commit()
        conn.close()
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        content = context.content.strip()
        user_id = context.get('from_user_id')
        
        if content.startswith("记录 "):
            note = content[3:].strip()
            self.save_note(user_id, note)
            reply = Reply(ReplyType.TEXT, "笔记已保存！")
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
        
        elif content == "我的笔记":
            notes = self.get_user_notes(user_id)
            reply = Reply(ReplyType.TEXT, notes)
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
    
    def save_note(self, user_id, note):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute("INSERT INTO user_notes (user_id, note) VALUES (?, ?)", 
                      (user_id, note))
        conn.commit()
        conn.close()
    
    def get_user_notes(self, user_id):
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        cursor.execute("SELECT note, timestamp FROM user_notes WHERE user_id = ? ORDER BY timestamp DESC LIMIT 5", 
                      (user_id,))
        notes = cursor.fetchall()
        conn.close()
        
        if notes:
            result = "您的最新笔记：\n"
            for note, timestamp in notes:
                result += f"• {note} ({timestamp})\n"
            return result
        else:
            return "您还没有保存任何笔记。"
```

### 3. Multi-Modal Plugin (Image + Text)

```python
import base64
from plugins.plugin import Plugin
from plugins.event import Event, EventContext, EventAction
from bridge.reply import Reply, ReplyType
from bridge.context import ContextType

class VisionPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        
        # Handle image messages
        if context.type == ContextType.IMAGE:
            image_path = context.content
            description = self.analyze_image(image_path)
            reply = Reply(ReplyType.TEXT, f"图片分析结果：{description}")
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
    
    def analyze_image(self, image_path):
        try:
            # Convert image to base64
            with open(image_path, "rb") as image_file:
                encoded_string = base64.b64encode(image_file.read()).decode()
            
            # Call vision API (example with OpenAI)
            from bridge.bridge import Bridge
            bridge = Bridge()
            
            # Prepare vision request
            vision_prompt = {
                "role": "user",
                "content": [
                    {"type": "text", "text": "请描述这张图片的内容"},
                    {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{encoded_string}"}}
                ]
            }
            
            # Get response (simplified)
            return "这是一张包含多个元素的图片..."
        except Exception as e:
            return f"图片分析失败：{str(e)}"
```

## Plugin Configuration Management

### Dynamic Configuration

```python
class ConfigurablePlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        content = context.content.strip()
        
        # Allow users to configure plugin settings
        if content.startswith("设置"):
            self.handle_config_command(content, e_context)
    
    def handle_config_command(self, content, e_context):
        parts = content.split()
        if len(parts) >= 3:
            action = parts[1]
            key = parts[2]
            
            if action == "查看":
                value = self.config.get(key, "未设置")
                reply = Reply(ReplyType.TEXT, f"{key}: {value}")
            elif action == "修改" and len(parts) >= 4:
                value = " ".join(parts[3:])
                self.config[key] = value
                self.save_config(self.config)
                reply = Reply(ReplyType.TEXT, f"已将 {key} 设置为: {value}")
            else:
                reply = Reply(ReplyType.TEXT, "用法：设置 查看/修改 键名 [值]")
        else:
            reply = Reply(ReplyType.TEXT, "用法：设置 查看/修改 键名 [值]")
        
        e_context['reply'] = reply
        e_context.action = EventAction.BREAK_PASS
```

## Plugin Deployment

### Directory Structure

```
plugins/
├── my_plugin/
│   ├── __init__.py
│   ├── main.py
│   ├── config.json
│   ├── plugin.json
│   ├── requirements.txt
│   └── README.md
```

### Installation Script

```bash
#!/bin/bash
# install_plugin.sh

PLUGIN_NAME=$1
PLUGIN_URL=$2

if [ -z "$PLUGIN_NAME" ] || [ -z "$PLUGIN_URL" ]; then
    echo "Usage: $0 <plugin_name> <plugin_url>"
    exit 1
fi

# Create plugin directory
mkdir -p plugins/$PLUGIN_NAME

# Download plugin files
curl -o plugins/$PLUGIN_NAME/main.py $PLUGIN_URL/main.py
curl -o plugins/$PLUGIN_NAME/config.json $PLUGIN_URL/config.json
curl -o plugins/$PLUGIN_NAME/plugin.json $PLUGIN_URL/plugin.json

# Install dependencies
if [ -f plugins/$PLUGIN_NAME/requirements.txt ]; then
    pip install -r plugins/$PLUGIN_NAME/requirements.txt
fi

echo "Plugin $PLUGIN_NAME installed successfully!"
```

## Testing and Debugging

### Plugin Testing Framework

```python
import unittest
from unittest.mock import Mock, patch
from plugins.my_plugin.main import MyPlugin
from plugins.event import EventContext
from bridge.context import Context
from bridge.reply import Reply, ReplyType

class TestMyPlugin(unittest.TestCase):
    def setUp(self):
        self.plugin = MyPlugin()
    
    def test_trigger_word_response(self):
        # Create mock context
        context = Context()
        context.content = "hello"
        context.type = "personal"
        
        # Create event context
        e_context = EventContext()
        e_context['context'] = context
        
        # Test plugin response
        self.plugin.on_handle_context(e_context)
        
        # Verify reply
        self.assertIn('reply', e_context)
        self.assertEqual(e_context['reply'].type, ReplyType.TEXT)
        self.assertIn("Hello", e_context['reply'].content)
    
    def test_configuration_loading(self):
        # Test configuration is loaded properly
        self.assertIsInstance(self.plugin.config, dict)
        self.assertIn('enabled', self.plugin.config)
        self.assertIn('trigger_words', self.plugin.config)

if __name__ == '__main__':
    unittest.main()
```

### Debug Mode

```python
class DebugPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.debug = self.config.get('debug', False)
        
    def debug_log(self, message):
        if self.debug:
            logger.debug(f"[{self.__class__.__name__}] {message}")
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        self.debug_log(f"Processing message: {context.content}")
        
        # Plugin logic here
        
        self.debug_log(f"Generated reply: {e_context.get('reply', 'None')}")
```

## Best Practices

### 1. Error Handling
Always wrap plugin logic in try-catch blocks:

```python
def on_handle_context(self, e_context: EventContext):
    try:
        # Plugin logic
        pass
    except Exception as e:
        logger.error(f"Plugin error: {str(e)}")
        reply = Reply(ReplyType.ERROR, "插件处理出错")
        e_context['reply'] = reply
```

### 2. Resource Management
Clean up resources properly:

```python
def __del__(self):
    # Close database connections, file handles, etc.
    if hasattr(self, 'db_connection'):
        self.db_connection.close()
```

### 3. Performance Considerations
- Use async operations for I/O intensive tasks
- Cache frequently accessed data
- Implement rate limiting for API calls

### 4. Security
- Validate user inputs
- Sanitize file paths
- Use secure API practices

### 5. User Experience
- Provide clear help text
- Use consistent command formats
- Give meaningful error messages

## Plugin Distribution

### Publishing to Plugin Registry

```python
# publish_plugin.py
import json
import requests

def publish_plugin(plugin_path):
    with open(f"{plugin_path}/plugin.json", 'r') as f:
        metadata = json.load(f)
    
    # Package plugin files
    # Upload to registry
    # Update plugin index
    pass
```

This guide provides a comprehensive overview of plugin development for the ChatGPT-on-WeChat framework. Use these examples and patterns to create powerful extensions that enhance the bot's capabilities.
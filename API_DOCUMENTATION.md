# ChatGPT-on-WeChat API Documentation

## Table of Contents

1. [Project Overview](#project-overview)
2. [Core Architecture](#core-architecture)
3. [Configuration API](#configuration-api)
4. [Bot API](#bot-api)
5. [Channel API](#channel-api)
6. [Bridge API](#bridge-api)
7. [Plugin API](#plugin-api)
8. [Common Utilities](#common-utilities)
9. [Voice & Translation APIs](#voice--translation-apis)
10. [Usage Examples](#usage-examples)
11. [Supported Models & Channels](#supported-models--channels)

## Project Overview

ChatGPT-on-WeChat (CoW) is a comprehensive chatbot framework that enables AI-powered conversations across multiple platforms including WeChat, Enterprise WeChat, DingTalk, Feishu, and more. It supports various AI models from OpenAI, Claude, Gemini, Baidu, Xunfei, and others.

### Key Features

- **Multi-Platform Support**: WeChat, Enterprise WeChat, DingTalk, Feishu, Terminal, Web
- **Multiple AI Models**: GPT-3.5/4, Claude, Gemini, Wenxin, Xunfei, ChatGLM, Moonshot, etc.
- **Voice Processing**: Speech recognition and text-to-speech in multiple languages
- **Image Generation**: DALL-E integration for image creation
- **Plugin System**: Extensible architecture for custom functionality
- **Knowledge Base**: Custom knowledge base integration via LinkAI
- **Multi-modal**: Text, voice, and image processing capabilities

---

## Core Architecture

### Application Entry Point

The main application entry point provides the following functions:

#### `app.py`

```python
def start_channel(channel_name: str)
```
Starts a specific channel with plugin loading and LinkAI integration.

**Parameters:**
- `channel_name` (str): Channel type to start (wx, terminal, wechatmp, etc.)

**Example:**
```python
start_channel("wx")  # Start WeChat channel
start_channel("terminal")  # Start terminal channel
```

```python
def run()
```
Main application runner that loads configuration, sets up signal handlers, and starts the selected channel.

**Example:**
```python
# Start the application
if __name__ == "__main__":
    run()
```

---

## Configuration API

### Configuration Management

The configuration system provides comprehensive settings management for all aspects of the bot.

#### `config.py`

```python
class Config(dict)
```
Main configuration class extending dict with validation and user data management.

**Methods:**

```python
def get(self, key: str, default=None) -> Any
```
Get configuration value with optional default.

**Parameters:**
- `key` (str): Configuration key
- `default` (Any): Default value if key not found

**Example:**
```python
model = conf().get("model", "gpt-3.5-turbo")
api_key = conf().get("open_ai_api_key")
```

```python
def get_user_data(self, user: str) -> dict
```
Get user-specific data dictionary.

**Parameters:**
- `user` (str): User identifier

**Returns:**
- `dict`: User data dictionary

```python
def load_user_datas(self)
def save_user_datas(self)
```
Load and save persistent user data.

**Global Configuration Functions:**

```python
def load_config()
```
Load configuration from config.json with environment variable overrides.

```python
def conf() -> Config
```
Get global configuration instance.

```python
def pconf(plugin_name: str) -> dict
```
Get plugin-specific configuration.

**Parameters:**
- `plugin_name` (str): Plugin name

**Returns:**
- `dict`: Plugin configuration

### Available Configuration Options

#### AI Model Configuration
```json
{
  "model": "gpt-3.5-turbo",
  "open_ai_api_key": "your-api-key",
  "open_ai_api_base": "https://api.openai.com/v1",
  "bot_type": "chatGPT",
  "use_azure_chatgpt": false,
  "temperature": 0.9,
  "conversation_max_tokens": 1000
}
```

#### Channel Configuration
```json
{
  "channel_type": "wx",
  "single_chat_prefix": ["bot", "@bot"],
  "group_chat_prefix": ["@bot"],
  "group_name_white_list": ["ChatGPT测试群"],
  "subscribe_msg": "Welcome message"
}
```

#### Voice & Image Configuration
```json
{
  "speech_recognition": true,
  "voice_reply_voice": false,
  "text_to_voice": "openai",
  "voice_to_text": "openai",
  "image_create_prefix": ["画", "看", "找"],
  "text_to_image": "dall-e-2"
}
```

---

## Bot API

### Base Bot Interface

#### `bot/bot.py`

```python
class Bot(object)
```
Abstract base class for all bot implementations.

**Methods:**

```python
def reply(self, query: str, context: Context = None) -> Reply
```
Generate bot response to user query.

**Parameters:**
- `query` (str): User input message
- `context` (Context): Conversation context

**Returns:**
- `Reply`: Bot response object

### Bot Factory

#### `bot/bot_factory.py`

```python
def create_bot(bot_type: str) -> Bot
```
Factory function to create bot instances.

**Parameters:**
- `bot_type` (str): Bot type (chatGPT, claude, gemini, etc.)

**Returns:**
- `Bot`: Bot instance

**Example:**
```python
from bot.bot_factory import create_bot

# Create different bot types
openai_bot = create_bot("chatGPT")
claude_bot = create_bot("claudeAPI")
gemini_bot = create_bot("gemini")
```

### Session Management

#### `bot/session_manager.py`

```python
class SessionManager
```
Manages conversation sessions and context.

**Methods:**

```python
def session_query(self, query: str, session_id: str) -> dict
```
Process query within a session context.

```python
def clear_session(self, session_id: str)
```
Clear session history.

```python
def clear_all_session()
```
Clear all session data.

---

## Channel API

### Base Channel Interface

#### `channel/channel.py`

```python
class Channel(object)
```
Abstract base class for all channel implementations.

**Attributes:**
- `channel_type` (str): Channel identifier
- `NOT_SUPPORT_REPLYTYPE` (list): Unsupported reply types

**Methods:**

```python
def startup(self)
```
Initialize and start the channel.

```python
def handle_text(self, msg)
```
Process received text messages.

**Parameters:**
- `msg`: Message object

```python
def send(self, reply: Reply, context: Context)
```
Send reply message to user.

**Parameters:**
- `reply` (Reply): Response to send
- `context` (Context): Conversation context

```python
def build_reply_content(self, query: str, context: Context = None) -> Reply
```
Build reply content using the bridge.

```python
def build_voice_to_text(self, voice_file) -> Reply
```
Convert voice to text.

```python
def build_text_to_voice(self, text: str) -> Reply
```
Convert text to voice.

### Channel Factory

#### `channel/channel_factory.py`

```python
def create_channel(channel_type: str) -> Channel
```
Factory function to create channel instances.

**Parameters:**
- `channel_type` (str): Channel type (wx, wechatmp, dingtalk, etc.)

**Returns:**
- `Channel`: Channel instance

**Example:**
```python
from channel.channel_factory import create_channel

# Create different channel types
wechat_channel = create_channel("wx")
dingtalk_channel = create_channel("dingtalk")
terminal_channel = create_channel("terminal")
```

### Chat Channel

#### `channel/chat_channel.py`

```python
class ChatChannel(Channel)
```
Enhanced channel with conversation management and multithreading support.

**Methods:**

```python
def produce(self, context: Context)
```
Produce response for given context.

```python
def consume(self)
```
Consume and process message queue.

```python
def cancel_session(self, session_id: str)
```
Cancel active session.

```python
def cancel_all_session()
```
Cancel all active sessions.

### Message Types

#### `channel/chat_message.py`

```python
class ChatMessage
```
Represents a chat message with metadata.

**Attributes:**
- `msg_type` (str): Message type (TEXT, VOICE, IMAGE, etc.)
- `content` (str): Message content
- `ctype` (str): Context type (group/personal)
- `from_user_id` (str): Sender ID
- `to_user_id` (str): Recipient ID
- `actual_user_id` (str): Actual user ID (for groups)

---

## Bridge API

### Core Bridge

#### `bridge/bridge.py`

```python
@singleton
class Bridge(object)
```
Central integration layer connecting bots and channels.

**Methods:**

```python
def get_bot(self, typename: str) -> Bot
```
Get bot instance by type.

**Parameters:**
- `typename` (str): Bot type (chat, voice_to_text, text_to_voice, translate)

**Returns:**
- `Bot`: Bot instance

```python
def fetch_reply_content(self, query: str, context: Context) -> Reply
```
Fetch chat reply from configured bot.

```python
def fetch_voice_to_text(self, voiceFile) -> Reply
```
Convert voice file to text.

```python
def fetch_text_to_voice(self, text: str) -> Reply
```
Convert text to voice.

```python
def fetch_translate(self, text: str, from_lang="", to_lang="en") -> Reply
```
Translate text between languages.

```python
def reset_bot(self)
```
Reset bot routing configuration.

**Example:**
```python
from bridge.bridge import Bridge

bridge = Bridge()
reply = bridge.fetch_reply_content("Hello", context)
voice_reply = bridge.fetch_text_to_voice("Hello world")
```

### Context Management

#### `bridge/context.py`

```python
class Context
```
Represents conversation context and metadata.

**Attributes:**
- `type` (str): Context type
- `content` (str): Message content
- `kwargs` (dict): Additional parameters

### Reply Management

#### `bridge/reply.py`

```python
class Reply
```
Represents bot response.

**Attributes:**
- `type` (ReplyType): Reply type (TEXT, VOICE, IMAGE, etc.)
- `content` (str): Reply content

```python
class ReplyType(Enum)
```
Enumeration of reply types:
- `TEXT`: Text message
- `VOICE`: Voice message
- `IMAGE`: Image message
- `IMAGE_URL`: Image URL
- `INFO`: Information message
- `ERROR`: Error message

---

## Plugin API

### Base Plugin

#### `plugins/plugin.py`

```python
class Plugin
```
Base class for all plugins.

**Attributes:**
- `handlers` (dict): Event handlers
- `name` (str): Plugin name
- `path` (str): Plugin directory path

**Methods:**

```python
def load_config(self) -> dict
```
Load plugin configuration.

**Returns:**
- `dict`: Plugin configuration

```python
def save_config(self, config: dict)
```
Save plugin configuration.

**Parameters:**
- `config` (dict): Configuration to save

```python
def get_help_text(self, **kwargs) -> str
```
Get plugin help text.

**Returns:**
- `str`: Help text

```python
def reload(self)
```
Reload plugin.

**Example:**
```python
class MyPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        if context.content.startswith("hello"):
            reply = Reply(ReplyType.TEXT, "Hello! How can I help you?")
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
```

### Plugin Manager

#### `plugins/plugin_manager.py`

```python
class PluginManager
```
Manages plugin lifecycle and events.

**Methods:**

```python
def register(self, name: str, plugin: Plugin)
```
Register a plugin.

```python
def load_plugins(self)
```
Load all plugins from plugins directory.

```python
def reload_plugin(self, name: str)
```
Reload specific plugin.

```python
def emit_event(self, event: Event, *args, **kwargs)
```
Emit event to all registered plugins.

### Plugin Events

#### `plugins/event.py`

```python
class Event(Enum)
```
Plugin event types:
- `ON_HANDLE_CONTEXT`: Handle message context
- `ON_DECORATE_REPLY`: Decorate reply
- `ON_SEND_REPLY`: Send reply
- `ON_VOICE_RECOGNIZE`: Voice recognition
- `ON_TEXT_TO_VOICE`: Text to voice conversion

```python
class EventContext
```
Event context for plugin handlers.

**Methods:**

```python
def set_action(self, action: EventAction)
```
Set action for event processing.

```python
def get_args(self) -> dict
```
Get event arguments.

---

## Common Utilities

### Constants

#### `common/const.py`

**Bot Types:**
- `CHATGPT`: OpenAI ChatGPT
- `CLAUDE`: Claude AI
- `GEMINI`: Google Gemini
- `BAIDU`: Baidu Wenxin
- `XUNFEI`: Xunfei Spark
- `ZHIPU_AI`: ChatGLM
- `MOONSHOT`: Kimi

**Models:**
- GPT: `GPT35`, `GPT4`, `GPT_4o`, `GPT_4o_MINI`
- Claude: `CLAUDE_3_OPUS`, `CLAUDE_35_SONNET`, `CLAUDE_3_HAIKU`
- Gemini: `GEMINI_PRO`, `GEMINI_15_PRO`

**Channels:**
- `FEISHU`: Feishu/Lark
- `DINGTALK`: DingTalk

### Utilities

#### `common/utils.py`

```python
def fsize(file_path: str) -> str
```
Get human-readable file size.

```python
def get_path_suffix(path: str) -> str
```
Get file extension from path.

```python
def split_string_by_utf8_length(string: str, max_length: int) -> list
```
Split string by UTF-8 byte length.

```python
def encode_image_to_base64(image_path: str) -> str
```
Encode image to base64 string.

```python
def download_and_compress_image(url: str, filename: str, quality=95) -> str
```
Download and compress image from URL.

### Logging

#### `common/log.py`

```python
logger
```
Global logger instance configured for the application.

**Usage:**
```python
from common.log import logger

logger.info("Information message")
logger.error("Error message")
logger.debug("Debug message")
```

### Memory & Caching

#### `common/expired_dict.py`

```python
class ExpiredDict(dict)
```
Dictionary with automatic expiration of entries.

**Methods:**

```python
def __init__(self, expires_in_seconds: int)
```
Initialize with expiration time.

```python
def get(self, key, default=None)
```
Get value, returns None if expired.

#### `common/token_bucket.py`

```python
class TokenBucket
```
Token bucket for rate limiting.

**Methods:**

```python
def __init__(self, capacity: int, fill_rate: float)
```
Initialize token bucket.

```python
def consume(self, tokens: int = 1) -> bool
```
Consume tokens, returns False if not enough tokens.

---

## Voice & Translation APIs

### Voice Factory

#### `voice/factory.py`

```python
def create_voice(voice_type: str)
```
Create voice processing instance.

**Parameters:**
- `voice_type` (str): Voice service type (openai, baidu, google, azure, etc.)

**Returns:**
- Voice processor instance

### Translation Factory

#### `translate/factory.py`

```python
def create_translator(translate_type: str)
```
Create translation service instance.

**Parameters:**
- `translate_type` (str): Translation service type (baidu, google, etc.)

**Returns:**
- Translator instance

---

## Usage Examples

### Basic Bot Setup

```python
# 1. Configure the bot
from config import load_config, conf

load_config()

# 2. Create and start a channel
from channel.channel_factory import create_channel

channel = create_channel("terminal")
channel.startup()
```

### Custom Plugin Development

```python
from plugins.plugin import Plugin
from plugins.event import Event, EventContext
from bridge.reply import Reply, ReplyType

class GreetingPlugin(Plugin):
    def __init__(self):
        super().__init__()
        self.handlers = {
            Event.ON_HANDLE_CONTEXT: self.on_handle_context
        }
    
    def on_handle_context(self, e_context: EventContext):
        context = e_context['context']
        if context.content.startswith("hello"):
            reply = Reply(ReplyType.TEXT, "Hello! How can I help you?")
            e_context['reply'] = reply
            e_context.action = EventAction.BREAK_PASS
```

### Multi-Model Integration

```python
from bridge.bridge import Bridge
from bridge.context import Context

bridge = Bridge()

# Use different models for different tasks
context = Context()
context.content = "Explain quantum computing"

# Get response from configured model
reply = bridge.fetch_reply_content(context.content, context)

# Convert to voice
voice_reply = bridge.fetch_text_to_voice(reply.content)

# Translate to another language
translated = bridge.fetch_translate(reply.content, to_lang="zh")
```

### Voice Processing

```python
# Process voice message
voice_file = "path/to/voice.wav"
text_reply = bridge.fetch_voice_to_text(voice_file)

# Convert text response to voice
response_text = "This is the bot response"
voice_response = bridge.fetch_text_to_voice(response_text)
```

### Configuration Management

```python
# Access configuration
model_name = conf().get("model", "gpt-3.5-turbo")
api_key = conf().get("open_ai_api_key")

# User data management
user_data = conf().get_user_data("user123")
user_data["preferences"] = {"language": "en"}
conf().save_user_datas()
```

---

## Supported Models & Channels

### AI Models

#### OpenAI Models
- GPT-3.5-turbo (variants: 0125, 1106, 16k)
- GPT-4 (variants: turbo, o, o-mini, 32k)
- DALL-E (2, 3) for image generation
- Whisper for speech recognition
- TTS for text-to-speech

#### Other AI Providers
- **Claude**: 3 Opus, 3.5 Sonnet, 3 Haiku
- **Gemini**: Pro, 1.5 Flash, 1.5 Pro, 2.0 Flash
- **Chinese Models**: Wenxin (Baidu), Xunfei Spark, ChatGLM, Qwen, Moonshot
- **Other**: MiniMax, LinkAI platform integration

### Communication Channels

#### Instant Messaging
- **WeChat**: Personal and group chats
- **Enterprise WeChat**: Corporate application integration
- **DingTalk**: Enterprise communication platform
- **Feishu/Lark**: Collaboration platform

#### Web & API
- **WeChat MP**: WeChat Official Account
- **Web Interface**: Browser-based chat
- **Terminal**: Command-line interface

### Voice & Speech Services

#### Speech Recognition
- OpenAI Whisper
- Baidu Speech API
- Google Speech API
- Azure Speech Services
- Xunfei Speech
- Ali Speech

#### Text-to-Speech
- OpenAI TTS
- Google TTS
- Azure TTS
- Baidu TTS
- Edge TTS (online)
- ElevenLabs
- PyTTS (offline)

### Image Generation
- DALL-E 2/3
- Stable Diffusion
- Midjourney (via LinkAI)
- CogView-3

### Translation Services
- Baidu Translate
- Google Translate

---

## Error Handling

### Common Error Types

```python
class BotException(Exception):
    """Base exception for bot errors"""
    pass

class ConfigException(Exception):
    """Configuration related errors"""
    pass

class ChannelException(Exception):
    """Channel communication errors"""
    pass
```

### Error Handling Best Practices

```python
try:
    reply = bridge.fetch_reply_content(query, context)
except Exception as e:
    logger.error(f"Error generating reply: {e}")
    reply = Reply(ReplyType.ERROR, "Sorry, I encountered an error.")
```

---

## Rate Limiting & Performance

### Configuration Options

```json
{
  "rate_limit_chatgpt": 20,
  "rate_limit_dalle": 50,
  "request_timeout": 180,
  "timeout": 120,
  "conversation_max_tokens": 1000,
  "concurrency_in_session": 1
}
```

### Usage

```python
from common.token_bucket import TokenBucket

# Create rate limiter
limiter = TokenBucket(capacity=20, fill_rate=1.0)

if limiter.consume():
    # Process request
    pass
else:
    # Rate limited
    pass
```

---

## Security & Best Practices

### API Key Management
- Store sensitive keys in environment variables
- Use configuration templates for sharing
- Implement key rotation strategies

### Content Filtering
- Configure sensitive word filtering via plugins
- Implement content moderation
- Set up user blacklists

### Session Management
- Set appropriate session timeouts
- Clear sensitive data regularly
- Implement user data privacy controls

---

This documentation provides comprehensive coverage of all public APIs, functions, and components in the ChatGPT-on-WeChat project. Each section includes detailed method signatures, parameters, return values, and practical usage examples to help developers effectively utilize and extend the framework.
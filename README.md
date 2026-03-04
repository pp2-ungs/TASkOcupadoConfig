# TASkOcupadoConfig

Configuration and extensibility module for the **TASkOcupado** system. This repository provides all the runtime configuration files and pre-built extension JARs that the application needs to load people, tasks, and external notifiers dynamically.

## Overview

TASkOcupado is a task-scheduling desktop application that assigns tasks to people and notifies them in real time through pluggable notification channels. The system is composed of several modules:

| Repository | Description |
|---|---|
| [TASkOcupadoCore](https://github.com/pp2-ungs/TASkOcupadoCore) | Domain model and core logic — task scheduling and Observer-pattern notifications |
| [TASkOcupadoUI](https://github.com/pp2-ungs/TASkOcupadoUI) | Desktop UI layer — MVC architecture and real-time UI updates |
| **TASkOcupadoConfig** *(this repo)* | Runtime configuration and extension JARs |
| [EmailNotifier](https://github.com/pp2-ungs/EmailNotifier) | Pluggable email notifier (SMTP) |
| [TelegramNotifier](https://github.com/pp2-ungs/TelegramNotifier) | Pluggable Telegram notifier |
| [AssignmentLogger](https://github.com/pp2-ungs/AssignmentLogger) | External service that logs all task assignment history |

## Repository Structure

```
.TASkOcupado/
├── Extensions/
│   ├── EmailNotifier.jar        # Pre-built email notification extension
│   └── TelegramNotifier.jar     # Pre-built Telegram notification extension
└── Resources/
    ├── config.properties        # Core configuration (content loader class)
    ├── Person.json              # List of people registered in the system
    ├── Task.json                # List of available tasks
    ├── Email.json               # Email addresses mapped by person name
    ├── Telegram.json            # Telegram chat IDs mapped by person name
    └── Classes/
        └── loaders/
            ├── JSONLoader.class         # Loads resources from JSON files
            └── ExternalSystemLoader.class  # Discovers and loads extension JARs
```

## Configuration Files

### `config.properties`

Specifies which `ContentLoader` implementation the core should use at runtime.

```properties
core.ContentLoader = loaders.JSONLoader
```

Setting this to `loaders.JSONLoader` tells the system to read all data (people, tasks, notification addresses) from the JSON files in the `Resources/` directory.

### `Person.json`

Defines the list of people that can be assigned tasks. Each entry has a `name` field.

```json
[
  { "name": "Ximena Ebertz" },
  { "name": "Hernán Rondelli" }
]
```

### `Task.json`

Defines the pool of tasks that can be assigned. Each entry has a `description` field.

```json
[
  { "description": "Ir al gym" },
  { "description": "Dormir" }
]
```

### `Email.json`

Maps each person's name to their email address. Used by the `EmailNotifier` extension to send assignment alerts via SMTP.

```json
{
  "Ximena Ebertz": "ebertz.xime@gmail.com",
  "Hernán Rondelli": "lucifer.unix.cabj@gmail.com"
}
```

### `Telegram.json`

Maps each person's name to their Telegram chat ID. Used by the `TelegramNotifier` extension to send instant Telegram messages when a task is assigned.

```json
{
  "Ximena Ebertz": 850210514,
  "Hernán Rondelli": 70417546
}
```

## Extensions

The `Extensions/` directory contains the compiled JAR files for the pluggable notifiers. At startup the `ExternalSystemLoader` scans this directory, discovers every JAR, and registers the observers it finds with the core notification system — no code changes are needed to add or remove a notifier.

| JAR | Source | Trigger |
|---|---|---|
| `EmailNotifier.jar` | [pp2-ungs/EmailNotifier](https://github.com/pp2-ungs/EmailNotifier) | Sends an email when a task is assigned |
| `TelegramNotifier.jar` | [pp2-ungs/TelegramNotifier](https://github.com/pp2-ungs/TelegramNotifier) | Sends a Telegram message when a task is assigned |

## How It Works

1. **TASkOcupadoCore** reads `config.properties` to determine which `ContentLoader` to use.
2. The `JSONLoader` reads `Person.json` and `Task.json` to populate the domain model.
3. The `ExternalSystemLoader` scans the `Extensions/` directory, loads each JAR, and registers the notifiers as Observers.
4. When **TASkOcupadoUI** assigns a task to a person, all registered Observers are notified.
5. Each notifier looks up the recipient's contact information from `Email.json` or `Telegram.json` and delivers the alert.

## Usage

Clone this repository into the root of your TASkOcupado workspace so that the `.TASkOcupado/` directory sits alongside the application at runtime:

```bash
git clone https://github.com/pp2-ungs/TASkOcupadoConfig.git
```

The application will automatically discover the `.TASkOcupado/` directory and load everything it needs from it.

To **add a new person**, append an entry to `Person.json` and, if applicable, add their contact details to `Email.json` and/or `Telegram.json`.

To **add a new task**, append an entry to `Task.json`.

To **add a new notifier extension**, drop its compiled JAR into the `Extensions/` directory — the loader will pick it up on the next run without any further configuration.

## Authors

- Ximena Ebertz
- Gonzalo Javier López
- Javier Martínez-Viademonte
- Matías Raia
- Hernán Rondelli
- Lautaro Tacchini

*Universidad Nacional de General Sarmiento (UNGS) — PP2*

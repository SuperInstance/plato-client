# plato-client

> Python client library for the PLATO knowledge server — query, submit, and verify knowledge tiles

Part of the [SuperInstance](https://github.com/SuperInstance) music constraint theory ecosystem. Provides synchronous (`PLATOClient`) and asynchronous (`AsyncPLATOClient`) Python clients for interacting with the PLATO knowledge server — a knowledge management system that stores, queries, and verifies structured knowledge tiles.

## What It Does

The PLATO system organizes knowledge into **tiles** — immutable records pairing a question with an answer, tagged by domain, source, and confidence. This client lets you query tiles by room and topic, submit new tiles through a gate (with confidence scores and provenance), and verify tiles against server-side constraints.

The client supports both synchronous usage (for scripts and notebooks) and async usage (for integration into async frameworks), both built on `httpx` with proper error handling, timeouts, and context manager support.

## Key Features

- **Synchronous + async clients** — `PLATOClient` and `AsyncPLATOClient` with identical APIs
- **Tile model** — immutable dataclass with SHA-256 content hashing and serialization
- **Room-based organization** — query and filter knowledge by room/prefix
- **Gate submission** — submit tiles with domain, confidence, source, and tags
- **Verification API** — verify tiles against server-side constraint checks
- **Proper error handling** — `PLATOError` with status codes and response bodies
- **Context managers** — `with` and `async with` support for resource cleanup

## Installation

```bash
pip install cocapn
```

Or from source:

```bash
git clone https://github.com/SuperInstance/plato-client.git
cd plato-client
pip install -e ".[dev]"
```

Requires Python 3.11+.

## Quick Start

### Synchronous

```python
from cocapn import PLATOClient, Tile

with PLATOClient() as client:
    # Health check
    print(client.health())

    # List rooms
    rooms = client.list_rooms(prefix="marine")
    print(rooms)

    # Query tiles
    tiles = client.query(room="marine", query="nmea parser", limit=5)
    for tile in tiles:
        print(f"{tile.question} → {tile.answer}")

    # Submit a tile
    result = client.submit(
        domain="marine",
        question="What does NMEA stand for?",
        answer="National Marine Electronics Association",
        source="wikipedia",
        confidence=0.95,
        tags=["nmea", "acronym"],
    )
    print(result)
```

### Asynchronous

```python
import asyncio
from cocapn import AsyncPLATOClient

async def main():
    async with AsyncPLATOClient() as client:
        print(await client.health())

        tiles = await client.query("marine", limit=3)
        for tile in tiles:
            print(f"{tile.question} → {tile.answer}")

asyncio.run(main())
```

### Working with Tiles directly

```python
from cocapn import Tile

tile = Tile(
    domain="math",
    question="2 + 2",
    answer="4",
    source="arithmetic",
    confidence=1.0,
    tags=["basic"],
)

print(tile.to_dict())      # Serialize
print(tile._hash)           # SHA-256 content hash (auto-computed)
```

## API Reference

### `PLATOClient(base_url, timeout)`

Synchronous client. `base_url` defaults to the public PLATO instance.

| Method | Description |
|---|---|
| `health()` | Check server health → `dict` |
| `list_rooms(prefix=None)` | List rooms, optionally filtered → `list[str]` |
| `query(room, query=None, limit=10)` | Query tiles from a room → `list[Tile]` |
| `submit(domain, question, answer, source, confidence, tags=None)` | Submit tile through gate → `dict` |
| `verify(tile_id)` | Verify tile against constraints → `dict` |
| `close()` | Close HTTP connection |

Supports `with PLATOClient() as client:` context manager.

### `AsyncPLATOClient(base_url, timeout)`

Async variant — identical API, all methods are coroutines. Supports `async with`.

### `Tile`

Immutable dataclass representing a knowledge tile.

| Field | Type | Description |
|---|---|---|
| `domain` | `str` | Knowledge domain |
| `question` | `str` | Question / key |
| `answer` | `str` | Answer / value |
| `source` | `str` | Origin or authority |
| `confidence` | `float` | Score in `[0.0, 1.0]` |
| `tags` | `list[str]` | Classification tags |
| `provenance` | `str` | Provenance trail |
| `_hash` | `str` | SHA-256 content hash (auto-computed) |

### `PLATOError`

Exception raised on network failures, timeouts, or non-2xx responses. Attributes: `status_code`, `response_body`.

## Development

```bash
pytest                    # Run tests
ruff check src            # Lint
mypy src                  # Type check
```

## Related Repos

- [**plato-adapters**](https://github.com/SuperInstance/plato-adapters) — Adapters for connecting PLATO to other SuperInstance services
- [**constraint-dsl**](https://github.com/SuperInstance/constraint-dsl) — DSL for defining constraint pipelines
- [**constraint-toolkit**](https://github.com/SuperInstance/constraint-toolkit) — Core constraint satisfaction engine
- [**flux-genome**](https://github.com/SuperInstance/flux-genome) — Genetic evolution of musical genomes

## License

MIT

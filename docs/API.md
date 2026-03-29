# API

## `ARC`

Provides an Adaptive Replacement Cache (ARC) implementation.

### `new ARC(size)`

Creates a new ARC cache instance.

#### Parameters

| Name | Type | Description |
|------|------|-------------|
| `size` | `number` | Maximum cache size |

### ` ARC.get(key)`

Retrieves a value from the cache. Updates access patterns for the ARC algorithm.

#### Parameters

| Name | Type | Description |
|------|------|-------------|
| `key` | `string \| number` | The key to retrieve |

#### Returns

`any \| undefined` - The cached value or `undefined` if not found.

### `ARC.set(key, value)`

Stores a value in the cache. Maintains cache bounds and access patterns.

#### Parameters

| Name | Type | Description |
|------|------|-------------|
| `key` | `string \| number` | The key to set |
| `value` | `any` | The value to cache |

### `ARC.delete(key)`

Removes a key from all internal cache lists.

#### Parameters

| Name | Type | Description |
|------|------|-------------|
| `key` | `string \| number` | The key to delete |

### `ARC.has(key)`

Checks if a key exists in the cache.

#### Parameters

| Name | Type | Description |
|------|------|-------------|
| `key` | `string \| number` | The key to check |

#### Returns

`boolean` - `true` if key exists, `false` otherwise.

## `arc(options)`

Factory function to create an ARC cache instance.

### Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `options` | `Object` | `{}` | Configuration options |
| `options.size` | `number` | `100` | Maximum cache size |

### Returns

`ARC` - New ARC cache instance.

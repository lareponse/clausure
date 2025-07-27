# Clausure

PHP closure generator for SQL clauses.

Functional composition for SQL building through reusable closure functions that generate SQL fragments with proper parameter binding.

Generates SQL with named placeholders (`:name` format) compatible with PDO, or can be adapted for use with mysqli and other database extensions.

## Requirements
- PHP 7.4+

## Installation
```bash
composer require lareponse/clausure
```

## Usage

### Basic Clauses

```php
require 'clausure.php';

// Create closure generators
$select = clause(CLAUSE_SELECT);
$where = clause(CLAUSE_WHERE, '=');

// Generate SQL fragments
[$sql, $bindings] = $select('id', 'username', 'email');
// → ["SELECT id, username, email", []]

[$sql, $bindings] = $where(['id' => 1, 'active' => true]);
// → ["WHERE `id` = :id AND `active` = :active", ['id' => 1, 'active' => true]]
```

### Raw SQL Integration

```php
$where = clause(CLAUSE_WHERE);
[$sql, $bindings] = $where("id = 1 AND username = 'test'");
// → ["WHERE id = 1 AND username = 'test'", []]
```

### Clause Composition

```php
$and = clause(OP_AND, '=');
$or = clause(OP_OR, '=');

[$sql, $bindings] = $and(
    ['id' => 1],
    $or(['username' => 'test', 'email' => 'test@test.com']),
    ['status' => 'active']
);
// → ["(`id` = :id AND (`username` = :username OR `email` = :email) AND `status` = :status)", {...bindings}]
```

### Complete Statements

```php
$select = clause(CLAUSE_SELECT);
$where = clause(CLAUSE_WHERE, '=');
$order = clause(CLAUSE_ORDER_BY);

[$sql, $bindings] = statement(
    $select('id', 'username', 'email'),
    'FROM users',
    $where(['active' => 1, 'verified' => 1, 'deleted_at IS NOT NULL']),
    $order(['created_at' => 'DESC'])
);
// → Complete SELECT statement with bindings
```

### Complex Composition

```php
$select = clause(CLAUSE_SELECT);
$where = clause(CLAUSE_WHERE, '=');
$and = clause(OP_AND, '=');
$or = clause(OP_OR, '>');
$order = clause(CLAUSE_ORDER_BY);

[$sql, $bindings] = statement(
    $select('id', 'username', 'email', 'age'),
    'FROM users',
    $where([
        'active' => 1,
        'verified' => 1,
        $and([
            'department' => 'engineering',
            $or(['age' => 25, 'experience_years' => 3])
        ])
    ]),
    $order(['created_at' => 'DESC'])
);
// → SELECT with nested AND/OR conditions and bindings


### Advanced Features

```php
// SELECT with aliases
clause(CLAUSE_SELECT)(['id', 'COUNT(*)', 'name', 'username' => 'email']);
// → "SELECT id, COUNT(*), name, email AS `username`"

// OR conditions
clause(CLAUSE_WHERE | OP_OR, '=')(['role' => 'admin', 'role' => 'moderator']);
// → "WHERE `role` = :role OR `role` = :role"

// VALUES clause
clause(CLAUSE_VALUES)(['name' => 'John', 'email' => 'john@example.com']);
// → "VALUES (:name, :email)"
```

## API

### Clause Types

| Constant | Description | Example Output |
|----------|-------------|----------------|
| `CLAUSE_SELECT` | SELECT clause | `SELECT id, name` |
| `CLAUSE_WHERE` | WHERE clause | `WHERE id = :id` |
| `CLAUSE_ORDER_BY` | ORDER BY clause | `ORDER BY created_at DESC` |
| `CLAUSE_GROUP_BY` | GROUP BY clause | `GROUP BY category` |
| `CLAUSE_SET` | SET clause (UPDATE) | `SET name = :name` |
| `CLAUSE_VALUES` | VALUES clause (INSERT) | `VALUES (:name, :email)` |

### Operators

| Constant | Description | Example Output |
|----------|-------------|----------------|
| `OP_AND` | AND grouping | `(condition AND condition)` |
| `OP_OR` | OR grouping | `(condition OR condition)` |
| `OP_IN` | IN operator | `IN (:val1, :val2)` |

### Core Functions

**`clause(int $type, string $glue = ''): callable`**

Creates a closure that generates SQL clauses.

- `$type`: Bitwise combination of clause constants
- `$glue`: Operator for associative arrays (e.g., '=', '>', 'LIKE')

**`statement(...$args): array`**

Combines multiple clauses into a complete SQL statement.

Returns `[$sql, $bindings]` tuple.

## Design Philosophy

### Comparison to Common Approaches

| Aspect | Raw SQL | Clausure | Laravel/Eloquent | Doctrine DBAL |
|--------|---------|----------|------------------|---------------|
| **Paradigm** | Procedural | Functional | OOP Fluent | OOP Fluent |
| **Memory** | Lowest | Low | Medium | Medium |
| **Reusability** | Low | High | Medium | Medium |
| **SQL Control** | Highest | High | Medium | High |
| **Learning Curve** | Lowest | Medium | Low | Medium |

### Key Advantages

- **Compositional**: Build complex queries from simple, reusable functions
- **Polymorphic**: Handles raw SQL strings, arrays, and nested closures
- **Safe**: Automatic parameter binding prevents SQL injection
- **Minimal**: Close to SQL with minimal abstraction overhead
- **Flexible**: Bitwise constants allow precise behavior control

### Performance

- **No object instantiation overhead**
- **Lazy evaluation** - SQL generated only when called
- **Memory efficient** - closures maintain minimal state
- **Reusable** - same closure generates different SQL variants



## License

MIT

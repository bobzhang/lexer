# TOML Lexer

A robust lexer implementation for TOML (Tom's Obvious, Minimal Language) with precise position tracking and error reporting.

## Features

- **Position Tracking**: Accurate line and column tracking for detailed error reporting
- **Unicode Support**: Proper handling of multi-byte characters and surrogate pairs
- **String View Integration**: Efficient string processing using MoonBit's string views
- **Error Handling**: Detailed error messages with position information

## Types

### Position

The `Position` struct tracks location information in the input:

```moonbit
pub(all) struct Position {
  line : Int
  column : Int
} derive(Eq, Show, ToJson)
```

### Lexer

The main lexer struct maintains state and position (defined in the lexer.mbt file):

- `input: String` - The input text being lexed
- `position: Int` - Current byte position in input  
- `line: Int` - Current line number (1-based)
- `column: Int` - Current column position (1-based)

## Basic Usage

### Creating a Lexer

```moonbit
test "creating a lexer" {
  let lexer = Lexer::new("key = value")
  inspect(lexer.get_position(), content="0")
  inspect(lexer.get_loc().line, content="1")
  inspect(lexer.get_loc().column, content="1")
}
```

### Position Tracking

```moonbit
test "position tracking example" {
  let lexer = Lexer::new("hello\nworld")
  inspect(lexer.get_loc().line, content="1")
  inspect(lexer.get_loc().column, content="1")

  // Advance through characters
  lexer.advance() // h
  inspect(lexer.get_loc().column, content="2")

  // Handle newlines explicitly
  lexer.advance() // move past \n
  lexer.new_line() // update line tracking
  inspect(lexer.get_loc().line, content="2")
  inspect(lexer.get_loc().column, content="1")
}
```

## Core Methods

### Character Access

- **`peek()`**: Get current character without advancing
- **`advance()`**: Move to next character with position tracking
- **`peek_charcode()`**: Get character code at current position

```moonbit
test "character access example" {
  let lexer = Lexer::new("Hello")
  inspect(lexer.peek(), content="Some('H')")
  lexer.advance()
  inspect(lexer.peek(), content="Some('e')")
}
```

### String Views

The lexer integrates with MoonBit's string views for efficient processing:

```moonbit
test "string views example" {
  let lexer = Lexer::new("😈x中world!")
  match lexer.view() {
    [.."😈x", .. rest] => lexer.update_view(rest)
    _ => ()
  }
  inspect(lexer.peek(), content="Some('中')")
}
```

### Whitespace Handling

```moonbit
test "whitespace handling example" {
  let lexer = Lexer::new("  \t\rHello, world!")
  lexer.skip_whitespace()
  inspect(lexer.peek(), content="Some('H')")
}
```

### Expectations and Error Handling

The lexer provides methods to expect specific characters or strings:

```moonbit
test "expectations example" {
  let lexer = Lexer::new("=true")
  // Expect a specific character
  lexer.expect_char('=', msg="Expected equals sign")
  
  // Expect a string
  lexer.expect_string("true", msg="Expected boolean value")
  
  // Error messages include precise position information
  let error_msg = lexer.error("Unexpected character")
  inspect(error_msg, content="Unexpected character at line 1, column 6")
}
```

## Advanced Features

### Unicode Support

The lexer properly handles Unicode characters including surrogate pairs:

```moonbit
test "unicode support example" {
  let lexer = Lexer::new("😈x")
  inspect(lexer.get_position(), content="0")
  lexer.advance() // Correctly advances past the 2-byte emoji
  inspect(lexer.get_position(), content="2") // 2 bytes for emoji
  inspect(lexer.peek(), content="Some('x')")
}
```

### Position Management

Get current position information:

```moonbit
test "position management example" {
  let lexer = Lexer::new("test")
  let pos = lexer.get_loc()           // Get Position struct
  let offset = lexer.get_position()   // Get byte offset
  inspect(pos.line, content="1")
  inspect(pos.column, content="1")
  inspect(offset, content="0")
}
```

## Example: Basic TOML Parsing

```moonbit
test "basic TOML parsing example" {
  let lexer = Lexer::new("key = \"value\"")

  // Skip initial whitespace
  lexer.skip_whitespace()

  // Parse key name
  let mut key = ""
  while lexer.peek() is Some(ch) && ch != ' ' {
    key = key + Char::to_string(ch)
    lexer.advance()
  }

  // Skip whitespace and expect equals
  lexer.skip_whitespace()
  lexer.expect_char('=')
  lexer.skip_whitespace()

  // Expect quoted string
  lexer.expect_char('"')
  let mut value = ""
  while lexer.peek() is Some(ch) && ch != '"' {
    value = value + Char::to_string(ch)
    lexer.advance()
  }
  lexer.expect_char('"')

  inspect(key, content="key")
  inspect(value, content="value")
}
```

## Package Information

- **Package**: `bob/toml/lexer`
- **Dependencies**: `moonbitlang/core/string`

This lexer is designed specifically for TOML parsing but can be adapted for other text processing tasks requiring precise position tracking and error reporting.

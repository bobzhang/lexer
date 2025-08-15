# bob/lexer

A generic lexer library for building handwritten lexers in MoonBit. This library provides essential lexing utilities including position tracking, character advancement, whitespace handling, and error reporting.

## Features

- **Position Tracking**: Accurate line and column tracking for better error reporting
- **Unicode Support**: Proper handling of surrogate pairs and multi-byte characters
- **Flexible Character Handling**: Peek, advance, and expect specific characters or strings
- **Whitespace Management**: Skip whitespace while preserving significant newlines
- **String View Integration**: Efficient string processing using MoonBit's string views
- **Error Reporting**: Detailed error messages with position information

## Installation

```bash
moon add bob/lexer
```

## Quick Start

```moonbit
let lexer = Lexer::new("key = value")

// Peek at current character without advancing
match lexer.peek() {
  Some('k') => println("Found 'k'")
  _ => ()
}

// Advance through characters
lexer.advance() // moves to 'e'
lexer.advance() // moves to 'y'

// Skip whitespace
lexer.skip_whitespace() // skips spaces, tabs, carriage returns

// Expect specific characters or strings
lexer.expect_char('=') |> ignore // advances if '=' is found, otherwise raises error
lexer.skip_whitespace()
lexer.expect_string("value") |> ignore // expects and consumes "value"

// Get current position for error reporting
let pos = lexer.get_loc() // returns Position { line: 1, column: 12 }
```

## API Reference

### Core Types

#### `Lexer`
The main lexer structure that maintains input state and position tracking.

#### `Position`
```moonbit
pub struct Position {
  line : Int
  column : Int
}
```
Represents a position in the input with line and column information.

### Methods

#### Creation
- `Lexer::new(input : String) -> Lexer` - Create a new lexer with the given input

#### Character Operations
- `peek() -> Char?` - Get current character without advancing
- `advance() -> Unit` - Advance to next character (handles Unicode properly)
- `expect_char(ch : Char, msg? : String) -> Unit raise` - Expect and consume a specific character

#### String Operations
- `expect_string(str : String, msg? : String) -> Unit raise` - Expect and consume a specific string
- `view() -> @string.View` - Get a view of the remaining input
- `update_view(view : @string.View) -> Unit` - Update position based on a new view

#### Whitespace Handling
- `skip_whitespace() -> Unit` - Skip spaces, tabs, and carriage returns (not newlines)
- `skip_single_newline() -> Unit` - Skip a single newline and update line tracking

#### Position Tracking
- `get_loc() -> Position` - Get current line and column position
- `get_position() -> Int` - Get current byte position in input
- `new_line() -> Unit` - Explicitly advance to new line (updates line/column tracking)

#### Error Handling
- `error(msg : String) -> String` - Create detailed error message with position info

## Advanced Usage

### Working with String Views

```moonbit
let lexer = Lexer::new("😈x中world!")
match lexer.view() {
  [.."😈x", .. rest] => lexer.update_view(rest) // skip emoji and 'x'
  _ => ()
}
// lexer is now positioned at '中'
```

### Position-Aware Error Handling

```moonbit
fn parse_identifier(lexer : Lexer) -> String raise {
  let start_pos = lexer.get_loc()
  // ... parsing logic ...
  if error_condition {
    let msg = "Invalid identifier at line " + start_pos.line.to_string()
    fail(lexer.error(msg))
  }
  // ...
}
```

### Custom Lexer Implementation

```moonbit
fn tokenize_simple(input : String) -> Array[Token] raise {
  let lexer = Lexer::new(input)
  let tokens = []
  
  loop {
    lexer.skip_whitespace()
    match lexer.peek() {
      None => break
      Some('=') => {
        lexer.advance()
        tokens.push(Token::Equals)
      }
      Some('"') => {
        tokens.push(parse_string(lexer))
      }
      Some(c) if c.is_ascii_alphabetic() => {
        tokens.push(parse_identifier(lexer))
      }
      Some(c) => fail(lexer.error("Unexpected character: " + c.to_string()))
    }
  }
  
  tokens
}
```

## License

Apache-2.0
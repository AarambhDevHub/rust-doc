+++
title = "Building a Simple CLI Calculator in Rust"
description = "Learn how to combine structs and enums to build a command-line calculator in Rust, handle errors with Result, and organize your code for maintainability."
date = 2025-08-13
draft = false
weight = 1
+++

# Building a CLI Calculator in Rust: Practice Project Documentation for Beginners

Understanding how to build a CLI calculator with structs, enums, and proper error handling is like learning to **create a sophisticated ordering system for a vegetarian restaurant** - you need to organize different types of data (menu items, orders, calculations), handle various scenarios (valid orders, errors, special requests), and create a smooth user experience. Just as a professional restaurant has organized stations, clear procedures, and excellent error handling, a well-built Rust CLI calculator combines structs, enums, and Result types into an elegant, reliable system.

## The Vegetarian Restaurant Ordering System Analogy 🍽️

### Imagine You're Building a Digital Ordering System

**Traditional Approach (Messy Code):**
```rust
// ❌ Everything mixed together - hard to maintain
fn calculate(input: String) -> String {
    // Parse numbers, handle operators, format output...
    // All mixed together in one giant function!
}
```

**Professional Approach (Organized System):**
```rust
// ✅ Well-organized like a professional restaurant
struct Calculator {
    operations: Vec,     // 📋 Order history
    current_result: Option,    // 💰 Running total
}

enum MathOperation {
    Add, Subtract, Multiply, Divide, // 🍽️ Menu options
}

enum CalcError {
    InvalidInput(String),            // ❌ Order problems
    DivisionByZero,                  // 🚫 Kitchen issues
    ParseError(String),              // 📝 Communication errors
}
```

**Why This Organization Works:**
- 🎯 **Clear responsibilities** - Each component has one job
- 🔧 **Easy maintenance** - Fix issues in specific areas
- 📈 **Extensible** - Add new features without breaking existing code
- 🛡️ **Robust error handling** - Handle problems gracefully
- 👥 **Team-friendly** - Other developers can understand the structure

## Project Overview and Setup

### Project Goals

We'll build a calculator that can:
- Perform basic arithmetic operations (+, -, *, /)
- Handle command-line arguments elegantly
- Provide clear error messages
- Maintain operation history
- Support interactive and batch modes

### Setting Up the Project Structure

```bash
# Create new Rust project
cargo new vegetarian_calculator
cd vegetarian_calculator

# Project structure we'll create:
# src/
# ├── main.rs           # Entry point
# ├── calculator.rs     # Main calculator logic
# ├── operations.rs     # Mathematical operations
# ├── errors.rs         # Error handling
# ├── parser.rs         # Input parsing
# └── lib.rs           # Library organization
```

Let's start building our professional calculator system:

### src/lib.rs - Library Organization

```rust
//! # Vegetarian Calculator
//!
//! A professional CLI calculator built like a well-organized vegetarian restaurant.
//! Each module handles specific responsibilities, just like kitchen stations.

pub mod calculator;
pub mod operations;
pub mod errors;
pub mod parser;

// Re-export main types for easy access
pub use calculator::Calculator;
pub use operations::{MathOperation, Operation};
pub use errors::{CalcError, CalcResult};
pub use parser::InputParser;

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_library_integration() {
        let mut calc = Calculator::new();
        let result = calc.calculate(2.0, MathOperation::Add, 3.0);
        assert!(result.is_ok());
        assert_eq!(result.unwrap(), 5.0);
    }
}
```

### src/errors.rs - Professional Error Handling

```rust
//! Error handling for the calculator
//!
//! Like a restaurant's incident reporting system - we need to know
//! exactly what went wrong and how to handle it gracefully.

use std::fmt;

/// Our Result type alias - like having a standard incident report format
pub type CalcResult = Result;

/// All the ways our calculator can encounter problems
/// Like different types of restaurant issues that need different responses
#[derive(Debug, Clone, PartialEq)]
pub enum CalcError {
    /// Input couldn't be parsed - like illegible handwriting on an order
    InvalidInput {
        input: String,
        reason: String,
    },

    /// Mathematical impossibility - like dividing by zero
    DivisionByZero {
        dividend: f64,
    },

    /// Number parsing failed - like ordering "seven and a half" pizzas
    ParseError {
        input: String,
        position: Option,
    },

    /// Unknown operation - like ordering a dish that doesn't exist
    UnknownOperation {
        operator: String,
        suggestions: Vec,
    },

    /// Missing arguments - like ordering food without specifying what
    MissingArguments {
        expected: usize,
        found: usize,
    },

    /// Result overflow - number too large to handle
    Overflow {
        operation: String,
        values: (f64, f64),
    },
}

impl fmt::Display for CalcError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            CalcError::InvalidInput { input, reason } => {
                write!(f, "❌ Invalid input '{}': {}", input, reason)
            }
            CalcError::DivisionByZero { dividend } => {
                write!(f, "🚫 Cannot divide {} by zero - that's mathematically impossible!", dividend)
            }
            CalcError::ParseError { input, position } => {
                match position {
                    Some(pos) => write!(f, "📝 Parse error in '{}' at position {}", input, pos),
                    None => write!(f, "📝 Cannot parse '{}'", input),
                }
            }
            CalcError::UnknownOperation { operator, suggestions } => {
                write!(f, "❓ Unknown operation '{}'. Did you mean: {}?",
                       operator, suggestions.join(", "))
            }
            CalcError::MissingArguments { expected, found } => {
                write!(f, "📋 Expected {} arguments, but found {}", expected, found)
            }
            CalcError::Overflow { operation, values } => {
                write!(f, "📊 Overflow in {} operation with values {} and {}",
                       operation, values.0, values.1)
            }
        }
    }
}

impl std::error::Error for CalcError {}

impl CalcError {
    /// Provide helpful suggestions based on the error type
    /// Like a helpful restaurant manager offering solutions
    pub fn get_suggestion(&self) -> String {
        match self {
            CalcError::InvalidInput { .. } => {
                "💡 Try using format: 'calculator 5 + 3' or run with --help".to_string()
            }
            CalcError::DivisionByZero { .. } => {
                "💡 Division by zero is undefined. Try a different divisor.".to_string()
            }
            CalcError::ParseError { .. } => {
                "💡 Make sure all numbers are valid (e.g., 5, 3.14, -2.5)".to_string()
            }
            CalcError::UnknownOperation { suggestions, .. } => {
                if suggestions.is_empty() {
                    "💡 Supported operations: +, -, *, /".to_string()
                } else {
                    format!("💡 Try one of: {}", suggestions.join(", "))
                }
            }
            CalcError::MissingArguments { expected, .. } => {
                format!("💡 Please provide {} arguments (e.g., calculator 5 + 3)", expected)
            }
            CalcError::Overflow { .. } => {
                "💡 Try smaller numbers to avoid overflow".to_string()
            }
        }
    }

    /// Check if this is a recoverable error
    pub fn is_recoverable(&self) -> bool {
        match self {
            CalcError::DivisionByZero { .. } => false,
            CalcError::Overflow { .. } => false,
            _ => true,
        }
    }
}

/// Helper function to create parse errors with context
pub fn parse_error(input: &str, position: Option) -> CalcError {
    CalcError::ParseError {
        input: input.to_string(),
        position,
    }
}

/// Helper function to create unknown operation errors with suggestions
pub fn unknown_operation_error(operator: &str) -> CalcError {
    let suggestions = match operator.to_lowercase().as_str() {
        "plus" | "add" => vec!["+".to_string()],
        "minus" | "subtract" => vec!["-".to_string()],
        "times" | "multiply" | "x" => vec!["*".to_string()],
        "divide" | "divided" => vec!["/".to_string()],
        _ => vec!["+".to_string(), "-".to_string(), "*".to_string(), "/".to_string()],
    };

    CalcError::UnknownOperation {
        operator: operator.to_string(),
        suggestions,
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_error_display() {
        let error = CalcError::DivisionByZero { dividend: 5.0 };
        let display = format!("{}", error);
        assert!(display.contains("Cannot divide 5 by zero"));
    }

    #[test]
    fn test_error_suggestions() {
        let error = CalcError::InvalidInput {
            input: "bad input".to_string(),
            reason: "testing".to_string(),
        };
        let suggestion = error.get_suggestion();
        assert!(suggestion.contains("help"));
    }

    #[test]
    fn test_unknown_operation_suggestions() {
        let error = unknown_operation_error("plus");
        if let CalcError::UnknownOperation { suggestions, .. } = error {
            assert_eq!(suggestions, vec!["+"]);
        } else {
            panic!("Expected UnknownOperation error");
        }
    }
}
```

### src/operations.rs - Mathematical Operations

```rust
//! Mathematical operations for the calculator
//!
//! Like the different cooking methods in our vegetarian kitchen -
//! each operation has its own technique and special considerations.

use crate::errors::{CalcError, CalcResult};
use std::fmt;

/// Supported mathematical operations
/// Like the main cooking methods in our kitchen
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum MathOperation {
    Add,        // 🍲 Combining ingredients
    Subtract,   // 🥄 Removing portions
    Multiply,   // 📈 Scaling recipes
    Divide,     // ✂️ Splitting portions
}

impl MathOperation {
    /// Parse operation from string
    /// Like interpreting customer orders
    pub fn from_str(s: &str) -> CalcResult {
        match s.trim() {
            "+" | "add" | "plus" => Ok(MathOperation::Add),
            "-" | "subtract" | "minus" => Ok(MathOperation::Subtract),
            "*" | "multiply" | "times" | "x" => Ok(MathOperation::Multiply),
            "/" | "divide" | "÷" => Ok(MathOperation::Divide),
            _ => Err(crate::errors::unknown_operation_error(s)),
        }
    }

    /// Get the symbol representation
    pub fn symbol(&self) -> &'static str {
        match self {
            MathOperation::Add => "+",
            MathOperation::Subtract => "-",
            MathOperation::Multiply => "*",
            MathOperation::Divide => "/",
        }
    }

    /// Get human-readable name
    pub fn name(&self) -> &'static str {
        match self {
            MathOperation::Add => "addition",
            MathOperation::Subtract => "subtraction",
            MathOperation::Multiply => "multiplication",
            MathOperation::Divide => "division",
        }
    }

    /// Check if operation might cause overflow
    pub fn might_overflow(&self, a: f64, b: f64) -> bool {
        match self {
            MathOperation::Multiply => {
                (a.abs() > 1e100 && b.abs() > 1.0) ||
                (b.abs() > 1e100 && a.abs() > 1.0)
            }
            MathOperation::Divide => b.abs()  false,
        }
    }
}

impl fmt::Display for MathOperation {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.symbol())
    }
}

/// Represents a complete operation with operands
/// Like a complete order with all details
#[derive(Debug, Clone, PartialEq)]
pub struct Operation {
    pub left_operand: f64,
    pub operation: MathOperation,
    pub right_operand: f64,
    pub result: f64,
    pub timestamp: std::time::SystemTime,
}

impl Operation {
    /// Create a new operation
    /// Like recording a new order
    pub fn new(left: f64, op: MathOperation, right: f64, result: f64) -> Self {
        Operation {
            left_operand: left,
            operation: op,
            right_operand: right,
            result,
            timestamp: std::time::SystemTime::now(),
        }
    }

    /// Execute the mathematical operation
    /// Like actually cooking the ordered dish
    pub fn execute(left: f64, operation: MathOperation, right: f64) -> CalcResult {
        // Check for potential overflow before calculating
        if operation.might_overflow(left, right) {
            return Err(CalcError::Overflow {
                operation: operation.name().to_string(),
                values: (left, right),
            });
        }

        let result = match operation {
            MathOperation::Add => {
                let result = left + right;
                if result.is_infinite() {
                    return Err(CalcError::Overflow {
                        operation: "addition".to_string(),
                        values: (left, right),
                    });
                }
                result
            }
            MathOperation::Subtract => {
                let result = left - right;
                if result.is_infinite() {
                    return Err(CalcError::Overflow {
                        operation: "subtraction".to_string(),
                        values: (left, right),
                    });
                }
                result
            }
            MathOperation::Multiply => {
                let result = left * right;
                if result.is_infinite() {
                    return Err(CalcError::Overflow {
                        operation: "multiplication".to_string(),
                        values: (left, right),
                    });
                }
                result
            }
            MathOperation::Divide => {
                if right == 0.0 {
                    return Err(CalcError::DivisionByZero { dividend: left });
                }
                let result = left / right;
                if result.is_infinite() {
                    return Err(CalcError::Overflow {
                        operation: "division".to_string(),
                        values: (left, right),
                    });
                }
                result
            }
        };

        Ok(result)
    }

    /// Format the operation as a string equation
    /// Like printing a receipt
    pub fn format_equation(&self) -> String {
        format!("{} {} {} = {}",
                self.left_operand,
                self.operation.symbol(),
                self.right_operand,
                self.result)
    }

    /// Get operation details for history
    pub fn get_details(&self) -> String {
        let elapsed = self.timestamp.elapsed()
            .map(|d| format!("{:.2}s ago", d.as_secs_f64()))
            .unwrap_or_else(|_| "recently".to_string());

        format!("{} ({})", self.format_equation(), elapsed)
    }
}

impl fmt::Display for Operation {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.format_equation())
    }
}

/// Collection of utility functions for operations
pub struct OperationUtils;

impl OperationUtils {
    /// Round result to reasonable precision
    /// Like plating food with proper presentation
    pub fn round_result(value: f64, precision: u32) -> f64 {
        let multiplier = 10_f64.powi(precision as i32);
        (value * multiplier).round() / multiplier
    }

    /// Check if result is a whole number
    pub fn is_whole_number(value: f64) -> bool {
        (value.fract()).abs()  String {
        if Self::is_whole_number(value) {
            format!("{:.0}", value)
        } else if value.abs()  1e6 {
            format!("{:.2e}", value)
        } else {
            format!("{:.6}", value).trim_end_matches('0').trim_end_matches('.').to_string()
        }
    }

    /// Get all supported operations
    pub fn all_operations() -> Vec {
        vec![
            MathOperation::Add,
            MathOperation::Subtract,
            MathOperation::Multiply,
            MathOperation::Divide,
        ]
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_operation_parsing() {
        assert_eq!(MathOperation::from_str("+").unwrap(), MathOperation::Add);
        assert_eq!(MathOperation::from_str("add").unwrap(), MathOperation::Add);
        assert_eq!(MathOperation::from_str("*").unwrap(), MathOperation::Multiply);
        assert!(MathOperation::from_str("invalid").is_err());
    }

    #[test]
    fn test_basic_operations() {
        assert_eq!(Operation::execute(5.0, MathOperation::Add, 3.0).unwrap(), 8.0);
        assert_eq!(Operation::execute(10.0, MathOperation::Subtract, 4.0).unwrap(), 6.0);
        assert_eq!(Operation::execute(6.0, MathOperation::Multiply, 7.0).unwrap(), 42.0);
        assert_eq!(Operation::execute(15.0, MathOperation::Divide, 3.0).unwrap(), 5.0);
    }

    #[test]
    fn test_division_by_zero() {
        let result = Operation::execute(5.0, MathOperation::Divide, 0.0);
        assert!(result.is_err());
        match result.err().unwrap() {
            CalcError::DivisionByZero { dividend } => assert_eq!(dividend, 5.0),
            _ => panic!("Expected DivisionByZero error"),
        }
    }

    #[test]
    fn test_operation_formatting() {
        let op = Operation::new(5.0, MathOperation::Add, 3.0, 8.0);
        assert_eq!(op.format_equation(), "5 + 3 = 8");
    }

    #[test]
    fn test_number_formatting() {
        assert_eq!(OperationUtils::format_number(5.0), "5");
        assert_eq!(OperationUtils::format_number(3.14159), "3.14159");
        assert_eq!(OperationUtils::format_number(0.00123), "1.23e-3");
    }
}
```

### src/parser.rs - Input Parsing System

```rust
//! Input parsing for the calculator
//!
//! Like the order-taking system in our restaurant - we need to understand
//! what the customer wants, even if they don't express it perfectly.

use crate::errors::{CalcError, CalcResult};
use crate::operations::MathOperation;

/// Handles parsing user input into calculator operations
/// Like a multilingual waiter who understands various ways customers order
pub struct InputParser;

impl InputParser {
    /// Parse command line arguments into a calculation
    /// Like interpreting a customer's order from their words
    pub fn parse_args(args: &[String]) -> CalcResult {
        if args.len()  3 {
            // Too many arguments - might be multiple operations
            return Err(CalcError::InvalidInput {
                input: args.join(" "),
                reason: "Multiple operations not supported in simple mode. Use interactive mode.".to_string(),
            });
        }

        // Parse first number
        let left = Self::parse_number(&args[0])
            .map_err(|_| CalcError::ParseError {
                input: args[0].clone(),
                position: Some(0),
            })?;

        // Parse operation
        let operation = MathOperation::from_str(&args[1])?;

        // Parse second number
        let right = Self::parse_number(&args[2])
            .map_err(|_| CalcError::ParseError {
                input: args[2].clone(),
                position: Some(2),
            })?;

        Ok((left, operation, right))
    }

    /// Parse a string expression like "5 + 3"
    /// Like understanding written orders
    pub fn parse_expression(expression: &str) -> CalcResult {
        let trimmed = expression.trim();

        if trimmed.is_empty() {
            return Err(CalcError::InvalidInput {
                input: expression.to_string(),
                reason: "Empty expression".to_string(),
            });
        }

        // Try to find operator position
        let operators = ["+", "-", "*", "/"];
        let mut operator_pos = None;
        let mut found_operator = "";

        // Find the last occurrence of an operator (to handle negative numbers)
        for (i, ch) in trimmed.char_indices() {
            let ch_str = ch.to_string();
            if operators.contains(&ch_str.as_str()) {
                // Check if this is actually an operator, not a negative sign
                if ch == '-' && i == 0 {
                    continue; // This is a negative sign at the start
                }
                if ch == '-' && i > 0 {
                    let prev_char = trimmed.chars().nth(i - 1);
                    if let Some(prev) = prev_char {
                        if prev.is_ascii_whitespace() || operators.contains(&prev.to_string().as_str()) {
                            // This might be a negative number, continue searching
                            continue;
                        }
                    }
                }
                operator_pos = Some(i);
                found_operator = &ch_str;
            }
        }

        let (operator_pos, operator_str) = operator_pos
            .map(|pos| (pos, found_operator))
            .ok_or_else(|| CalcError::InvalidInput {
                input: expression.to_string(),
                reason: "No operator found".to_string(),
            })?;

        // Split the expression
        let (left_str, right_with_op) = trimmed.split_at(operator_pos);
        let right_str = &right_with_op[1..]; // Remove the operator character

        // Parse the parts
        let left = Self::parse_number(left_str.trim())
            .map_err(|_| CalcError::ParseError {
                input: left_str.to_string(),
                position: Some(0),
            })?;

        let operation = MathOperation::from_str(operator_str)?;

        let right = Self::parse_number(right_str.trim())
            .map_err(|_| CalcError::ParseError {
                input: right_str.to_string(),
                position: Some(operator_pos + 1),
            })?;

        Ok((left, operation, right))
    }

    /// Parse a number from string, handling various formats
    /// Like understanding when customers say "five point five" or "5.5"
    fn parse_number(s: &str) -> Result {
        let cleaned = s.trim()
            .replace(" ", "")          // Remove spaces
            .replace(",", "");         // Remove commas (for large numbers)

        // Handle some common variations
        let normalized = match cleaned.to_lowercase().as_str() {
            "pi" | "π" => std::f64::consts::PI.to_string(),
            "e" => std::f64::consts::E.to_string(),
            _ => cleaned,
        };

        normalized.parse()
    }

    /// Parse interactive input (more forgiving)
    /// Like understanding casual conversation with regular customers
    pub fn parse_interactive(input: &str) -> CalcResult {
        let input = input.trim();

        // Handle special cases
        if input.is_empty() {
            return Err(CalcError::InvalidInput {
                input: input.to_string(),
                reason: "Please enter a calculation (e.g., '5 + 3')".to_string(),
            });
        }

        // Check for help requests
        if input.to_lowercase().contains("help") || input == "?" {
            return Err(CalcError::InvalidInput {
                input: input.to_string(),
                reason: "Available operations: +, -, *, /. Example: 5 + 3".to_string(),
            });
        }

        // Try expression parsing first
        Self::parse_expression(input)
    }

    /// Validate parsed input for common issues
    /// Like double-checking orders before sending to the kitchen
    pub fn validate_input(left: f64, _operation: MathOperation, right: f64) -> CalcResult {
        // Check for NaN or infinite values
        if left.is_nan() || left.is_infinite() {
            return Err(CalcError::InvalidInput {
                input: left.to_string(),
                reason: "First number is not valid".to_string(),
            });
        }

        if right.is_nan() || right.is_infinite() {
            return Err(CalcError::InvalidInput {
                input: right.to_string(),
                reason: "Second number is not valid".to_string(),
            });
        }

        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_args() {
        let args = vec!["5".to_string(), "+".to_string(), "3".to_string()];
        let result = InputParser::parse_args(&args).unwrap();
        assert_eq!(result, (5.0, MathOperation::Add, 3.0));
    }

    #[test]
    fn test_parse_expression() {
        let result = InputParser::parse_expression("10 - 4").unwrap();
        assert_eq!(result, (10.0, MathOperation::Subtract, 4.0));

        let result = InputParser::parse_expression("3.14 * 2").unwrap();
        assert_eq!(result, (3.14, MathOperation::Multiply, 2.0));
    }

    #[test]
    fn test_parse_negative_numbers() {
        let result = InputParser::parse_expression("-5 + 3").unwrap();
        assert_eq!(result, (-5.0, MathOperation::Add, 3.0));

        let result = InputParser::parse_expression("10 + -3").unwrap();
        assert_eq!(result, (10.0, MathOperation::Add, -3.0));
    }

    #[test]
    fn test_parse_special_numbers() {
        let result = InputParser::parse_expression("pi * 2").unwrap();
        assert!((result.0 - std::f64::consts::PI).abs() ,
    max_history: usize,
    current_result: Option,
    precision: u32,
}

impl Calculator {
    /// Create a new calculator
    /// Like opening the restaurant for the day
    pub fn new() -> Self {
        Calculator {
            history: VecDeque::new(),
            max_history: 100,
            current_result: None,
            precision: 6,
        }
    }

    /// Create calculator with custom settings
    /// Like setting up a restaurant with specific preferences
    pub fn with_config(max_history: usize, precision: u32) -> Self {
        Calculator {
            history: VecDeque::new(),
            max_history,
            current_result: None,
            precision,
        }
    }

    /// Perform a calculation
    /// Like processing a complete order
    pub fn calculate(&mut self, left: f64, operation: MathOperation, right: f64) -> CalcResult {
        // Validate input
        InputParser::validate_input(left, operation, right)?;

        // Execute the operation
        let result = Operation::execute(left, operation, right)?;

        // Round to specified precision
        let rounded_result = OperationUtils::round_result(result, self.precision);

        // Create operation record
        let op = Operation::new(left, operation, right, rounded_result);

        // Add to history
        self.add_to_history(op);

        // Update current result
        self.current_result = Some(rounded_result);

        Ok(rounded_result)
    }

    /// Calculate from string expression
    /// Like taking a verbal order
    pub fn calculate_from_string(&mut self, expression: &str) -> CalcResult {
        let (left, operation, right) = InputParser::parse_expression(expression)?;
        self.calculate(left, operation, right)
    }

    /// Calculate from command line arguments
    /// Like processing a written order
    pub fn calculate_from_args(&mut self, args: &[String]) -> CalcResult {
        let (left, operation, right) = InputParser::parse_args(args)?;
        self.calculate(left, operation, right)
    }

    /// Add operation to history
    /// Like recording completed orders
    fn add_to_history(&mut self, operation: Operation) {
        if self.history.len() >= self.max_history {
            self.history.pop_front(); // Remove oldest
        }
        self.history.push_back(operation);
    }

    /// Get calculation history
    /// Like reviewing the day's orders
    pub fn get_history(&self) -> &VecDeque {
        &self.history
    }

    /// Get last result
    /// Like checking the last completed order
    pub fn get_last_result(&self) -> Option {
        self.current_result
    }

    /// Get last operation
    /// Like reviewing the most recent order
    pub fn get_last_operation(&self) -> Option {
        self.history.back()
    }

    /// Clear history
    /// Like starting fresh for a new day
    pub fn clear_history(&mut self) {
        self.history.clear();
        self.current_result = None;
    }

    /// Get formatted history
    /// Like printing a summary report
    pub fn format_history(&self, limit: Option) -> String {
        if self.history.is_empty() {
            return "📋 No calculations yet".to_string();
        }

        let mut result = String::from("📋 Calculation History:\n");
        let items_to_show = limit.unwrap_or(self.history.len()).min(self.history.len());

        for (i, operation) in self.history.iter().rev().take(items_to_show).enumerate() {
            result.push_str(&format!("   {}. {}\n", i + 1, operation.get_details()));
        }

        if limit.is_some() && limit.unwrap()  CalculatorStats {
        let mut stats = CalculatorStats::new();

        for operation in &self.history {
            stats.total_operations += 1;

            match operation.operation {
                MathOperation::Add => stats.additions += 1,
                MathOperation::Subtract => stats.subtractions += 1,
                MathOperation::Multiply => stats.multiplications += 1,
                MathOperation::Divide => stats.divisions += 1,
            }

            if operation.result > stats.largest_result {
                stats.largest_result = operation.result;
            }

            if operation.result  u32 {
        self.precision
    }

    /// Format a result for display
    /// Like presenting the final dish
    pub fn format_result(&self, value: f64) -> String {
        OperationUtils::format_number(value)
    }
}

impl Default for Calculator {
    fn default() -> Self {
        Self::new()
    }
}

/// Statistics about calculator usage
/// Like restaurant performance metrics
#[derive(Debug, Clone, Default)]
pub struct CalculatorStats {
    pub total_operations: usize,
    pub additions: usize,
    pub subtractions: usize,
    pub multiplications: usize,
    pub divisions: usize,
    pub largest_result: f64,
    pub smallest_result: f64,
}

impl CalculatorStats {
    fn new() -> Self {
        CalculatorStats {
            total_operations: 0,
            additions: 0,
            subtractions: 0,
            multiplications: 0,
            divisions: 0,
            largest_result: f64::NEG_INFINITY,
            smallest_result: f64::INFINITY,
        }
    }

    /// Format statistics for display
    /// Like creating a performance report
    pub fn format(&self) -> String {
        if self.total_operations == 0 {
            return "📊 No operations performed yet".to_string();
        }

        format!(
            "📊 Calculator Statistics:\n\
             Total Operations: {}\n\
             - Additions: {} ({:.1}%)\n\
             - Subtractions: {} ({:.1}%)\n\
             - Multiplications: {} ({:.1}%)\n\
             - Divisions: {} ({:.1}%)\n\
             Largest Result: {}\n\
             Smallest Result: {}",
            self.total_operations,
            self.additions, self.percentage(self.additions),
            self.subtractions, self.percentage(self.subtractions),
            self.multiplications, self.percentage(self.multiplications),
            self.divisions, self.percentage(self.divisions),
            OperationUtils::format_number(self.largest_result),
            OperationUtils::format_number(self.smallest_result)
        )
    }

    fn percentage(&self, count: usize) -> f64 {
        if self.total_operations == 0 {
            0.0
        } else {
            (count as f64 / self.total_operations as f64) * 100.0
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_calculation() {
        let mut calc = Calculator::new();
        let result = calc.calculate(5.0, MathOperation::Add, 3.0).unwrap();
        assert_eq!(result, 8.0);
        assert_eq!(calc.get_last_result(), Some(8.0));
    }

    #[test]
    fn test_calculation_from_string() {
        let mut calc = Calculator::new();
        let result = calc.calculate_from_string("10 * 4").unwrap();
        assert_eq!(result, 40.0);
    }

    #[test]
    fn test_history_management() {
        let mut calc = Calculator::with_config(2, 6); // Max 2 items in history

        calc.calculate(1.0, MathOperation::Add, 1.0).unwrap();
        calc.calculate(2.0, MathOperation::Add, 2.0).unwrap();
        calc.calculate(3.0, MathOperation::Add, 3.0).unwrap(); // Should push out first

        assert_eq!(calc.get_history().len(), 2);
        assert_eq!(calc.get_history().back().unwrap().result, 6.0);
    }

    #[test]
    fn test_precision_setting() {
        let mut calc = Calculator::new();
        calc.set_precision(2);

        let result = calc.calculate(1.0, MathOperation::Divide, 3.0).unwrap();
        // Should be rounded to 2 decimal places
        assert!((result - 0.33).abs()  = env::args().collect();

    // Remove program name from arguments
    let calc_args = &args[1..];

    match calc_args.len() {
        0 => run_interactive_mode(),
        1 if calc_args[0] == "--help" || calc_args[0] == "-h" => show_help(),
        1 if calc_args[0] == "--version" || calc_args[0] == "-v" => show_version(),
        3 => run_single_calculation(calc_args),
        _ => {
            eprintln!("❌ Invalid arguments. Use --help for usage information.");
            std::process::exit(1);
        }
    }
}

/// Run the calculator in interactive mode
/// Like having a conversation with a helpful waiter
fn run_interactive_mode() {
    println!("🧮 Welcome to the Vegetarian Calculator!");
    println!("Like a friendly restaurant, we're here to serve your mathematical needs.");
    println!("Enter calculations like '5 + 3' or type 'help' for assistance.");
    println!("Type 'quit' or 'exit' to leave.\n");

    let mut calculator = Calculator::new();
    let mut input = String::new();

    loop {
        print!("🍽️ calc> ");
        io::stdout().flush().unwrap();

        input.clear();
        match io::stdin().read_line(&mut input) {
            Ok(_) => {
                let trimmed = input.trim();

                // Handle special commands
                match trimmed.to_lowercase().as_str() {
                    "quit" | "exit" | "q" => {
                        println!("👋 Thanks for dining with us! Come back soon!");
                        break;
                    }
                    "help" | "h" | "?" => {
                        show_interactive_help();
                        continue;
                    }
                    "history" => {
                        println!("{}", calculator.format_history(Some(10)));
                        continue;
                    }
                    "stats" => {
                        println!("{}", calculator.get_statistics().format());
                        continue;
                    }
                    "clear" => {
                        calculator.clear_history();
                        println!("🧹 History cleared!");
                        continue;
                    }
                    "" => continue, // Empty input
                    _ => {}
                }

                // Process calculation
                match calculator.calculate_from_string(trimmed) {
                    Ok(result) => {
                        println!("✅ {} = {}",
                                get_last_equation(&calculator),
                                calculator.format_result(result));
                    }
                    Err(error) => {
                        println!("{}", error);
                        println!("{}", error.get_suggestion());

                        if !error.is_recoverable() {
                            println!("⚠️ This error requires attention before continuing.");
                        }
                    }
                }
            }
            Err(error) => {
                eprintln!("❌ Error reading input: {}", error);
                break;
            }
        }
    }
}

/// Run a single calculation from command line arguments
/// Like processing a takeout order
fn run_single_calculation(args: &[String]) {
    let mut calculator = Calculator::new();

    match calculator.calculate_from_args(args) {
        Ok(result) => {
            let equation = format!("{} {} {} = {}",
                                 args[0], args[1], args[2],
                                 calculator.format_result(result));
            println!("{}", equation);
            std::process::exit(0);
        }
        Err(error) => {
            eprintln!("{}", error);
            eprintln!("{}", error.get_suggestion());
            std::process::exit(1);
        }
    }
}

/// Get the last equation from calculator history
/// Like showing the customer what they ordered
fn get_last_equation(calculator: &Calculator) -> String {
    calculator.get_last_operation()
        .map(|op| format!("{} {} {}", op.left_operand, op.operation, op.right_operand))
        .unwrap_or_else(|| "calculation".to_string())
}

/// Show help information
/// Like providing a menu and explaining how to order
fn show_help() {
    println!("🧮 Vegetarian Calculator - Help");
    println!("{:=      # Single calculation");
    println!("  calculator                                 # Interactive mode");
    println!();
    println!("🍽️ EXAMPLES:");
    println!("  calculator 5 + 3          # Outputs: 5 + 3 = 8");
    println!("  calculator 15.5 / 2.5     # Outputs: 15.5 / 2.5 = 6.2");
    println!("  calculator -10 * 3        # Outputs: -10 * 3 = -30");
    println!();
    println!("🔧 OPERATORS:");
    println!("  +    Addition       (5 + 3 = 8)");
    println!("  -    Subtraction    (10 - 4 = 6)");
    println!("  *    Multiplication (6 * 7 = 42)");
    println!("  /    Division       (15 / 3 = 5)");
    println!();
    println!("🎮 INTERACTIVE MODE COMMANDS:");
    println!("  help      Show this help");
    println!("  history   Show calculation history");
    println!("  stats     Show usage statistics");
    println!("  clear     Clear history");
    println!("  quit      Exit the calculator");
    println!();
    println!("📊 FEATURES:");
    println!("  • Supports decimal numbers (3.14, -2.5)");
    println!("  • Handles negative numbers (-5 + 3)");
    println!("  • Maintains calculation history");
    println!("  • Provides helpful error messages");
    println!("  • Special constants: pi, e");
    println!();
    println!("🚨 ERROR HANDLING:");
    println!("  • Division by zero prevention");
    println!("  • Overflow detection");
    println!("  • Invalid input correction suggestions");
}

/// Show interactive mode help
/// Like explaining the dining experience to seated customers
fn show_interactive_help() {
    println!("🎮 Interactive Mode Help");
    println!("{:-"]
description = "A friendly CLI calculator built with Rust - like having a personal mathematical chef!"
license = "MIT"
repository = "https://github.com/yourusername/vegetarian_calculator"
keywords = ["calculator", "cli", "math", "rust", "beginner"]
categories = ["command-line-utilities", "mathematics"]

[[bin]]
name = "calculator"
path = "src/main.rs"

[dependencies]
# No external dependencies - pure Rust power! 🦀

[dev-dependencies]
# For testing
assert_cmd = "2.0"
predicates = "3.0"

[features]
default = []
# Future features could go here
extended_math = []  # For advanced operations like sin, cos, log
scientific = []     # For scientific notation

[profile.release]
# Optimize for size and speed - like efficient kitchen operations
opt-level = 3
lto = true
codegen-units = 1
strip = true
```

## Building and Running the Calculator

### Compilation and Testing

```bash
# Build the project
cargo build

# Run tests
cargo test

# Run with optimizations
cargo build --release

# Install locally
cargo install --path .
```

### Usage Examples

```bash
# Single calculations
./target/release/calculator 5 + 3
# Output: 5 + 3 = 8

./target/release/calculator 15.5 / 2.5
# Output: 15.5 / 2.5 = 6.2

./target/release/calculator -10 "*" 3
# Output: -10 * 3 = -30

# Interactive mode
./target/release/calculator
# 🧮 Welcome to the Vegetarian Calculator!
# 🍽️ calc> 5 + 3
# ✅ 5 + 3 = 8
# 🍽️ calc> history
# 📋 Calculation History:
#    1. 5 + 3 = 8 (0.12s ago)

# Help
./target/release/calculator --help
```

## Advanced Features and Extensions

### Adding More Operations

```rust
// In src/operations.rs - extend the MathOperation enum
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum MathOperation {
    Add, Subtract, Multiply, Divide,
    // New operations
    Power,      // ^ or **
    Modulo,     // %
    SquareRoot, // sqrt
}

impl MathOperation {
    pub fn from_str(s: &str) -> CalcResult {
        match s.trim() {
            "+" | "add" | "plus" => Ok(MathOperation::Add),
            "-" | "subtract" | "minus" => Ok(MathOperation::Subtract),
            "*" | "multiply" | "times" | "x" => Ok(MathOperation::Multiply),
            "/" | "divide" | "÷" => Ok(MathOperation::Divide),
            "^" | "**" | "pow" | "power" => Ok(MathOperation::Power),
            "%" | "mod" | "modulo" => Ok(MathOperation::Modulo),
            "sqrt" | "√" => Ok(MathOperation::SquareRoot),
            _ => Err(crate::errors::unknown_operation_error(s)),
        }
    }
}

// Extend Operation::execute to handle new operations
impl Operation {
    pub fn execute(left: f64, operation: MathOperation, right: f64) -> CalcResult {
        let result = match operation {
            // ... existing operations ...
            MathOperation::Power => {
                if left == 0.0 && right  {
                if right == 0.0 {
                    return Err(CalcError::DivisionByZero { dividend: left });
                }
                left % right
            }
            MathOperation::SquareRoot => {
                if left  Self {
        CalculatorConfig {
            precision: 6,
            max_history: 100,
            default_mode: "interactive".to_string(),
            color_output: true,
            scientific_notation: false,
        }
    }
}

impl CalculatorConfig {
    pub fn load_from_file(path: &str) -> CalcResult {
        let content = std::fs::read_to_string(path)
            .map_err(|e| CalcError::InvalidInput {
                input: path.to_string(),
                reason: format!("Cannot read config file: {}", e),
            })?;

        toml::from_str(&content)
            .map_err(|e| CalcError::ParseError {
                input: content,
                position: None,
            })
    }

    pub fn save_to_file(&self, path: &str) -> CalcResult {
        let content = toml::to_string_pretty(self)
            .map_err(|e| CalcError::InvalidInput {
                input: "config".to_string(),
                reason: format!("Cannot serialize config: {}", e),
            })?;

        std::fs::write(path, content)
            .map_err(|e| CalcError::InvalidInput {
                input: path.to_string(),
                reason: format!("Cannot write config file: {}", e),
            })?;

        Ok(())
    }
}
```

## Best Practices Demonstrated

### 1. **Error Handling Excellence**
```rust
// ✅ Rich, specific error types
enum CalcError {
    InvalidInput { input: String, reason: String },
    DivisionByZero { dividend: f64 },
    // ... more specific errors
}

// ✅ Helpful error messages with suggestions
impl CalcError {
    pub fn get_suggestion(&self) -> String {
        match self {
            CalcError::InvalidInput { .. } => "💡 Try format: 'calculator 5 + 3'",
            // ... more helpful suggestions
        }
    }
}
```

### 2. **Clean Code Organization**
```rust
// ✅ Logical module structure
pub mod calculator;    // Main logic
pub mod operations;    // Math operations
pub mod errors;        // Error handling
pub mod parser;        // Input parsing

// ✅ Clear separation of concerns
// Each module has a single, clear responsibility
```

### 3. **Comprehensive Testing**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_operations() {
        // Test normal cases
        assert_eq!(Operation::execute(5.0, MathOperation::Add, 3.0).unwrap(), 8.0);
    }

    #[test]
    fn test_error_cases() {
        // Test error conditions
        let result = Operation::execute(5.0, MathOperation::Divide, 0.0);
        assert!(result.is_err());
    }

    #[test]
    fn test_edge_cases() {
        // Test boundary conditions
        // Test with very large numbers, very small numbers, etc.
    }
}
```

### 4. **User-Friendly Interface**
```rust
// ✅ Multiple interaction modes
fn main() {
    match args.len() {
        0 => run_interactive_mode(),      // Conversational
        3 => run_single_calculation(),    // Quick operations
        _ => show_help(),                 // Guidance
    }
}

// ✅ Helpful output formatting
println!("✅ {} = {}", equation, result);  // Clear success
println!("❌ {}", error);                  // Clear errors
```

## Summary and Key Takeaways

### **Mental Model: The Professional Vegetarian Restaurant**

Remember our restaurant system analogy:
- 🏗️ **Project structure** = Restaurant layout with organized stations
- 📋 **Structs** = Order tickets, inventory records, staff information
- 🎛️ **Enums** = Menu categories, cooking methods, order statuses
- 🛡️ **Result** = Quality control system that catches problems
- 👨🍳 **Calculator** = Head chef coordinating everything

### **Essential Architecture Principles**

1. **Separation of Concerns** - Each module handles one responsibility
2. **Rich Error Handling** - Specific errors with helpful suggestions
3. **Type Safety** - Use enums and structs to prevent invalid states
4. **User Experience** - Multiple interfaces (CLI args, interactive, help)
5. **Extensibility** - Easy to add new operations and features
6. **Testing** - Comprehensive test coverage for reliability

### **Real-World Benefits**

| **Feature** | **Benefit** | **Like Restaurant** |
|-------------|-------------|-------------------|
| **Modular design** | Easy maintenance | Organized kitchen stations |
| **Rich error types** | Clear problem identification | Specific incident reports |
| **Result** | Reliable error handling | Quality control system |
| **Interactive mode** | User-friendly experience | Attentive table service |
| **History tracking** | Operation audit trail | Order history records |
| **Comprehensive tests** | Reliable software | Food safety procedures |

### **Extension Opportunities**

**Beginner Extensions:**
- Add more mathematical operations (power, modulo, square root)
- Implement memory functions (store, recall, clear)
- Add expression validation and better error messages
- Create a configuration file system

**Intermediate Extensions:**
- Support for parentheses and order of operations
- Variable storage and retrieval
- Basic graphing capabilities
- Integration tests with file I/O

**Advanced Extensions:**
- Scientific calculator functions (sin, cos, log)
- Matrix operations
- Statistical functions
- Web interface using web frameworks

**This CLI calculator project demonstrates professional Rust development** - it's like building a complete restaurant operation with proper organization, quality control, customer service, and room for growth. Every component works together harmoniously, errors are handled gracefully, and the user experience is delightful.

The combination of structs, enums, and Result types creates a robust foundation that can handle real-world complexity while remaining maintainable and extensible - just like a well-run restaurant that can serve both simple orders and complex requests with equal excellence!

1. https://dev.to/khair_al_anam/rust-tutorial-4-lets-build-a-simple-calculator-part-1-59fe
2. https://www.youtube.com/watch?v=lZ2xWczv1lI
3. https://techwasti.com/building-a-small-calculator-in-rust
4. https://rust-cli.github.io/book/index.html
5. https://igordejanovic.net/rustemo/tutorials/calculator/calculator.html
6. https://dev.to/khair_al_anam/rust-tutorial-5-lets-build-a-simple-calculator-part-2-332o
7. https://www.freecodecamp.org/news/rust-in-replit/
8. https://www.linkedin.com/learning/using-rust-with-python/python-calc-cli

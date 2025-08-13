+++
title = "Creating Custom Error Types in Rust"
description = "Learn how to define and use custom error types in Rust using enums, structs, and the std::error::Error trait for more expressive error handling."
date = 2025-08-13
weight = 4
keywords = ["Rust", "custom error types", "error handling", "Result", "thiserror", "enum errors", "struct errors", "std::error::Error"]
+++


# Custom Error Types in Rust: Comprehensive Documentation for Beginners

Understanding custom error types in Rust is like learning to **design a comprehensive incident reporting system for your vegetarian restaurant** - instead of just saying "something went wrong," you need detailed categories that tell you exactly what type of problem occurred (kitchen issue, customer complaint, supply chain problem, staff shortage), where it happened, why it happened, and what specific actions need to be taken. Just as a professional restaurant manager needs different protocols for different problems, Rust's custom error types allow you to create sophisticated error handling systems that provide rich context, enable appropriate responses, and make debugging and maintenance much more effective.

## The Professional Restaurant Incident Management System Analogy 🏪

### Imagine You're Designing a Complete Error Classification System for Your Restaurant

**The Problem with Generic Errors:**

```rust
// ❌ Generic errors - like saying "something went wrong"
fn process_order_generic(order: &str) -> Result<String, String> {
    if order.is_empty() {
        return Err("Error".to_string()); // What kind of error?
    }

    if !order.contains("$") {
        return Err("Bad input".to_string()); // Why is it bad?
    }

    Ok("Order processed".to_string())
}
```

**The Power of Custom Error Types - Professional Incident Classification:**

```rust
// ✅ Custom error types - like having detailed incident categories
#[derive(Debug)]
enum RestaurantError {
    // Kitchen-related incidents
    Kitchen(KitchenError),

    // Customer service incidents
    CustomerService(CustomerServiceError),

    // Supply chain incidents
    SupplyChain(SupplyChainError),

    // Staff-related incidents
    Staff(StaffError),

    // System/equipment incidents
    System(SystemError),
}

#[derive(Debug)]
enum KitchenError {
    EquipmentFailure { equipment: String, error_code: u32 },
    InsufficientIngredients { item: String, needed: u32, available: u32 },
    FoodSafety { violation: String, severity: SafetySeverity },
    PreparationError { dish: String, step: String, details: String },
}

#[derive(Debug)]
enum CustomerServiceError {
    InvalidOrder { details: String, suggested_fix: String },
    PaymentIssue { amount: f64, payment_method: String, reason: String },
    SpecialRequestUnavailable { request: String, alternatives: Vec<String> },
}
```

**Why Custom Error Types Are Superior:**

- 🎯 **Precise problem identification** - Know exactly what went wrong
- 🔧 **Targeted solutions** - Different errors need different responses
- 📊 **Rich context** - Carry all relevant information for debugging
- 🏗️ **Structured hierarchy** - Organize errors by domain and severity
- 🛡️ **Type safety** - Compiler ensures you handle all error cases
- 📈 **Better monitoring** - Track and analyze specific error patterns


## Understanding Custom Error Types Fundamentals

### What are Custom Error Types?

Custom error types are user-defined types that represent specific error conditions in your application:

```rust
fn demonstrate_custom_error_basics() {
    println!("🏗️ Custom Error Types Fundamentals - Restaurant Management");
    println!("{:=<65}", "");

    // Custom error types provide structure and context to error handling
    println!("💡 Why Custom Error Types Matter:");
    println!("   • 🎯 Specific error identification");
    println!("   • 🔧 Appropriate error responses");
    println!("   • 📊 Rich debugging information");
    println!("   • 🏗️ Hierarchical error organization");
    println!("   • 🛡️ Compile-time error handling verification");

    // Example 1: Simple enum-based custom error
    #[derive(Debug)]
    enum OrderError {
        EmptyOrder,
        InvalidItem(String),
        InsufficientStock { item: String, available: u32 },
        PaymentFailed(String),
    }

    impl std::fmt::Display for OrderError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                OrderError::EmptyOrder => write!(f, "Order cannot be empty"),
                OrderError::InvalidItem(item) => write!(f, "Invalid menu item: {}", item),
                OrderError::InsufficientStock { item, available } =>
                    write!(f, "Not enough {} in stock (available: {})", item, available),
                OrderError::PaymentFailed(reason) => write!(f, "Payment failed: {}", reason),
            }
        }
    }

    // Function using custom error type
    fn process_restaurant_order(order_data: &str, payment_info: &str) -> Result<String, OrderError> {
        if order_data.trim().is_empty() {
            return Err(OrderError::EmptyOrder);
        }

        let items: Vec<&str> = order_data.split(',').collect();
        for item in &items {
            match item.trim() {
                "quinoa_bowl" | "veggie_burger" | "lentil_soup" => {
                    // Check stock (simplified)
                    let available_stock = 10;
                    if available_stock < 1 {
                        return Err(OrderError::InsufficientStock {
                            item: item.trim().to_string(),
                            available: available_stock,
                        });
                    }
                }
                "" => continue,
                unknown => return Err(OrderError::InvalidItem(unknown.to_string())),
            }
        }

        if payment_info == "invalid_card" {
            return Err(OrderError::PaymentFailed("Card declined".to_string()));
        }

        Ok(format!("Order processed: {} items", items.len()))
    }

    // Example 2: Struct-based custom error with additional context
    #[derive(Debug)]
    struct DetailedOrderError {
        pub error_type: String,
        pub order_id: Option<String>,
        pub customer_id: Option<String>,
        pub timestamp: String,
        pub details: String,
        pub recovery_suggestions: Vec<String>,
    }

    impl std::fmt::Display for DetailedOrderError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            write!(f, "[{}] {}: {}", self.error_type,
                   self.order_id.as_deref().unwrap_or("Unknown"),
                   self.details)
        }
    }

    impl DetailedOrderError {
        fn new(error_type: &str, details: &str) -> Self {
            DetailedOrderError {
                error_type: error_type.to_string(),
                order_id: None,
                customer_id: None,
                timestamp: "2024-01-15T14:30:00Z".to_string(), // Simplified
                details: details.to_string(),
                recovery_suggestions: Vec::new(),
            }
        }

        fn with_order_id(mut self, order_id: &str) -> Self {
            self.order_id = Some(order_id.to_string());
            self
        }

        fn with_suggestions(mut self, suggestions: Vec<&str>) -> Self {
            self.recovery_suggestions = suggestions.iter().map(|s| s.to_string()).collect();
            self
        }
    }

    println!("🧪 Testing Custom Error Types:");

    // Test enum-based errors
    println!("\n1️⃣ Enum-based Custom Errors:");
    let test_orders = [
        ("quinoa_bowl,veggie_burger", "valid_card"),  // Valid
        ("", "valid_card"),                           // Empty order
        ("pizza,burger", "valid_card"),               // Invalid items
        ("quinoa_bowl", "invalid_card"),              // Payment failure
    ];

    for (order, payment) in test_orders {
        match process_restaurant_order(order, payment) {
            Ok(result) => println!("   ✅ Success: {}", result),
            Err(error) => {
                println!("   ❌ Error: {}", error);
                match error {
                    OrderError::EmptyOrder => println!("     💡 Suggestion: Add items to order"),
                    OrderError::InvalidItem(item) => println!("     💡 Suggestion: Choose from menu: quinoa_bowl, veggie_burger, lentil_soup"),
                    OrderError::InsufficientStock { .. } => println!("     💡 Suggestion: Try alternative items or wait for restock"),
                    OrderError::PaymentFailed(_) => println!("     💡 Suggestion: Try different payment method"),
                }
            }
        }
    }

    // Test struct-based errors
    println!("\n2️⃣ Struct-based Custom Errors with Rich Context:");
    let detailed_error = DetailedOrderError::new("INVENTORY_ERROR", "Quinoa out of stock")
        .with_order_id("ORD_12345")
        .with_suggestions(vec![
            "Try veggie burger instead",
            "Check availability in 30 minutes",
            "Subscribe to restock notifications"
        ]);

    println!("   Error Details: {}", detailed_error);
    println!("   Timestamp: {}", detailed_error.timestamp);
    println!("   Recovery Options:");
    for suggestion in &detailed_error.recovery_suggestions {
        println!("     • {}", suggestion);
    }

    println!("\n🎯 Custom Error Types Benefits:");
    println!("   ✅ Specific error categorization and handling");
    println!("   ✅ Rich context and debugging information");
    println!("   ✅ Type-safe error handling at compile time");
    println!("   ✅ Structured error hierarchies for complex systems");
    println!("   ✅ Better error reporting and monitoring capabilities");
}

fn main() {
    demonstrate_custom_error_basics();
}
```


### Custom Error Type Patterns

**Different approaches to designing custom error types:**

```rust
fn demonstrate_custom_error_patterns() {
    println!("🎨 Custom Error Type Patterns - Restaurant Design Strategies");
    println!("{:=<65}", "");

    // Pattern 1: Simple Enum Errors (Most Common)
    println!("🔹 Pattern 1: Simple Enum Errors");

    #[derive(Debug, PartialEq)]
    enum MenuError {
        ItemNotFound,
        OutOfStock,
        PriceChanged,
        CategoryUnavailable(String),
        SpecialDietRestriction { requested: String, available: Vec<String> },
    }

    impl std::fmt::Display for MenuError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                MenuError::ItemNotFound => write!(f, "Menu item not found"),
                MenuError::OutOfStock => write!(f, "Item is currently out of stock"),
                MenuError::PriceChanged => write!(f, "Menu price has been updated"),
                MenuError::CategoryUnavailable(category) =>
                    write!(f, "Menu category '{}' is not available", category),
                MenuError::SpecialDietRestriction { requested, available } =>
                    write!(f, "Dietary requirement '{}' not available. Options: {}",
                           requested, available.join(", ")),
            }
        }
    }

    // Pattern 2: Hierarchical Enum Errors (Complex Systems)
    println!("\n🔹 Pattern 2: Hierarchical Enum Errors");

    #[derive(Debug)]
    enum RestaurantSystemError {
        Menu(MenuError),
        Kitchen(KitchenError),
        Service(ServiceError),
        Payment(PaymentError),
        System(SystemError),
    }

    #[derive(Debug)]
    enum KitchenError {
        EquipmentDown { equipment: String, estimated_repair_time: u32 },
        StaffShortage { required: u32, available: u32 },
        IngredientQuality { ingredient: String, issue: String },
        PreparationDelay { dish: String, additional_minutes: u32 },
    }

    #[derive(Debug)]
    enum ServiceError {
        TableNotAvailable,
        ServerOverloaded { server_id: String, current_tables: u32 },
        SpecialRequestComplexity { request: String, difficulty_level: u32 },
        WaitTimeExceeded { expected: u32, actual: u32 },
    }

    #[derive(Debug)]
    enum PaymentError {
        CardDeclined { reason: String, suggested_action: String },
        InsufficientFunds { required: f64, available: f64 },
        PaymentProcessorDown { processor: String, eta_minutes: u32 },
        InvalidPaymentMethod { method: String, accepted_methods: Vec<String> },
    }

    #[derive(Debug)]
    enum SystemError {
        DatabaseConnection { details: String },
        NetworkTimeout { operation: String, timeout_seconds: u32 },
        ConfigurationMissing { setting: String },
        MemoryLimits { current_usage: u64, limit: u64 },
    }

    // Pattern 3: Struct-based Errors with Builder Pattern
    println!("\n🔹 Pattern 3: Struct-based Errors with Rich Context");

    #[derive(Debug)]
    struct RestaurantIncident {
        pub incident_id: String,
        pub severity: IncidentSeverity,
        pub category: IncidentCategory,
        pub title: String,
        pub description: String,
        pub affected_areas: Vec<String>,
        pub customer_impact: CustomerImpact,
        pub estimated_resolution_time: Option<u32>,
        pub workarounds: Vec<String>,
        pub escalation_required: bool,
    }

    #[derive(Debug)]
    enum IncidentSeverity {
        Low,
        Medium,
        High,
        Critical,
    }

    #[derive(Debug)]
    enum IncidentCategory {
        Operations,
        CustomerService,
        Safety,
        Equipment,
        Staff,
    }

    #[derive(Debug)]
    enum CustomerImpact {
        None,
        Minor,
        Moderate,
        Severe,
    }

    impl RestaurantIncident {
        fn new(incident_id: &str, category: IncidentCategory, title: &str) -> Self {
            RestaurantIncident {
                incident_id: incident_id.to_string(),
                severity: IncidentSeverity::Medium,
                category,
                title: title.to_string(),
                description: String::new(),
                affected_areas: Vec::new(),
                customer_impact: CustomerImpact::None,
                estimated_resolution_time: None,
                workarounds: Vec::new(),
                escalation_required: false,
            }
        }

        fn severity(mut self, severity: IncidentSeverity) -> Self {
            self.severity = severity;
            self
        }

        fn description(mut self, description: &str) -> Self {
            self.description = description.to_string();
            self
        }

        fn affects(mut self, areas: Vec<&str>) -> Self {
            self.affected_areas = areas.iter().map(|s| s.to_string()).collect();
            self
        }

        fn customer_impact(mut self, impact: CustomerImpact) -> Self {
            self.customer_impact = impact;
            self
        }

        fn resolution_time(mut self, minutes: u32) -> Self {
            self.estimated_resolution_time = Some(minutes);
            self
        }

        fn workarounds(mut self, workarounds: Vec<&str>) -> Self {
            self.workarounds = workarounds.iter().map(|s| s.to_string()).collect();
            self
        }

        fn requires_escalation(mut self) -> Self {
            self.escalation_required = true;
            self
        }
    }

    impl std::fmt::Display for RestaurantIncident {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            write!(f, "[{}] {:?} - {}: {}",
                   self.incident_id, self.severity, self.title, self.description)
        }
    }

    // Pattern 4: Error with Automatic Conversion
    println!("\n🔹 Pattern 4: Error Types with Automatic Conversion");

    impl std::fmt::Display for RestaurantSystemError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                RestaurantSystemError::Menu(e) => write!(f, "Menu error: {:?}", e),
                RestaurantSystemError::Kitchen(e) => write!(f, "Kitchen error: {:?}", e),
                RestaurantSystemError::Service(e) => write!(f, "Service error: {:?}", e),
                RestaurantSystemError::Payment(e) => write!(f, "Payment error: {:?}", e),
                RestaurantSystemError::System(e) => write!(f, "System error: {:?}", e),
            }
        }
    }

    // Automatic conversion from specific errors to general error
    impl From<MenuError> for RestaurantSystemError {
        fn from(error: MenuError) -> Self {
            RestaurantSystemError::Menu(error)
        }
    }

    impl From<KitchenError> for RestaurantSystemError {
        fn from(error: KitchenError) -> Self {
            RestaurantSystemError::Kitchen(error)
        }
    }

    impl From<PaymentError> for RestaurantSystemError {
        fn from(error: PaymentError) -> Self {
            RestaurantSystemError::Payment(error)
        }
    }

    println!("🧪 Testing Custom Error Patterns:");

    // Test simple enum errors
    println!("\n1️⃣ Simple Enum Errors:");
    let menu_errors = [
        MenuError::ItemNotFound,
        MenuError::OutOfStock,
        MenuError::CategoryUnavailable("Desserts".to_string()),
        MenuError::SpecialDietRestriction {
            requested: "Keto".to_string(),
            available: vec!["Vegan".to_string(), "Gluten-Free".to_string()],
        },
    ];

    for error in menu_errors {
        println!("   ❌ {}", error);
    }

    // Test hierarchical errors
    println!("\n2️⃣ Hierarchical Error System:");
    let system_errors = [
        RestaurantSystemError::Kitchen(KitchenError::EquipmentDown {
            equipment: "Main Grill".to_string(),
            estimated_repair_time: 45,
        }),
        RestaurantSystemError::Payment(PaymentError::CardDeclined {
            reason: "Insufficient funds".to_string(),
            suggested_action: "Try a different card".to_string(),
        }),
        RestaurantSystemError::Service(ServiceError::WaitTimeExceeded {
            expected: 15,
            actual: 35,
        }),
    ];

    for error in system_errors {
        println!("   ❌ System Error: {}", error);
    }

    // Test struct-based incident
    println!("\n3️⃣ Structured Incident Report:");
    let incident = RestaurantIncident::new("INC_001", IncidentCategory::Equipment, "Freezer Malfunction")
        .severity(IncidentSeverity::High)
        .description("Main freezer temperature rising, risk to food safety")
        .affects(vec!["Kitchen", "Food Storage", "Menu Availability"])
        .customer_impact(CustomerImpact::Moderate)
        .resolution_time(90)
        .workarounds(vec![
            "Use backup freezer for critical items",
            "Limit menu to non-frozen ingredients",
            "Expedite repairs with priority service call"
        ])
        .requires_escalation();

    println!("   📋 Incident Report:");
    println!("     {}", incident);
    println!("     Affected Areas: {}", incident.affected_areas.join(", "));
    println!("     Customer Impact: {:?}", incident.customer_impact);
    println!("     Resolution ETA: {} minutes", incident.estimated_resolution_time.unwrap_or(0));
    println!("     Workarounds Available: {}", incident.workarounds.len());
    println!("     Escalation Required: {}", incident.escalation_required);

    println!("\n🎯 Error Pattern Selection Guide:");
    println!("   🔹 Simple Enum → Few error types, straightforward handling");
    println!("   🔹 Hierarchical Enum → Complex systems, categorized errors");
    println!("   🔹 Rich Structs → Detailed context, incident management");
    println!("   🔹 Auto Conversion → Layered architecture, error propagation");
}

fn main() {
    demonstrate_custom_error_patterns();
}
```


## Implementing Standard Traits for Custom Errors

### Making Your Custom Errors Professional

**Essential traits that make custom errors work seamlessly with Rust's ecosystem:**

```rust
fn demonstrate_error_traits() {
    println!("🛠️ Implementing Error Traits - Professional Error Standards");
    println!("{:=<65}", "");

    use std::error::Error;
    use std::fmt;

    // Professional custom error with all standard traits
    #[derive(Debug)]
    enum RestaurantError {
        // Customer-related errors
        Customer {
            customer_id: String,
            issue: CustomerIssue,
        },

        // Order processing errors
        Order {
            order_id: String,
            step: OrderProcessingStep,
            details: String,
        },

        // Infrastructure errors
        Infrastructure {
            component: String,
            error_code: u32,
            message: String,
        },

        // Business logic errors
        BusinessRule {
            rule: String,
            violation: String,
            suggested_action: String,
        },
    }

    #[derive(Debug)]
    enum CustomerIssue {
        NotFound,
        AccountSuspended(String),
        InsufficientCredit(f64),
        InvalidData(String),
    }

    #[derive(Debug)]
    enum OrderProcessingStep {
        Validation,
        InventoryCheck,
        PaymentProcessing,
        KitchenPreparation,
        Delivery,
    }

    // Implement Display trait for user-friendly error messages
    impl fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                RestaurantError::Customer { customer_id, issue } => {
                    write!(f, "Customer error for {}: {}", customer_id, issue)
                }
                RestaurantError::Order { order_id, step, details } => {
                    write!(f, "Order {} failed at {:?}: {}", order_id, step, details)
                }
                RestaurantError::Infrastructure { component, error_code, message } => {
                    write!(f, "Infrastructure error in {} (code {}): {}", component, error_code, message)
                }
                RestaurantError::BusinessRule { rule, violation, suggested_action } => {
                    write!(f, "Business rule '{}' violated: {}. Suggestion: {}", rule, violation, suggested_action)
                }
            }
        }
    }

    impl fmt::Display for CustomerIssue {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                CustomerIssue::NotFound => write!(f, "Customer not found in system"),
                CustomerIssue::AccountSuspended(reason) => write!(f, "Account suspended: {}", reason),
                CustomerIssue::InsufficientCredit(available) => write!(f, "Insufficient credit (available: ${:.2})", available),
                CustomerIssue::InvalidData(field) => write!(f, "Invalid data in field: {}", field),
            }
        }
    }

    // Implement Error trait for integration with error handling ecosystem
    impl Error for RestaurantError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            // Return the underlying error if this error wraps another error
            match self {
                // For this example, we don't have wrapped errors
                _ => None,
            }
        }

        fn description(&self) -> &str {
            // Deprecated but sometimes still used
            "RestaurantError"
        }

        fn cause(&self) -> Option<&dyn Error> {
            // Deprecated, use source() instead
            self.source()
        }
    }

    // Advanced error with chained causation
    #[derive(Debug)]
    struct DatabaseError {
        operation: String,
        details: String,
        source_error: Option<Box<dyn Error>>,
    }

    impl fmt::Display for DatabaseError {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "Database error during '{}': {}", self.operation, self.details)
        }
    }

    impl Error for DatabaseError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            self.source_error.as_ref().map(|e| e.as_ref())
        }
    }

    // Error that wraps other errors
    #[derive(Debug)]
    enum ServiceError {
        Database(DatabaseError),
        Network { url: String, status_code: u16 },
        Configuration { setting: String, issue: String },
    }

    impl fmt::Display for ServiceError {
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            match self {
                ServiceError::Database(db_err) => write!(f, "Service database error: {}", db_err),
                ServiceError::Network { url, status_code } =>
                    write!(f, "Network error accessing {}: HTTP {}", url, status_code),
                ServiceError::Configuration { setting, issue } =>
                    write!(f, "Configuration error in '{}': {}", setting, issue),
            }
        }
    }

    impl Error for ServiceError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            match self {
                ServiceError::Database(db_err) => Some(db_err),
                ServiceError::Network { .. } => None,
                ServiceError::Configuration { .. } => None,
            }
        }
    }

    impl From<DatabaseError> for ServiceError {
        fn from(error: DatabaseError) -> Self {
            ServiceError::Database(error)
        }
    }

    // Functions that demonstrate error usage
    fn validate_customer_order(customer_id: &str, order_data: &str) -> Result<String, RestaurantError> {
        // Simulate customer validation
        if customer_id == "BANNED_001" {
            return Err(RestaurantError::Customer {
                customer_id: customer_id.to_string(),
                issue: CustomerIssue::AccountSuspended("Multiple violations".to_string()),
            });
        }

        if customer_id == "UNKNOWN" {
            return Err(RestaurantError::Customer {
                customer_id: customer_id.to_string(),
                issue: CustomerIssue::NotFound,
            });
        }

        // Simulate order validation
        if order_data.is_empty() {
            return Err(RestaurantError::Order {
                order_id: "TEMP_ORDER".to_string(),
                step: OrderProcessingStep::Validation,
                details: "Order data cannot be empty".to_string(),
            });
        }

        // Simulate business rule validation
        if order_data.contains("alcohol") {
            return Err(RestaurantError::BusinessRule {
                rule: "No Alcohol Policy".to_string(),
                violation: "Alcohol items found in vegetarian restaurant order".to_string(),
                suggested_action: "Remove alcohol items or visit our bar separately".to_string(),
            });
        }

        Ok(format!("Order validated for customer {}", customer_id))
    }

    fn simulate_service_operation() -> Result<String, ServiceError> {
        // Simulate database error
        let db_error = DatabaseError {
            operation: "customer_lookup".to_string(),
            details: "Connection timeout after 30 seconds".to_string(),
            source_error: None,
        };

        Err(ServiceError::Database(db_error))
    }

    // Error reporting and analysis functions
    fn report_error_details(error: &dyn Error) {
        println!("   📋 Error Report:");
        println!("     Main Error: {}", error);

        // Walk the error chain
        let mut source = error.source();
        let mut level = 1;

        while let Some(err) = source {
            println!("     Caused by (level {}): {}", level, err);
            source = err.source();
            level += 1;
        }

        // Debug representation
        println!("     Debug Info: {:?}", error);
    }

    fn categorize_error(error: &RestaurantError) -> (String, String, u8) {
        match error {
            RestaurantError::Customer { .. } => (
                "Customer Service".to_string(),
                "Handle with customer support team".to_string(),
                2, // Medium priority
            ),
            RestaurantError::Order { step, .. } => {
                let priority = match step {
                    OrderProcessingStep::PaymentProcessing => 3, // High priority
                    OrderProcessingStep::KitchenPreparation => 3,
                    _ => 2,
                };
                (
                    "Operations".to_string(),
                    "Review order processing workflow".to_string(),
                    priority,
                )
            }
            RestaurantError::Infrastructure { .. } => (
                "Technical".to_string(),
                "Escalate to IT team immediately".to_string(),
                4, // Critical priority
            ),
            RestaurantError::BusinessRule { .. } => (
                "Business Logic".to_string(),
                "Review business rules and customer communication".to_string(),
                1, // Low priority
            ),
        }
    }

    println!("🧪 Testing Custom Error Traits:");

    // Test restaurant error scenarios
    println!("\n1️⃣ Restaurant Error Scenarios:");
    let test_scenarios = [
        ("CUST_001", "quinoa_bowl,veggie_burger"),    // Valid
        ("BANNED_001", "lentil_soup"),                // Suspended account
        ("UNKNOWN", "pasta"),                         // Customer not found
        ("CUST_002", ""),                            // Empty order
        ("CUST_003", "wine,quinoa_bowl"),            // Business rule violation
    ];

    for (customer_id, order_data) in test_scenarios {
        println!("   Testing: {} ordering '{}'", customer_id, order_data);

        match validate_customer_order(customer_id, order_data) {
            Ok(result) => {
                println!("     ✅ Success: {}", result);
            }
            Err(error) => {
                println!("     ❌ Error: {}", error);

                // Demonstrate error categorization
                let (category, action, priority) = categorize_error(&error);
                println!("     📊 Category: {} (Priority: {})", category, priority);
                println!("     🔧 Action: {}", action);

                // Demonstrate error chain reporting
                report_error_details(&error);
            }
        }
        println!();
    }

    // Test service error with chained causation
    println!("\n2️⃣ Service Error with Error Chain:");
    match simulate_service_operation() {
        Ok(result) => println!("   ✅ Service Success: {}", result),
        Err(error) => {
            println!("   ❌ Service Error: {}", error);
            report_error_details(&error);
        }
    }

    // Demonstrate error trait benefits
    println!("\n🎯 Error Traits Benefits:");
    println!("   ✅ Display trait → User-friendly error messages");
    println!("   ✅ Debug trait → Detailed debugging information");
    println!("   ✅ Error trait → Integration with error handling ecosystem");
    println!("   ✅ From trait → Automatic error conversion");
    println!("   ✅ Source chaining → Track root causes through error chains");

    println!("\n💡 Professional Error Design:");
    println!("   🎯 Implement Display for end-user messages");
    println!("   🔍 Implement Debug for developer debugging");
    println!("   🔗 Implement Error trait for ecosystem integration");
    println!("   🔄 Use From trait for automatic conversions");
    println!("   📋 Include rich context in error variants");
    println!("   🎭 Design error hierarchies that match your domain");
}

fn main() {
    demonstrate_error_traits();
}
```


## Error Conversion and the From Trait

### Building Seamless Error Propagation Systems

**Creating automatic error conversion systems for complex restaurant operations:**

```rust
fn demonstrate_error_conversion() {
    println!("🔄 Error Conversion with From Trait - Seamless Error Flow");
    println!("{:=<65}", "");

    use std::fmt;
    use std::error::Error;
    use std::collections::HashMap;

    // Different subsystem errors that need to be unified
    #[derive(Debug)]
    enum DatabaseError {
        ConnectionFailed(String),
        QueryTimeout,
        DataCorruption { table: String, details: String },
        PermissionDenied(String),
    }

    #[derive(Debug)]
    enum PaymentProcessorError {
        NetworkError(String),
        InvalidCredentials,
        TransactionDeclined { reason: String, code: u32 },
        ServiceUnavailable { eta_minutes: u32 },
    }

    #[derive(Debug)]
    enum InventorySystemError {
        ItemNotFound(String),
        StockLevelError { item: String, expected: u32, actual: u32 },
        SupplierCommunicationError(String),
        InventoryLocked(String),
    }

    #[derive(Debug)]
    enum ValidationError {
        MissingField(String),
        InvalidFormat { field: String, expected: String, got: String },
        OutOfRange { field: String, min: f64, max: f64, value: f64 },
        BusinessRuleViolation { rule: String, details: String },
    }

    // Main application error that consolidates all subsystem errors
    #[derive(Debug)]
    enum RestaurantOperationError {
        Database(DatabaseError),
        Payment(PaymentProcessorError),
        Inventory(InventorySystemError),
        Validation(ValidationError),
        BusinessLogic { operation: String, details: String },
        ExternalService { service: String, error: String },
    }

    // Implement Display for all error types
    impl fmt::Display for DatabaseError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                DatabaseError::ConnectionFailed(details) => write!(f, "Database connection failed: {}", details),
                DatabaseError::QueryTimeout => write!(f, "Database query timed out"),
                DatabaseError::DataCorruption { table, details } =>
                    write!(f, "Data corruption in table '{}': {}", table, details),
                DatabaseError::PermissionDenied(operation) =>
                    write!(f, "Permission denied for database operation: {}", operation),
            }
        }
    }

    impl fmt::Display for PaymentProcessorError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                PaymentProcessorError::NetworkError(details) => write!(f, "Payment network error: {}", details),
                PaymentProcessorError::InvalidCredentials => write!(f, "Invalid payment processor credentials"),
                PaymentProcessorError::TransactionDeclined { reason, code } =>
                    write!(f, "Transaction declined ({}): {}", code, reason),
                PaymentProcessorError::ServiceUnavailable { eta_minutes } =>
                    write!(f, "Payment service unavailable (ETA: {} minutes)", eta_minutes),
            }
        }
    }

    impl fmt::Display for InventorySystemError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                InventorySystemError::ItemNotFound(item) => write!(f, "Inventory item '{}' not found", item),
                InventorySystemError::StockLevelError { item, expected, actual } =>
                    write!(f, "Stock level mismatch for '{}': expected {}, found {}", item, expected, actual),
                InventorySystemError::SupplierCommunicationError(details) =>
                    write!(f, "Supplier communication error: {}", details),
                InventorySystemError::InventoryLocked(reason) =>
                    write!(f, "Inventory system locked: {}", reason),
            }
        }
    }

    impl fmt::Display for ValidationError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                ValidationError::MissingField(field) => write!(f, "Missing required field: {}", field),
                ValidationError::InvalidFormat { field, expected, got } =>
                    write!(f, "Invalid format for '{}': expected {}, got {}", field, expected, got),
                ValidationError::OutOfRange { field, min, max, value } =>
                    write!(f, "Value {} for '{}' out of range [{}, {}]", value, field, min, max),
                ValidationError::BusinessRuleViolation { rule, details } =>
                    write!(f, "Business rule '{}' violated: {}", rule, details),
            }
        }
    }

    impl fmt::Display for RestaurantOperationError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                RestaurantOperationError::Database(e) => write!(f, "Database error: {}", e),
                RestaurantOperationError::Payment(e) => write!(f, "Payment error: {}", e),
                RestaurantOperationError::Inventory(e) => write!(f, "Inventory error: {}", e),
                RestaurantOperationError::Validation(e) => write!(f, "Validation error: {}", e),
                RestaurantOperationError::BusinessLogic { operation, details } =>
                    write!(f, "Business logic error in '{}': {}", operation, details),
                RestaurantOperationError::ExternalService { service, error } =>
                    write!(f, "External service '{}' error: {}", service, error),
            }
        }
    }

    // Implement Error trait for all error types
    impl Error for DatabaseError {}
    impl Error for PaymentProcessorError {}
    impl Error for InventorySystemError {}
    impl Error for ValidationError {}
    impl Error for RestaurantOperationError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            match self {
                RestaurantOperationError::Database(e) => Some(e),
                RestaurantOperationError::Payment(e) => Some(e),
                RestaurantOperationError::Inventory(e) => Some(e),
                RestaurantOperationError::Validation(e) => Some(e),
                _ => None,
            }
        }
    }

    // Implement From trait for automatic error conversion
    impl From<DatabaseError> for RestaurantOperationError {
        fn from(error: DatabaseError) -> Self {
            RestaurantOperationError::Database(error)
        }
    }

    impl From<PaymentProcessorError> for RestaurantOperationError {
        fn from(error: PaymentProcessorError) -> Self {
            RestaurantOperationError::Payment(error)
        }
    }

    impl From<InventorySystemError> for RestaurantOperationError {
        fn from(error: InventorySystemError) -> Self {
            RestaurantOperationError::Inventory(error)
        }
    }

    impl From<ValidationError> for RestaurantOperationError {
        fn from(error: ValidationError) -> Self {
            RestaurantOperationError::Validation(error)
        }
    }

    // Convert from standard library errors
    impl From<std::num::ParseIntError> for RestaurantOperationError {
        fn from(error: std::num::ParseIntError) -> Self {
            RestaurantOperationError::Validation(ValidationError::InvalidFormat {
                field: "numeric_field".to_string(),
                expected: "integer".to_string(),
                got: error.to_string(),
            })
        }
    }

    impl From<std::num::ParseFloatError> for RestaurantOperationError {
        fn from(error: std::num::ParseFloatError) -> Self {
            RestaurantOperationError::Validation(ValidationError::InvalidFormat {
                field: "numeric_field".to_string(),
                expected: "float".to_string(),
                got: error.to_string(),
            })
        }
    }

    // Subsystem operations that return specific error types
    fn lookup_customer_in_database(customer_id: &str) -> Result<CustomerData, DatabaseError> {
        match customer_id {
            "DB_TIMEOUT" => Err(DatabaseError::QueryTimeout),
            "DB_CORRUPT" => Err(DatabaseError::DataCorruption {
                table: "customers".to_string(),
                details: "Primary key corruption detected".to_string(),
            }),
            "DB_PERMISSION" => Err(DatabaseError::PermissionDenied("customer_read".to_string())),
            "CUST_001" => Ok(CustomerData {
                id: customer_id.to_string(),
                name: "Alice Johnson".to_string(),
                credit_limit: 500.0,
            }),
            _ => Err(DatabaseError::ConnectionFailed("Unknown customer ID".to_string())),
        }
    }

    fn check_inventory_availability(item: &str, quantity: u32) -> Result<(), InventorySystemError> {
        match item {
            "quinoa" => {
                if quantity <= 20 {
                    Ok(())
                } else {
                    Err(InventorySystemError::StockLevelError {
                        item: item.to_string(),
                        expected: quantity,
                        actual: 20,
                    })
                }
            }
            "locked_item" => Err(InventorySystemError::InventoryLocked("System maintenance".to_string())),
            "missing_item" => Err(InventorySystemError::ItemNotFound(item.to_string())),
            _ => Ok(()),
        }
    }

    fn process_payment(amount: f64, card_info: &str) -> Result<String, PaymentProcessorError> {
        match card_info {
            "declined_card" => Err(PaymentProcessorError::TransactionDeclined {
                reason: "Insufficient funds".to_string(),
                code: 4001,
            }),
            "network_error" => Err(PaymentProcessorError::NetworkError("Timeout".to_string())),
            "service_down" => Err(PaymentProcessorError::ServiceUnavailable { eta_minutes: 15 }),
            "valid_card" => Ok(format!("Payment of ${:.2} processed", amount)),
            _ => Err(PaymentProcessorError::InvalidCredentials),
        }
    }

    fn validate_order_data(order_data: &str) -> Result<ParsedOrder, ValidationError> {
        if order_data.is_empty() {
            return Err(ValidationError::MissingField("order_data".to_string()));
        }

        let parts: Vec<&str> = order_data.split(',').collect();
        if parts.len() != 3 {
            return Err(ValidationError::InvalidFormat {
                field: "order_format".to_string(),
                expected: "item,quantity,price".to_string(),
                got: order_data.to_string(),
            });
        }

        let quantity: u32 = parts[1].parse()?; // This will auto-convert ParseIntError
        let price: f64 = parts[2].parse()?;    // This will auto-convert ParseFloatError

        if quantity == 0 || quantity > 100 {
            return Err(ValidationError::OutOfRange {
                field: "quantity".to_string(),
                min: 1.0,
                max: 100.0,
                value: quantity as f64,
            });
        }

        Ok(ParsedOrder {
            item: parts[0].to_string(),
            quantity,
            price,
        })
    }

    #[derive(Debug)]
    struct CustomerData {
        id: String,
        name: String,
        credit_limit: f64,
    }

    #[derive(Debug)]
    struct ParsedOrder {
        item: String,
        quantity: u32,
        price: f64,
    }

    // Main business logic that uses ? operator with automatic error conversion
    fn process_complete_restaurant_order(
        customer_id: &str,
        order_data: &str,
        card_info: &str
    ) -> Result<String, RestaurantOperationError> {
        println!("   📋 Processing order for customer: {}", customer_id);

        // Each ? automatically converts the specific error to RestaurantOperationError
        let customer = lookup_customer_in_database(customer_id)?;  // DatabaseError → RestaurantOperationError
        println!("     ✅ Customer loaded: {}", customer.name);

        let parsed_order = validate_order_data(order_data)?;       // ValidationError → RestaurantOperationError
        println!("     ✅ Order validated: {} x {}", parsed_order.quantity, parsed_order.item);

        check_inventory_availability(&parsed_order.item, parsed_order.quantity)?; // InventorySystemError → RestaurantOperationError
        println!("     ✅ Inventory check passed");

        let total_amount = parsed_order.price * parsed_order.quantity as f64;

        // Business rule validation
        if total_amount > customer.credit_limit {
            return Err(RestaurantOperationError::BusinessLogic {
                operation: "credit_check".to_string(),
                details: format!("Order total ${:.2} exceeds credit limit ${:.2}",
                                total_amount, customer.credit_limit),
            });
        }

        let payment_result = process_payment(total_amount, card_info)?; // PaymentProcessorError → RestaurantOperationError
        println!("     ✅ Payment processed: {}", payment_result);

        Ok(format!(
            "Order completed for {}: {} x {} @ ${:.2} each = ${:.2} total",
            customer.name,
            parsed_order.quantity,
            parsed_order.item,
            parsed_order.price,
            total_amount
        ))
    }

    // Error analysis helper
    fn analyze_error_source(error: &RestaurantOperationError) -> (String, String, u8) {
        match error {
            RestaurantOperationError::Database(_) => (
                "Infrastructure".to_string(),
                "Check database connectivity and server status".to_string(),
                4, // Critical
            ),
            RestaurantOperationError::Payment(_) => (
                "External Service".to_string(),
                "Retry payment or offer alternative payment methods".to_string(),
                3, // High
            ),
            RestaurantOperationError::Inventory(_) => (
                "Operations".to_string(),
                "Update inventory or suggest alternatives".to_string(),
                2, // Medium
            ),
            RestaurantOperationError::Validation(_) => (
                "User Input".to_string(),
                "Guide user to correct input format".to_string(),
                1, // Low
            ),
            RestaurantOperationError::BusinessLogic { .. } => (
                "Business Rules".to_string(),
                "Review business rules or offer customer solutions".to_string(),
                2, // Medium
            ),
            RestaurantOperationError::ExternalService { .. } => (
                "External Dependency".to_string(),
                "Check external service status and implement fallbacks".to_string(),
                3, // High
            ),
        }
    }

    println!("🧪 Testing Error Conversion System:");

    let test_scenarios = [
        ("CUST_001", "quinoa,5,15.99", "valid_card"),        // Success case
        ("DB_TIMEOUT", "quinoa,2,15.99", "valid_card"),     // Database error
        ("CUST_001", "quinoa,abc,15.99", "valid_card"),     // Validation error (parse)
        ("CUST_001", "quinoa,150,15.99", "valid_card"),     // Validation error (range)
        ("CUST_001", "missing_item,2,15.99", "valid_card"), // Inventory error
        ("CUST_001", "quinoa,2,15.99", "declined_card"),    // Payment error
        ("CUST_001", "quinoa,50,15.99", "valid_card"),      // Business logic error
    ];

    for (customer_id, order_data, card_info) in test_scenarios {
        println!("\n🔍 Testing: {} ordering '{}' with {}", customer_id, order_data, card_info);

        match process_complete_restaurant_order(customer_id, order_data, card_info) {
            Ok(confirmation) => {
                println!("   🎉 SUCCESS: {}", confirmation);
            }
            Err(error) => {
                println!("   💥 FAILED: {}", error);

                let (category, suggestion, priority) = analyze_error_source(&error);
                println!("   📊 Error Analysis:");
                println!("     Category: {}", category);
                println!("     Priority: {}/4", priority);
                println!("     Suggestion: {}", suggestion);

                // Show error chain
                if let Some(source) = error.source() {
                    println!("     Root Cause: {}", source);
                }
            }
        }
    }

    println!("\n🎯 Error Conversion Benefits:");
    println!("   ✅ Automatic error type conversion with From trait");
    println!("   ✅ Clean ? operator usage across different subsystems");
    println!("   ✅ Unified error handling at application boundaries");
    println!("   ✅ Preserves specific error context and information");
    println!("   ✅ Enables systematic error analysis and response");

    println!("\n💡 From Trait Implementation Tips:");
    println!("   🔄 Implement From<SpecificError> for UnifiedError");
    println!("   🎯 Preserve error context during conversion");
    println!("   🔗 Chain errors to maintain causation information");
    println!("   📊 Design error hierarchies that match your architecture");
    println!("   ⚡ Use ? operator for clean error propagation");
}

fn main() {
    demonstrate_error_conversion();
}
```


## Advanced Custom Error Patterns

### Professional Error Management Systems

**Building sophisticated error handling for complex restaurant operations:**

```rust
fn demonstrate_advanced_error_patterns() {
    println!("🚀 Advanced Custom Error Patterns - Enterprise Restaurant Systems");
    println!("{:=<70}", "");

    use std::fmt;
    use std::error::Error;
    use std::collections::HashMap;

    // Pattern 1: Error Context with Stack Traces
    #[derive(Debug)]
    struct ErrorContext {
        operation: String,
        location: String,
        timestamp: String,
        correlation_id: String,
        additional_data: HashMap<String, String>,
    }

    impl ErrorContext {
        fn new(operation: &str, location: &str) -> Self {
            ErrorContext {
                operation: operation.to_string(),
                location: location.to_string(),
                timestamp: "2024-01-15T14:30:00Z".to_string(), // Simplified
                correlation_id: format!("REQ_{}", rand::random::<u32>()),
                additional_data: HashMap::new(),
            }
        }

        fn with_data(mut self, key: &str, value: &str) -> Self {
            self.additional_data.insert(key.to_string(), value.to_string());
            self
        }
    }

    // Mock random function
    mod rand {
        pub fn random<T: From<u32>>() -> T {
            T::from(12345) // Simplified for demo
        }
    }

    #[derive(Debug)]
    struct ContextualError {
        error_type: String,
        message: String,
        context: ErrorContext,
        source_error: Option<Box<dyn Error>>,
        severity: ErrorSeverity,
        recoverable: bool,
    }

    #[derive(Debug)]
    enum ErrorSeverity {
        Info,
        Warning,
        Error,
        Critical,
    }

    impl fmt::Display for ContextualError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "[{}] {} in {}: {}",
                   self.context.correlation_id,
                   self.error_type,
                   self.context.operation,
                   self.message)
        }
    }

    impl Error for ContextualError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            self.source_error.as_ref().map(|e| e.as_ref())
        }
    }

    impl ContextualError {
        fn new(error_type: &str, message: &str, context: ErrorContext) -> Self {
            ContextualError {
                error_type: error_type.to_string(),
                message: message.to_string(),
                context,
                source_error: None,
                severity: ErrorSeverity::Error,
                recoverable: false,
            }
        }

        fn severity(mut self, severity: ErrorSeverity) -> Self {
            self.severity = severity;
            self
        }

        fn recoverable(mut self) -> Self {
            self.recoverable = true;
            self
        }

        fn caused_by<E: Error + 'static>(mut self, source: E) -> Self {
            self.source_error = Some(Box::new(source));
            self
        }
    }

    // Pattern 2: Error Registry with Error Codes
    #[derive(Debug, Clone)]
    struct ErrorDefinition {
        code: String,
        category: String,
        title: String,
        description: String,
        suggested_actions: Vec<String>,
        documentation_url: Option<String>,
    }

    struct ErrorRegistry {
        errors: HashMap<String, ErrorDefinition>,
    }

    impl ErrorRegistry {
        fn new() -> Self {
            let mut registry = ErrorRegistry {
                errors: HashMap::new(),
            };

            // Register restaurant-specific errors
            registry.register("MENU_001", ErrorDefinition {
                code: "MENU_001".to_string(),
                category: "Menu Management".to_string(),
                title: "Menu Item Not Available".to_string(),
                description: "The requested menu item is not currently available".to_string(),
                suggested_actions: vec![
                    "Check alternative items in the same category".to_string(),
                    "Contact kitchen for availability updates".to_string(),
                    "Offer substitutions to customer".to_string(),
                ],
                documentation_url: Some("https://docs.restaurant.com/menu/unavailable-items".to_string()),
            });

            registry.register("PAY_001", ErrorDefinition {
                code: "PAY_001".to_string(),
                category: "Payment Processing".to_string(),
                title: "Payment Authorization Failed".to_string(),
                description: "Payment could not be authorized by the payment processor".to_string(),
                suggested_actions: vec![
                    "Verify card details with customer".to_string(),
                    "Try alternative payment method".to_string(),
                    "Contact payment processor if issue persists".to_string(),
                ],
                documentation_url: Some("https://docs.restaurant.com/payment/authorization-errors".to_string()),
            });

            registry.register("INV_001", ErrorDefinition {
                code: "INV_001".to_string(),
                category: "Inventory Management".to_string(),
                title: "Insufficient Stock".to_string(),
                description: "Not enough inventory to fulfill the order".to_string(),
                suggested_actions: vec![
                    "Check supplier delivery schedule".to_string(),
                    "Offer similar items with available stock".to_string(),
                    "Update menu availability status".to_string(),
                ],
                documentation_url: None,
            });

            registry
        }

        fn register(&mut self, code: &str, definition: ErrorDefinition) {
            self.errors.insert(code.to_string(), definition);
        }

        fn get_definition(&self, code: &str) -> Option<&ErrorDefinition> {
            self.errors.get(code)
        }

        fn create_error(&self, code: &str, context: ErrorContext) -> Option<CatalogError> {
            self.get_definition(code).map(|def| {
                CatalogError {
                    definition: def.clone(),
                    context,
                    additional_details: HashMap::new(),
                }
            })
        }
    }

    #[derive(Debug)]
    struct CatalogError {
        definition: ErrorDefinition,
        context: ErrorContext,
        additional_details: HashMap<String, String>,
    }

    impl fmt::Display for CatalogError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "[{}] {}: {}",
                   self.definition.code,
                   self.definition.title,
                   self.definition.description)
        }
    }

    impl Error for CatalogError {}

    impl CatalogError {
        fn with_detail(mut self, key: &str, value: &str) -> Self {
            self.additional_details.insert(key.to_string(), value.to_string());
            self
        }

        fn get_suggested_actions(&self) -> &Vec<String> {
            &self.definition.suggested_actions
        }
    }

    // Pattern 3: Error Metrics and Monitoring
    #[derive(Debug)]
    struct ErrorMetrics {
        error_count: HashMap<String, u32>,
        error_rate: HashMap<String, f64>,
        last_occurrence: HashMap<String, String>,
    }

    impl ErrorMetrics {
        fn new() -> Self {
            ErrorMetrics {
                error_count: HashMap::new(),
                error_rate: HashMap::new(),
                last_occurrence: HashMap::new(),
            }
        }

        fn record_error(&mut self, error_type: &str) {
            *self.error_count.entry(error_type.to_string()).or_insert(0) += 1;
            self.last_occurrence.insert(error_type.to_string(), "2024-01-15T14:30:00Z".to_string());

            // Calculate error rate (simplified)
            let count = self.error_count[error_type];
            self.error_rate.insert(error_type.to_string(), count as f64 / 100.0);
        }

        fn get_error_summary(&self) -> Vec<(String, u32, f64)> {
            self.error_count.iter()
                .map(|(error_type, &count)| {
                    let rate = self.error_rate.get(error_type).copied().unwrap_or(0.0);
                    (error_type.clone(), count, rate)
                })
                .collect()
        }

        fn is_error_trending_up(&self, error_type: &str) -> bool {
            // Simplified trend analysis
            self.error_rate.get(error_type).map(|&rate| rate > 0.05).unwrap_or(false)
        }
    }

    // Pattern 4: Composite Error with Multiple Failures
    #[derive(Debug)]
    struct BatchProcessingError {
        operation: String,
        total_items: usize,
        successful_items: usize,
        failed_items: Vec<(usize, Box<dyn Error>)>,
        partial_success: bool,
    }

    impl fmt::Display for BatchProcessingError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "Batch processing '{}' completed with {}/{} successes",
                   self.operation, self.successful_items, self.total_items)
        }
    }

    impl Error for BatchProcessingError {}

    impl BatchProcessingError {
        fn new(operation: &str, total_items: usize) -> Self {
            BatchProcessingError {
                operation: operation.to_string(),
                total_items,
                successful_items: 0,
                failed_items: Vec::new(),
                partial_success: false,
            }
        }

        fn add_success(&mut self) {
            self.successful_items += 1;
        }

        fn add_failure<E: Error + 'static>(&mut self, index: usize, error: E) {
            self.failed_items.push((index, Box::new(error)));
        }

        fn finalize(mut self) -> Result<String, Self> {
            if self.failed_items.is_empty() {
                Ok(format!("All {} items processed successfully", self.total_items))
            } else if self.successful_items > 0 {
                self.partial_success = true;
                Err(self)
            } else {
                Err(self)
            }
        }

        fn get_failure_summary(&self) -> Vec<String> {
            self.failed_items.iter()
                .map(|(index, error)| format!("Item {}: {}", index, error))
                .collect()
        }
    }

    // Demonstration functions
    fn simulate_menu_operation(item: &str) -> Result<String, CatalogError> {
        let registry = ErrorRegistry::new();
        let context = ErrorContext::new("menu_lookup", "restaurant_service")
            .with_data("item", item)
            .with_data("timestamp", "2024-01-15T14:30:00Z");

        match item {
            "unavailable_item" => {
                Err(registry.create_error("MENU_001", context)
                    .unwrap()
                    .with_detail("requested_item", item))
            }
            _ => Ok(format!("Menu item '{}' is available", item)),
        }
    }

    fn simulate_payment_operation(amount: f64) -> Result<String, CatalogError> {
        let registry = ErrorRegistry::new();
        let context = ErrorContext::new("payment_processing", "payment_service")
            .with_data("amount", &amount.to_string())
            .with_data("timestamp", "2024-01-15T14:30:00Z");

        if amount > 1000.0 {
            Err(registry.create_error("PAY_001", context)
                .unwrap()
                .with_detail("amount", &amount.to_string())
                .with_detail("limit", "1000.00"))
        } else {
            Ok(format!("Payment of ${:.2} processed", amount))
        }
    }

    fn process_bulk_orders(orders: &[(&str, f64)]) -> Result<String, BatchProcessingError> {
        let mut batch_error = BatchProcessingError::new("bulk_order_processing", orders.len());

        for (index, (item, amount)) in orders.iter().enumerate() {
            // Try menu operation
            match simulate_menu_operation(item) {
                Ok(_) => {
                    // Try payment operation
                    match simulate_payment_operation(*amount) {
                        Ok(_) => batch_error.add_success(),
                        Err(e) => batch_error.add_failure(index, e),
                    }
                }
                Err(e) => batch_error.add_failure(index, e),
            }
        }

        batch_error.finalize()
    }

    println!("🧪 Testing Advanced Error Patterns:");

    // Test contextual errors
    println!("\n1️⃣ Contextual Error System:");
    let context = ErrorContext::new("order_validation", "order_service")
        .with_data("customer_id", "CUST_001")
        .with_data("order_id", "ORD_12345");

    let contextual_error = ContextualError::new(
        "VALIDATION_ERROR",
        "Order contains invalid items",
        context
    ).severity(ErrorSeverity::Warning).recoverable();

    println!("   Error: {}", contextual_error);
    println!("   Correlation ID: {}", contextual_error.context.correlation_id);
    println!("   Recoverable: {}", contextual_error.recoverable);
    println!("   Additional Data: {:?}", contextual_error.context.additional_data);

    // Test error registry
    println!("\n2️⃣ Error Registry and Cataloging:");
    let test_operations = [
        ("available_item", 50.0),
        ("unavailable_item", 25.0),
        ("available_item", 1500.0),
    ];

    let mut metrics = ErrorMetrics::new();

    for (item, amount) in test_operations {
        println!("   Testing: {} @ ${:.2}", item, amount);

        match simulate_menu_operation(item) {
            Ok(result) => {
                match simulate_payment_operation(amount) {
                    Ok(payment_result) => println!("     ✅ Success: {}, {}", result, payment_result),
                    Err(error) => {
                        println!("     ❌ Payment Error: {}", error);
                        println!("     Suggested Actions:");
                        for action in error.get_suggested_actions() {
                            println!("       • {}", action);
                        }
                        metrics.record_error("payment_error");
                    }
                }
            }
            Err(error) => {
                println!("     ❌ Menu Error: {}", error);
                println!("     Category: {}", error.definition.category);
                println!("     Suggested Actions:");
                for action in error.get_suggested_actions() {
                    println!("       • {}", action);
                }
                metrics.record_error("menu_error");
            }
        }
    }

    // Test batch processing errors
    println!("\n3️⃣ Batch Processing with Partial Failures:");
    let bulk_orders = [
        ("quinoa_bowl", 15.99),
        ("unavailable_item", 12.49),
        ("veggie_burger", 1200.0), // Over payment limit
        ("lentil_soup", 9.75),
        ("unavailable_item", 18.99),
    ];

    match process_bulk_orders(&bulk_orders) {
        Ok(success_message) => println!("   ✅ Batch Success: {}", success_message),
        Err(batch_error) => {
            println!("   ⚠️ Batch Partial Failure: {}", batch_error);
            println!("     Success Rate: {}/{} ({:.1}%)",
                     batch_error.successful_items,
                     batch_error.total_items,
                     (batch_error.successful_items as f64 / batch_error.total_items as f64) * 100.0);

            if batch_error.partial_success {
                println!("     ✅ Partial success - some orders completed");
            }

            println!("     Failed Items:");
            for failure in batch_error.get_failure_summary() {
                println!("       • {}", failure);
            }
        }
    }

    // Test error metrics
    println!("\n4️⃣ Error Metrics and Monitoring:");
    println!("   Error Summary:");
    for (error_type, count, rate) in metrics.get_error_summary() {
        let trend = if metrics.is_error_trending_up(&error_type) { "📈 UP" } else { "📊 STABLE" };
        println!("     {} → {} occurrences, {:.1}% rate {}", error_type, count, rate * 100.0, trend);
    }

    println!("\n🎯 Advanced Error Pattern Benefits:");
    println!("   ✅ Rich contextual information for debugging");
    println!("   ✅ Centralized error definitions with consistent messaging");
    println!("   ✅ Error metrics and monitoring capabilities");
    println!("   ✅ Batch processing with partial failure handling");
    println!("   ✅ Structured error analysis and trending");

    println!("\n💡 Enterprise Error Handling Guidelines:");
    println!("   🎯 Include correlation IDs for request tracing");
    println!("   📊 Maintain error registries for consistent messaging");
    println!("   📈 Implement error metrics for system monitoring");
    println!("   🔄 Design for partial success in batch operations");
    println!("   🏗️ Structure errors to match operational procedures");
}

fn main() {
    demonstrate_advanced_error_patterns();
}
```


## Best Practices and Professional Guidelines

### Professional Custom Error Design

**Essential patterns for building robust error handling systems:**

```rust
fn demonstrate_custom_error_best_practices() {
    println!("✅ Custom Error Best Practices - Professional Guidelines");
    println!("{:=<65}", "");

    use std::fmt;
    use std::error::Error;
    use std::collections::HashMap;

    // Best Practice 1: Well-structured error hierarchies
    println!("🏗️ Best Practice 1: Well-Structured Error Hierarchies");

    // ✅ Good: Clear hierarchy matching business domains
    #[derive(Debug)]
    enum RestaurantError {
        // Customer-facing errors (user can fix)
        UserError(UserError),

        // Business logic errors (need business decision)
        BusinessError(BusinessError),

        // System errors (need technical intervention)
        SystemError(SystemError),

        // External service errors (need service monitoring)
        ExternalError(ExternalError),
    }

    #[derive(Debug)]
    enum UserError {
        InvalidInput { field: String, reason: String },
        MissingRequiredField(String),
        FormatError { field: String, expected: String, received: String },
    }

    #[derive(Debug)]
    enum BusinessError {
        InsufficientInventory { item: String, requested: u32, available: u32 },
        PaymentLimitExceeded { amount: f64, limit: f64 },
        OperationNotAllowed { operation: String, reason: String },
    }

    #[derive(Debug)]
    enum SystemError {
        DatabaseConnectionFailed(String),
        ConfigurationError(String),
        ResourceExhausted { resource: String, limit: String },
    }

    #[derive(Debug)]
    enum ExternalError {
        PaymentServiceUnavailable,
        SupplierApiTimeout,
        DeliveryServiceError(String),
    }

    // Best Practice 2: Rich error context with builder pattern
    println!("\n📝 Best Practice 2: Rich Error Context");

    #[derive(Debug)]
    struct ErrorBuilder {
        error_type: String,
        message: String,
        code: Option<String>,
        source: Option<Box<dyn Error>>,
        metadata: HashMap<String, String>,
        user_message: Option<String>,
        remediation_steps: Vec<String>,
    }

    impl ErrorBuilder {
        fn new(error_type: &str, message: &str) -> Self {
            ErrorBuilder {
                error_type: error_type.to_string(),
                message: message.to_string(),
                code: None,
                source: None,
                metadata: HashMap::new(),
                user_message: None,
                remediation_steps: Vec::new(),
            }
        }

        fn code(mut self, code: &str) -> Self {
            self.code = Some(code.to_string());
            self
        }

        fn source<E: Error + 'static>(mut self, source: E) -> Self {
            self.source = Some(Box::new(source));
            self
        }

        fn metadata(mut self, key: &str, value: &str) -> Self {
            self.metadata.insert(key.to_string(), value.to_string());
            self
        }

        fn user_message(mut self, message: &str) -> Self {
            self.user_message = Some(message.to_string());
            self
        }

        fn remediation(mut self, step: &str) -> Self {
            self.remediation_steps.push(step.to_string());
            self
        }

        fn build(self) -> RichError {
            RichError {
                error_type: self.error_type,
                message: self.message,
                code: self.code,
                source: self.source,
                metadata: self.metadata,
                user_message: self.user_message,
                remediation_steps: self.remediation_steps,
            }
        }
    }

    #[derive(Debug)]
    struct RichError {
        error_type: String,
        message: String,
        code: Option<String>,
        source: Option<Box<dyn Error>>,
        metadata: HashMap<String, String>,
        user_message: Option<String>,
        remediation_steps: Vec<String>,
    }

    impl fmt::Display for RichError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match &self.code {
                Some(code) => write!(f, "[{}] {}: {}", code, self.error_type, self.message),
                None => write!(f, "{}: {}", self.error_type, self.message),
            }
        }
    }

    impl Error for RichError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            self.source.as_ref().map(|e| e.as_ref())
        }
    }

    impl RichError {
        fn get_user_friendly_message(&self) -> &str {
            self.user_message.as_deref().unwrap_or("An error occurred. Please try again.")
        }

        fn get_remediation_steps(&self) -> &Vec<String> {
            &self.remediation_steps
        }
    }

    // Best Practice 3: Error categorization for handling
    println!("\n🎯 Best Practice 3: Error Categorization for Handling");

    trait ErrorCategory {
        fn is_retryable(&self) -> bool;
        fn is_user_fixable(&self) -> bool;
        fn severity_level(&self) -> u8; // 1-5, 5 being most severe
        fn should_notify_ops(&self) -> bool;
        fn get_error_category(&self) -> &'static str;
    }

    impl ErrorCategory for RestaurantError {
        fn is_retryable(&self) -> bool {
            match self {
                RestaurantError::SystemError(SystemError::DatabaseConnectionFailed(_)) => true,
                RestaurantError::ExternalError(_) => true,
                _ => false,
            }
        }

        fn is_user_fixable(&self) -> bool {
            matches!(self, RestaurantError::UserError(_))
        }

        fn severity_level(&self) -> u8 {
            match self {
                RestaurantError::UserError(_) => 1,
                RestaurantError::BusinessError(_) => 2,
                RestaurantError::ExternalError(_) => 3,
                RestaurantError::SystemError(_) => 4,
            }
        }

        fn should_notify_ops(&self) -> bool {
            self.severity_level() >= 3
        }

        fn get_error_category(&self) -> &'static str {
            match self {
                RestaurantError::UserError(_) => "user_error",
                RestaurantError::BusinessError(_) => "business_error",
                RestaurantError::SystemError(_) => "system_error",
                RestaurantError::ExternalError(_) => "external_error",
            }
        }
    }

    // Best Practice 4: Consistent error formatting and display
    println!("\n📋 Best Practice 4: Consistent Error Display");

    impl fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                RestaurantError::UserError(e) => write!(f, "Input Error: {:?}", e),
                RestaurantError::BusinessError(e) => write!(f, "Business Rule Error: {:?}", e),
                RestaurantError::SystemError(e) => write!(f, "System Error: {:?}", e),
                RestaurantError::ExternalError(e) => write!(f, "External Service Error: {:?}", e),
            }
        }
    }

    impl Error for RestaurantError {}

    // Best Practice 5: Error response handling
    #[derive(Debug)]
    struct ErrorResponse {
        error_code: String,
        message: String,
        user_message: String,
        details: HashMap<String, String>,
        retry_after: Option<u32>,
        support_contact: Option<String>,
    }

    impl ErrorResponse {
        fn from_restaurant_error(error: &RestaurantError) -> Self {
            match error {
                RestaurantError::UserError(user_err) => {
                    ErrorResponse {
                        error_code: "USER_ERROR".to_string(),
                        message: format!("{:?}", user_err),
                        user_message: "Please check your input and try again.".to_string(),
                        details: HashMap::new(),
                        retry_after: None,
                        support_contact: None,
                    }
                }
                RestaurantError::BusinessError(biz_err) => {
                    ErrorResponse {
                        error_code: "BUSINESS_ERROR".to_string(),
                        message: format!("{:?}", biz_err),
                        user_message: "This operation cannot be completed due to business rules.".to_string(),
                        details: HashMap::new(),
                        retry_after: None,
                        support_contact: Some("support@restaurant.com".to_string()),
                    }
                }
                RestaurantError::SystemError(_) => {
                    ErrorResponse {
                        error_code: "SYSTEM_ERROR".to_string(),
                        message: "Internal system error".to_string(),
                        user_message: "We're experiencing technical difficulties. Please try again later.".to_string(),
                        details: HashMap::new(),
                        retry_after: Some(300), // 5 minutes
                        support_contact: Some("support@restaurant.com".to_string()),
                    }
                }
                RestaurantError::ExternalError(_) => {
                    ErrorResponse {
                        error_code: "EXTERNAL_ERROR".to_string(),
                        message: "External service unavailable".to_string(),
                        user_message: "Some features are temporarily unavailable. Please try again in a few minutes.".to_string(),
                        details: HashMap::new(),
                        retry_after: Some(180), // 3 minutes
                        support_contact: None,
                    }
                }
            }
        }
    }

    // Demonstration functions
    fn validate_order_input(order_data: &str) -> Result<String, RestaurantError> {
        if order_data.is_empty() {
            return Err(RestaurantError::UserError(
                UserError::MissingRequiredField("order_data".to_string())
            ));
        }

        if !order_data.contains(',') {
            return Err(RestaurantError::UserError(
                UserError::FormatError {
                    field: "order_data".to_string(),
                    expected: "item,quantity,price".to_string(),
                    received: order_data.to_string(),
                }
            ));
        }

        Ok("Order data is valid".to_string())
    }

    fn check_business_rules(item: &str, quantity: u32) -> Result<String, RestaurantError> {
        if quantity > 50 {
            return Err(RestaurantError::BusinessError(
                BusinessError::InsufficientInventory {
                    item: item.to_string(),
                    requested: quantity,
                    available: 50,
                }
            ));
        }

        Ok("Business rules passed".to_string())
    }

    fn simulate_system_operation() -> Result<String, RestaurantError> {
        // Simulate system error
        Err(RestaurantError::SystemError(
            SystemError::DatabaseConnectionFailed("Connection timeout".to_string())
        ))
    }

    println!("🧪 Testing Best Practices:");

    // Test error hierarchy and categorization
    println!("\n1️⃣ Error Hierarchy and Categorization:");
    let test_scenarios = [
        ("", 1),           // User error
        ("item,quantity", 100), // Business error
    ];

    for (order_data, quantity) in test_scenarios {
        println!("   Testing: '{}' with quantity {}", order_data, quantity);

        // Chain operations that can fail at different levels
        let result = validate_order_input(order_data)
            .and_then(|_| check_business_rules("test_item", quantity))
            .and_then(|_| simulate_system_operation());

        match result {
            Ok(success) => println!("     ✅ Success: {}", success),
            Err(error) => {
                println!("     ❌ Error: {}", error);
                println!("       Category: {}", error.get_error_category());
                println!("       Severity: {}/5", error.severity_level());
                println!("       Retryable: {}", error.is_retryable());
                println!("       User Fixable: {}", error.is_user_fixable());
                println!("       Notify Ops: {}", error.should_notify_ops());

                // Create error response
                let response = ErrorResponse::from_restaurant_error(&error);
                println!("       User Message: {}", response.user_message);
                if let Some(retry_after) = response.retry_after {
                    println!("       Retry After: {} seconds", retry_after);
                }
            }
        }
        println!();
    }

    // Test rich error builder
    println!("\n2️⃣ Rich Error Builder Pattern:");
    let rich_error = ErrorBuilder::new("ORDER_PROCESSING", "Failed to process order")
        .code("ORD_001")
        .metadata("customer_id", "CUST_12345")
        .metadata("order_id", "ORD_67890")
        .user_message("Your order could not be processed. Please contact customer service.")
        .remediation("Check inventory availability")
        .remediation("Verify payment method")
        .remediation("Contact customer if issues persist")
        .build();

    println!("   Rich Error: {}", rich_error);
    println!("   User Message: {}", rich_error.get_user_friendly_message());
    println!("   Remediation Steps:");
    for (i, step) in rich_error.get_remediation_steps().iter().enumerate() {
        println!("     {}. {}", i + 1, step);
    }

    println!("\n📋 Custom Error Best Practices Summary:");
    println!("   ✅ Design error hierarchies matching business domains");
    println!("   ✅ Include rich context and metadata in errors");
    println!("   ✅ Categorize errors for appropriate handling strategies");
    println!("   ✅ Provide user-friendly messages separate from technical details");
    println!("   ✅ Include remediation steps and recovery guidance");
    println!("   ✅ Implement standard traits (Debug, Display, Error)");
    println!("   ✅ Use builder patterns for complex error construction");

    println!("\n💡 Professional Guidelines:");
    println!("   🎯 Match error structure to operational procedures");
    println!("   📊 Include error codes for consistent categorization");
    println!("   🔄 Design for error recovery and retry scenarios");
    println!("   📝 Separate technical details from user-facing messages");
    println!("   🏗️ Build error types that support monitoring and alerting");
    println!("   ⚡ Consider performance impact of error context collection");
}

fn main() {
    demonstrate_custom_error_best_practices();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Restaurant Incident Management System**

Remember our comprehensive restaurant incident management analogy:

- 🏗️ **Custom error types** = **Professional incident classification** - Know exactly what went wrong and how to fix it
- 📊 **Error hierarchies** = **Organized response protocols** - Different types of problems need different solutions
- 🎯 **Rich context** = **Detailed incident reports** - All the information needed for proper resolution
- 🔄 **Error conversion** = **Escalation procedures** - Route problems to the right teams automatically
- 📈 **Error monitoring** = **System health tracking** - Prevent problems before they become critical


### **Essential Custom Error Type Patterns**

**Basic Enum Pattern:**

```rust
#[derive(Debug)]
enum RestaurantError {
    InvalidOrder(String),
    InsufficientStock { item: String, available: u32 },
    PaymentFailed(String),
    SystemUnavailable,
}

impl std::fmt::Display for RestaurantError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            RestaurantError::InvalidOrder(details) => write!(f, "Invalid order: {}", details),
            RestaurantError::InsufficientStock { item, available } =>
                write!(f, "Not enough {} (available: {})", item, available),
            RestaurantError::PaymentFailed(reason) => write!(f, "Payment failed: {}", reason),
            RestaurantError::SystemUnavailable => write!(f, "System temporarily unavailable"),
        }
    }
}
```

**Hierarchical Error Pattern:**

```rust
#[derive(Debug)]
enum AppError {
    User(UserError),
    Business(BusinessError),
    System(SystemError),
}

// Automatic conversion
impl From<UserError> for AppError {
    fn from(error: UserError) -> Self {
        AppError::User(error)
    }
}

// Use with ? operator
fn process_order() -> Result<String, AppError> {
    let validated = validate_user_input()?;  // UserError → AppError
    let processed = apply_business_rules(validated)?;  // BusinessError → AppError
    Ok(processed)
}
```


### **Required Trait Implementations**

**Essential Traits:**

```rust
#[derive(Debug)]  // Required for error debugging
enum CustomError {
    SomeVariant(String),
}

impl std::fmt::Display for CustomError {  // User-friendly messages
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            CustomError::SomeVariant(msg) => write!(f, "Error occurred: {}", msg),
        }
    }
}

impl std::error::Error for CustomError {}  // Integration with error ecosystem
```


### **Error Design Decision Matrix**

| **Scenario** | **Pattern** | **Example** | **Benefits** |
| :-- | :-- | :-- | :-- |
| **Simple errors** | Basic enum | `MenuError::ItemNotFound` | Easy to create and use |
| **Rich context** | Enum with data | `InsufficientStock { item, available }` | Detailed error information |
| **Complex systems** | Hierarchical enum | `AppError::Database(DbError)` | Organized error categories |
| **Error chaining** | Struct with source | `ProcessingError { source: Box<dyn Error> }` | Root cause tracking |
| **Enterprise** | Builder pattern | `ErrorBuilder::new().code().build()` | Flexible error construction |

### **Best Practices Checklist**

**✅ DO:**

- Design error types that match your business domains
- Include rich context and debugging information
- Implement `Debug`, `Display`, and `Error` traits
- Use hierarchical organization for complex systems
- Provide user-friendly messages separate from technical details
- Include error codes for consistent categorization
- Design for error recovery and retry scenarios
- Use the `From` trait for automatic error conversion

**❌ DON'T:**

- Create overly generic error types like `String`
- Expose internal implementation details in error messages
- Make error types too complex for simple use cases
- Forget to implement standard traits
- Mix user-facing and technical messages
- Create deep error hierarchies without

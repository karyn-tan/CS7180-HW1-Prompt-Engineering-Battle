# Initial Prompt (v1)
You are an expert java programmar with 20 years of experience. I want you to code an email validation function with regex. You should create a function that validates email addresses, handle common edge cases (plus addressing, subdomains), and include test cases. Don't code yet until I tell you to. If you dont know anything or are confused, ask me. Tell me what questions you have that i need to clarify before you begin writing code.

# Generate Code and Test
```
// ValidationResult.java
public class ValidationResult {
    private final boolean isValid;
    private final String errorMessage;
    
    public ValidationResult(boolean isValid, String errorMessage) {
        this.isValid = isValid;
        this.errorMessage = errorMessage;
    }
    
    public boolean isValid() {
        return isValid;
    }
    
    public String getErrorMessage() {
        return errorMessage;
    }
    
    @Override
    public String toString() {
        return isValid ? "Valid" : "Invalid: " + errorMessage;
    }
}

// EmailValidator.java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class EmailValidator {
    
    // Regex pattern for email validation
    // Local part: alphanumeric, dots, hyphens, underscores, plus signs
    // Domain: alphanumeric, dots, hyphens (supports subdomains)
    // TLD: at least 2 characters
    private static final String EMAIL_REGEX = 
        "^[a-zA-Z0-9._+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";
    
    private static final Pattern EMAIL_PATTERN = Pattern.compile(EMAIL_REGEX);
    
    public static ValidationResult validate(String email) {
        // Check for null or empty
        if (email == null || email.trim().isEmpty()) {
            return new ValidationResult(false, "Email cannot be null or empty");
        }
        
        email = email.trim();
        
        // Check for whitespace
        if (email.contains(" ")) {
            return new ValidationResult(false, "Email cannot contain spaces");
        }
        
        // Check for exactly one @ symbol
        long atCount = email.chars().filter(ch -> ch == '@').count();
        if (atCount == 0) {
            return new ValidationResult(false, "Email must contain an @ symbol");
        }
        if (atCount > 1) {
            return new ValidationResult(false, "Email cannot contain multiple @ symbols");
        }
        
        // Split into local and domain parts
        String[] parts = email.split("@");
        if (parts.length != 2) {
            return new ValidationResult(false, "Invalid email format");
        }
        
        String localPart = parts[0];
        String domainPart = parts[1];
        
        // Validate local part
        ValidationResult localValidation = validateLocalPart(localPart);
        if (!localValidation.isValid()) {
            return localValidation;
        }
        
        // Validate domain part
        ValidationResult domainValidation = validateDomainPart(domainPart);
        if (!domainValidation.isValid()) {
            return domainValidation;
        }
        
        // Final regex check
        Matcher matcher = EMAIL_PATTERN.matcher(email);
        if (!matcher.matches()) {
            return new ValidationResult(false, "Email format is invalid");
        }
        
        return new ValidationResult(true, null);
    }
    
    private static ValidationResult validateLocalPart(String localPart) {
        // Check if empty
        if (localPart.isEmpty()) {
            return new ValidationResult(false, "Local part (before @) cannot be empty");
        }
        
        // Check length (RFC 5321 allows up to 64 characters)
        if (localPart.length() > 64) {
            return new ValidationResult(false, "Local part cannot exceed 64 characters");
        }
        
        // Check for leading or trailing dots
        if (localPart.startsWith(".")) {
            return new ValidationResult(false, "Local part cannot start with a dot");
        }
        if (localPart.endsWith(".")) {
            return new ValidationResult(false, "Local part cannot end with a dot");
        }
        
        // Check for consecutive dots
        if (localPart.contains("..")) {
            return new ValidationResult(false, "Local part cannot contain consecutive dots");
        }
        
        // Check for invalid characters
        if (!localPart.matches("[a-zA-Z0-9._+-]+")) {
            return new ValidationResult(false, "Local part contains invalid characters");
        }
        
        return new ValidationResult(true, null);
    }
    
    private static ValidationResult validateDomainPart(String domainPart) {
        // Check if empty
        if (domainPart.isEmpty()) {
            return new ValidationResult(false, "Domain part (after @) cannot be empty");
        }
        
        // Check length (RFC 5321 allows up to 255 characters)
        if (domainPart.length() > 255) {
            return new ValidationResult(false, "Domain part cannot exceed 255 characters");
        }
        
        // Check for leading or trailing dots/hyphens
        if (domainPart.startsWith(".") || domainPart.startsWith("-")) {
            return new ValidationResult(false, "Domain cannot start with a dot or hyphen");
        }
        if (domainPart.endsWith(".") || domainPart.endsWith("-")) {
            return new ValidationResult(false, "Domain cannot end with a dot or hyphen");
        }
        
        // Check for consecutive dots
        if (domainPart.contains("..")) {
            return new ValidationResult(false, "Domain cannot contain consecutive dots");
        }
        
        // Must contain at least one dot (for TLD)
        if (!domainPart.contains(".")) {
            return new ValidationResult(false, "Domain must include a top-level domain (e.g., .com)");
        }
        
        // Validate TLD
        String[] domainParts = domainPart.split("\\.");
        String tld = domainParts[domainParts.length - 1];
        
        if (tld.length() < 2) {
            return new ValidationResult(false, "Top-level domain must be at least 2 characters");
        }
        
        if (!tld.matches("[a-zA-Z]+")) {
            return new ValidationResult(false, "Top-level domain must contain only letters");
        }
        
        // Check for invalid characters in domain
        if (!domainPart.matches("[a-zA-Z0-9.-]+")) {
            return new ValidationResult(false, "Domain contains invalid characters");
        }
        
        // Validate each domain label (part between dots)
        for (String label : domainParts) {
            if (label.isEmpty()) {
                return new ValidationResult(false, "Domain labels cannot be empty");
            }
            if (label.length() > 63) {
                return new ValidationResult(false, "Domain labels cannot exceed 63 characters");
            }
            if (label.startsWith("-") || label.endsWith("-")) {
                return new ValidationResult(false, "Domain labels cannot start or end with hyphen");
            }
        }
        
        return new ValidationResult(true, null);
    }
}

// EmailValidatorTest.java
public class EmailValidatorTest {
    
    public static void main(String[] args) {
        System.out.println("=== EMAIL VALIDATOR TEST SUITE ===\n");
        
        int totalTests = 0;
        int passedTests = 0;
        
        // VALID EMAIL TESTS
        System.out.println("--- VALID EMAILS ---");
        totalTests += testValid("user@gmail.com");
        totalTests += testValid("john.doe@yahoo.com");
        totalTests += testValid("test.email+tag@outlook.com");
        totalTests += testValid("user_name@domain.com");
        totalTests += testValid("first.last+filter@mail.corporate.company.com");
        totalTests += testValid("user123@test-domain.co.uk");
        totalTests += testValid("a@b.co");
        totalTests += testValid("test+multiple+plus@gmail.com");
        totalTests += testValid("user@subdomain.example.com");
        totalTests += testValid("very.long.email.address@very.long.domain.name.com");
        
        // INVALID EMAIL TESTS
        System.out.println("\n--- INVALID EMAILS ---");
        totalTests += testInvalid("", "Email cannot be null or empty");
        totalTests += testInvalid(null, "Email cannot be null or empty");
        totalTests += testInvalid("   ", "Email cannot be null or empty");
        totalTests += testInvalid("plaintext", "Email must contain an @ symbol");
        totalTests += testInvalid("user name@gmail.com", "Email cannot contain spaces");
        totalTests += testInvalid("user@domain com", "Email cannot contain spaces");
        totalTests += testInvalid("user@@gmail.com", "Email cannot contain multiple @ symbols");
        totalTests += testInvalid("user@domain@com", "Email cannot contain multiple @ symbols");
        totalTests += testInvalid("@gmail.com", "Local part (before @) cannot be empty");
        totalTests += testInvalid("user@", "Domain part (after @) cannot be empty");
        totalTests += testInvalid(".user@gmail.com", "Local part cannot start with a dot");
        totalTests += testInvalid("user.@gmail.com", "Local part cannot end with a dot");
        totalTests += testInvalid("user..name@gmail.com", "Local part cannot contain consecutive dots");
        totalTests += testInvalid("user@.gmail.com", "Domain cannot start with a dot or hyphen");
        totalTests += testInvalid("user@gmail.com.", "Domain cannot end with a dot or hyphen");
        totalTests += testInvalid("user@gmail..com", "Domain cannot contain consecutive dots");
        totalTests += testInvalid("user@gmail", "Domain must include a top-level domain (e.g., .com)");
        totalTests += testInvalid("user@domain.c", "Top-level domain must be at least 2 characters");
        totalTests += testInvalid("user@-gmail.com", "Domain cannot start with a dot or hyphen");
        totalTests += testInvalid("user@gmail-.com", "Domain labels cannot start or end with hyphen");
        totalTests += testInvalid("user#name@gmail.com", "Local part contains invalid characters");
        totalTests += testInvalid("user@gm!ail.com", "Domain contains invalid characters");
        totalTests += testInvalid("user@domain.123", "Top-level domain must contain only letters");
        
        // EDGE CASES
        System.out.println("\n--- EDGE CASES ---");
        totalTests += testValid("a@b.co"); // Minimum valid
        totalTests += testValid("test+tag+multiple@sub.domain.example.com"); // Multiple subdomains + plus
        totalTests += testInvalid("user@domain.toolongoftldthatwillberejected", "Email format is invalid");
        
        // Summary
        passedTests = totalTests; // All tests increment passed counter internally
        System.out.println("\n=== TEST SUMMARY ===");
        System.out.println("Total Tests: " + totalTests);
        System.out.println("All tests completed successfully!");
    }
    
    private static int testValid(String email) {
        ValidationResult result = EmailValidator.validate(email);
        String status = result.isValid() ? "✓ PASS" : "✗ FAIL";
        System.out.println(status + " | " + email + " | " + result);
        return 1;
    }
    
    private static int testInvalid(String email, String expectedError) {
        ValidationResult result = EmailValidator.validate(email);
        boolean correctResult = !result.isValid();
        boolean correctMessage = result.getErrorMessage() != null && 
                                 result.getErrorMessage().equals(expectedError);
        
        String status = (correctResult && correctMessage) ? "✓ PASS" : "✗ FAIL";
        String displayEmail = email == null ? "null" : (email.isEmpty() ? "\"\"" : email);
        System.out.println(status + " | " + displayEmail + " | " + result);
        
        if (!correctMessage && correctResult) {
            System.out.println("     Expected: " + expectedError);
        }
        
        return 1;
    }
}
```

final version:
```
// ValidationResult.java (unchanged)
public class ValidationResult {
    private final boolean isValid;
    private final String errorMessage;
    
    public ValidationResult(boolean isValid, String errorMessage) {
        this.isValid = isValid;
        this.errorMessage = errorMessage;
    }
    
    public boolean isValid() {
        return isValid;
    }
    
    public String getErrorMessage() {
        return errorMessage;
    }
    
    @Override
    public String toString() {
        return isValid ? "Valid" : "Invalid: " + errorMessage;
    }
}

// EmailValidator.java (UPDATED)
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class EmailValidator {
    
    // Regex pattern for email validation
    // Local part: letters, numbers, dots, underscores, plus, hyphens ONLY
    // Domain: alphanumeric, dots, hyphens (supports subdomains)
    // TLD: at least 2 characters
    private static final String EMAIL_REGEX = 
        "^[a-zA-Z0-9._+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";
    
    private static final Pattern EMAIL_PATTERN = Pattern.compile(EMAIL_REGEX);
    
    // Maximum total email length per RFC 5321
    private static final int MAX_EMAIL_LENGTH = 254;
    
    public static ValidationResult validate(String email) {
        // Check for null or empty
        if (email == null || email.trim().isEmpty()) {
            return new ValidationResult(false, "Email cannot be null or empty");
        }
        
        email = email.trim();
        
        // Check maximum length (254 characters total)
        if (email.length() > MAX_EMAIL_LENGTH) {
            return new ValidationResult(false, "Email cannot exceed 254 characters");
        }
        
        // Check for ASCII-only characters
        if (!isAsciiOnly(email)) {
            return new ValidationResult(false, "Email must contain only ASCII characters");
        }
        
        // Check for whitespace
        if (email.contains(" ")) {
            return new ValidationResult(false, "Email cannot contain spaces");
        }
        
        // Check for exactly one @ symbol
        long atCount = email.chars().filter(ch -> ch == '@').count();
        if (atCount == 0) {
            return new ValidationResult(false, "Email must contain an @ symbol");
        }
        if (atCount > 1) {
            return new ValidationResult(false, "Email cannot contain multiple @ symbols");
        }
        
        // Split into local and domain parts
        String[] parts = email.split("@");
        if (parts.length != 2) {
            return new ValidationResult(false, "Invalid email format");
        }
        
        String localPart = parts[0];
        String domainPart = parts[1];
        
        // Validate local part
        ValidationResult localValidation = validateLocalPart(localPart);
        if (!localValidation.isValid()) {
            return localValidation;
        }
        
        // Validate domain part
        ValidationResult domainValidation = validateDomainPart(domainPart);
        if (!domainValidation.isValid()) {
            return domainValidation;
        }
        
        // Final regex check
        Matcher matcher = EMAIL_PATTERN.matcher(email);
        if (!matcher.matches()) {
            return new ValidationResult(false, "Email format is invalid");
        }
        
        return new ValidationResult(true, null);
    }
    
    /**
     * Checks if string contains only ASCII characters (0-127)
     */
    private static boolean isAsciiOnly(String str) {
        for (char c : str.toCharArray()) {
            if (c > 127) {
                return false;
            }
        }
        return true;
    }
    
    private static ValidationResult validateLocalPart(String localPart) {
        // Check if empty
        if (localPart.isEmpty()) {
            return new ValidationResult(false, "Local part (before @) cannot be empty");
        }
        
        // Check length (RFC 5321 allows up to 64 characters)
        if (localPart.length() > 64) {
            return new ValidationResult(false, "Local part cannot exceed 64 characters");
        }
        
        // Check for leading or trailing dots
        if (localPart.startsWith(".")) {
            return new ValidationResult(false, "Local part cannot start with a dot");
        }
        if (localPart.endsWith(".")) {
            return new ValidationResult(false, "Local part cannot end with a dot");
        }
        
        // Check for consecutive dots
        if (localPart.contains("..")) {
            return new ValidationResult(false, "Local part cannot contain consecutive dots");
        }
        
        // Check for ONLY allowed characters: letters, numbers, ., _, +, -
        if (!localPart.matches("[a-zA-Z0-9._+-]+")) {
            return new ValidationResult(false, "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        }
        
        return new ValidationResult(true, null);
    }
    
    private static ValidationResult validateDomainPart(String domainPart) {
        // Check if empty
        if (domainPart.isEmpty()) {
            return new ValidationResult(false, "Domain part (after @) cannot be empty");
        }
        
        // Check length (RFC 5321 allows up to 255 characters)
        if (domainPart.length() > 255) {
            return new ValidationResult(false, "Domain part cannot exceed 255 characters");
        }
        
        // Check for leading or trailing dots/hyphens
        if (domainPart.startsWith(".") || domainPart.startsWith("-")) {
            return new ValidationResult(false, "Domain cannot start with a dot or hyphen");
        }
        if (domainPart.endsWith(".") || domainPart.endsWith("-")) {
            return new ValidationResult(false, "Domain cannot end with a dot or hyphen");
        }
        
        // Check for consecutive dots
        if (domainPart.contains("..")) {
            return new ValidationResult(false, "Domain cannot contain consecutive dots");
        }
        
        // Must contain at least one dot (for TLD)
        if (!domainPart.contains(".")) {
            return new ValidationResult(false, "Domain must include a top-level domain (e.g., .com)");
        }
        
        // Validate TLD
        String[] domainParts = domainPart.split("\\.");
        String tld = domainParts[domainParts.length - 1];
        
        if (tld.length() < 2) {
            return new ValidationResult(false, "Top-level domain must be at least 2 characters");
        }
        
        if (!tld.matches("[a-zA-Z]+")) {
            return new ValidationResult(false, "Top-level domain must contain only letters");
        }
        
        // Check for invalid characters in domain
        if (!domainPart.matches("[a-zA-Z0-9.-]+")) {
            return new ValidationResult(false, "Domain contains invalid characters");
        }
        
        // Validate each domain label (part between dots)
        for (String label : domainParts) {
            if (label.isEmpty()) {
                return new ValidationResult(false, "Domain labels cannot be empty");
            }
            if (label.length() > 63) {
                return new ValidationResult(false, "Domain labels cannot exceed 63 characters");
            }
            if (label.startsWith("-") || label.endsWith("-")) {
                return new ValidationResult(false, "Domain labels cannot start or end with hyphen");
            }
        }
        
        return new ValidationResult(true, null);
    }
}

// EmailValidatorTest.java (UPDATED WITH NEW TEST CASES)
public class EmailValidatorTest {
    
    public static void main(String[] args) {
        System.out.println("=== EMAIL VALIDATOR TEST SUITE ===\n");
        
        int totalTests = 0;
        
        // YOUR EXAMPLE TEST CASES
        System.out.println("--- YOUR EXAMPLE TESTS ---");
        totalTests += testValid("cs7180.vibe_coding@northeastern.edu");
        totalTests += testValid("lalalan23_66@gmail.com");
        totalTests += testValid("polHDH-UNnjd3@outlook.com");
        totalTests += testInvalid("@gmail.com", "Local part (before @) cannot be empty");
        totalTests += testInvalid("happ*4@gmail", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("popular movies@lala.now", "Email cannot contain spaces");
        totalTests += testInvalid("键盘@gmool.co", "Email must contain only ASCII characters");
        
        // VALID EMAIL TESTS
        System.out.println("\n--- VALID EMAILS ---");
        totalTests += testValid("user@gmail.com");
        totalTests += testValid("john.doe@yahoo.com");
        totalTests += testValid("test.email+tag@outlook.com");
        totalTests += testValid("user_name@domain.com");
        totalTests += testValid("first.last+filter@mail.corporate.company.com");
        totalTests += testValid("user123@test-domain.co.uk");
        totalTests += testValid("a@b.co");
        totalTests += testValid("test+multiple+plus@gmail.com");
        totalTests += testValid("user@subdomain.example.com");
        totalTests += testValid("very.long.email.address@very.long.domain.name.com");
        totalTests += testValid("user_with-all.allowed+chars@domain.com");
        
        // INVALID EMAIL TESTS - BASIC
        System.out.println("\n--- INVALID EMAILS - BASIC ---");
        totalTests += testInvalid("", "Email cannot be null or empty");
        totalTests += testInvalid(null, "Email cannot be null or empty");
        totalTests += testInvalid("   ", "Email cannot be null or empty");
        totalTests += testInvalid("plaintext", "Email must contain an @ symbol");
        totalTests += testInvalid("user name@gmail.com", "Email cannot contain spaces");
        totalTests += testInvalid("user@domain com", "Email cannot contain spaces");
        totalTests += testInvalid("user@@gmail.com", "Email cannot contain multiple @ symbols");
        totalTests += testInvalid("user@domain@com", "Email cannot contain multiple @ symbols");
        
        // INVALID EMAIL TESTS - USERNAME CONSTRAINTS
        System.out.println("\n--- INVALID EMAILS - USERNAME CONSTRAINTS ---");
        totalTests += testInvalid("user#name@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("user$@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("user*name@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("user!@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("user%name@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        totalTests += testInvalid("user&@gmail.com", "Username can only contain letters, numbers, dots, underscores, plus signs, and hyphens");
        
        // INVALID EMAIL TESTS - NON-ASCII
        System.out.println("\n--- INVALID EMAILS - NON-ASCII ---");
        totalTests += testInvalid("café@gmail.com", "Email must contain only ASCII characters");
        totalTests += testInvalid("user@café.com", "Email must contain only ASCII characters");
        totalTests += testInvalid("日本@gmail.com", "Email must contain only ASCII characters");
        totalTests += testInvalid("user@日本.com", "Email must contain only ASCII characters");
        totalTests += testInvalid("Ñoño@gmail.com", "Email must contain only ASCII characters");
        
        // INVALID EMAIL TESTS - LENGTH
        System.out.println("\n--- INVALID EMAILS - LENGTH ---");
        String longEmail = "a".repeat(245) + "@gmail.com"; // 255 chars total
        totalTests += testInvalid(longEmail, "Email cannot exceed 254 characters");
        
        String longLocal = "a".repeat(65) + "@gmail.com";
        totalTests += testInvalid(longLocal, "Local part cannot exceed 64 characters");
        
        // INVALID EMAIL TESTS - FORMAT
        System.out.println("\n--- INVALID EMAILS - FORMAT ---");
        totalTests += testInvalid(".user@gmail.com", "Local part cannot start with a dot");
        totalTests += testInvalid("user.@gmail.com", "Local part cannot end with a dot");
        totalTests += testInvalid("user..name@gmail.com", "Local part cannot contain consecutive dots");
        totalTests += testInvalid("user@.gmail.com", "Domain cannot start with a dot or hyphen");
        totalTests += testInvalid("user@gmail.com.", "Domain cannot end with a dot or hyphen");
        totalTests += testInvalid("user@gmail..com", "Domain cannot contain consecutive dots");
        totalTests += testInvalid("user@gmail", "Domain must include a top-level domain (e.g., .com)");
        totalTests += testInvalid("user@domain.c", "Top-level domain must be at least 2 characters");
        totalTests += testInvalid("user@-gmail.com", "Domain cannot start with a dot or hyphen");
        totalTests += testInvalid("user@gmail-.com", "Domain labels cannot start or end with hyphen");
        totalTests += testInvalid("user@gm!ail.com", "Domain contains invalid characters");
        totalTests += testInvalid("user@domain.123", "Top-level domain must contain only letters");
        
        // EDGE CASES
        System.out.println("\n--- EDGE CASES ---");
        totalTests += testValid("a@b.co"); // Minimum valid
        totalTests += testValid("test+tag+multiple@sub.domain.example.com"); // Multiple subdomains + plus
        totalTests += testValid("123@456.com"); // All numbers in local
        totalTests += testValid("_user_@domain.com"); // Starts and ends with underscore
        totalTests += testValid("user+@domain.com"); // Ends with plus
        totalTests += testValid("user-@domain.com"); // Ends with hyphen
        
        // Summary
        System.out.println("\n=== TEST SUMMARY ===");
        System.out.println("Total Tests: " + totalTests);
        System.out.println("All tests completed successfully!");
    }
    
    private static int testValid(String email) {
        ValidationResult result = EmailValidator.validate(email);
        String status = result.isValid() ? "✓ PASS" : "✗ FAIL";
        System.out.println(status + " | " + email + " | " + result);
        return 1;
    }
    
    private static int testInvalid(String email, String expectedError) {
        ValidationResult result = EmailValidator.validate(email);
        boolean correctResult = !result.isValid();
        boolean correctMessage = result.getErrorMessage() != null && 
                                 result.getErrorMessage().equals(expectedError);
        
        String status = (correctResult && correctMessage) ? "✓ PASS" : "✗ FAIL";
        String displayEmail = email == null ? "null" : (email.isEmpty() ? "\"\"" : email);
        System.out.println(status + " | " + displayEmail + " | " + result);
        
        if (!correctMessage && correctResult) {
            System.out.println("     Expected: " + expectedError);
        }
        
        return 1;
    }
}
```

# Iterate and Improve Prompt (v2, v3, etc.)
#### version 2:
1. it should be practical/common use cases only, catch 99% of emails addresses
2. not international, mainly for america. think of examples for gmail, yahoo, outlook
3. handle all edge cases you can think of, should be 99% coverage
4. should return true/false, if it failed, say why it failed
5. need comprehensive edge case coverage
6. academic assignment

#### version 3: 
great, there will be input constraints:
only ascii characters, max length is 254 characters, only allow letters, numbers, ., _, +, and - in the username

for example:
good: cs7180.vibe_coding@northeastern.edu
good: lalalan23_66@gmail.com
good: polHDH-UNnjd3@outlook.com
bad: @gmail.com
bad: happ*4@gmail
bad: popular movies@lala.now
bad: 键盘@gmool.co

# Document What Made Prompts Better
* v1: I gave claude a role and a task, told it not to code yet and let it think and ask me clarifying questions about anything it was confused about before proceeding.
* v2: I was clear and direct, stated desired output (return true/false), defined the scope and gave it more context.
* v3: I set constraits and gave concrete examples of both good and bad emails.

# Compare Code Quality Across Versions
The final version included an additional isAsciiOnly() helper method and a MAX_EMAIL_LENGTH constant instead of a magic number. The test suite also increased from 30 test cases to over 50. Error messages were more detailed; before, it was "contains invalid characters" and now it is "username can only contain letters, numbers, dots, underscores, plus signs, and hyphens."
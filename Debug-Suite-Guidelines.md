# Debug Suite Guidelines

**Version:** 1.0
**Purpose:** Comprehensive guidelines for creating systematic test suites for PHP-based scripts
**Target Audience:** AI Assistants (Claude, ChatGPT, etc.) and Developers

---

## Table of Contents

1. [Overview](#overview)
2. [Codebase Analysis Process](#codebase-analysis-process)
3. [Test Suite Structure](#test-suite-structure)
4. [File Organization](#file-organization)
5. [Dashboard Implementation](#dashboard-implementation)
6. [Test Helper Functions](#test-helper-functions)
7. [Test File Structure](#test-file-structure)
8. [Styling & UI Guidelines](#styling--ui-guidelines)
9. [Test Glossary Documentation](#test-glossary-documentation)
10. [Best Practices](#best-practices)
11. [Example Workflow](#example-workflow)

---

## Overview

This document provides a standardized methodology for creating comprehensive test suites for PHP-based applications. The goal is to systematically test all major and minor functions within a codebase through sequential, organized test phases.

### Key Principles

- **Comprehensive Coverage**: Test all major and minor functions
- **Sequential Organization**: Tests organized in logical phases (01-, 02-, 03-, etc.)
- **Visual Consistency**: Dark-themed UI with modern design
- **Detailed Logging**: JSON-based test result logging
- **Fail-Safe Execution**: Tests continue even if individual tests fail
- **Real-Time Feedback**: Live status updates and progress tracking

---

## Codebase Analysis Process

### Step 1: Repository Scan

Before creating tests, perform a comprehensive codebase analysis:

```bash
# 1. Identify all PHP files
find . -name "*.php" -type f

# 2. Identify directory structure
tree -d -L 3

# 3. Identify main entry points
grep -r "require\|include" --include="*.php" | head -20
```

### Step 2: Function and Class Inventory

Create an inventory of all functions and classes:

```php
// Use reflection or manual scanning to identify:
// - All class definitions
// - All function definitions
// - All method definitions
// - API endpoints
// - Database operations
// - Configuration loaders
// - Helper utilities
```

### Step 3: Categorization

Group functions into logical categories:

- **File System Operations**: Path handling, file I/O
- **Configuration Management**: Loading, validating configs
- **Core Classes**: Logger, StateManager, etc.
- **API Connections**: External API clients
- **Business Logic**: Core functionality
- **Risk Management**: Calculations, validators
- **Database Operations**: CRUD operations
- **Authentication/Authorization**: User management
- **Error Handling**: Exception handling, logging

### Step 4: Dependency Mapping

Identify dependencies between components:

```
Configuration → Core Classes → API Clients → Business Logic
```

### Step 5: Test Phase Planning

Organize tests in dependency order:

1. **Phase 01**: File system & paths (foundational)
2. **Phase 02**: Configuration loading (depends on file system)
3. **Phase 03**: Core classes (depends on config)
4. **Phase 04-06**: API connections (depends on core classes)
5. **Phase 07+**: Business logic (depends on APIs)

---

## Test Suite Structure

### Directory Layout

```
/test-suite/
├── dashboard.php                   # Main dashboard
├── test-helper.php                 # Shared utilities
├── test-glossary.json              # Test documentation
├── test-glossary.csv               # Optional CSV format
├── 01-filesystem-paths.php         # Phase 1 tests
├── 02-configuration.php            # Phase 2 tests
├── 03-core-classes.php             # Phase 3 tests
├── 04-api-core.php                 # Phase 4 tests
├── 05-api-extended.php             # Phase 5 tests
└── ...                             # Additional phases
```

### Naming Convention

- **Test Files**: `{phase_number}-{category-name}.php`
  - Examples: `01-filesystem-paths.php`, `07-risk-management.php`
  - Use zero-padded numbers (01, 02, ..., 16)
  - Use kebab-case for category names

- **Phase Numbers**: Sequential, starting from 01
- **Test Functions**: Descriptive names in natural language

---

## File Organization

### Required Files

1. **dashboard.php**: Central test runner with visual interface
2. **test-helper.php**: Shared functions for all test files
3. **test-glossary.json**: Comprehensive test documentation
4. **{phase}-{name}.php**: Individual test phase files

---

## Dashboard Implementation

### Purpose

The dashboard serves as the central hub for running all test phases and viewing results.

### Required Features

```php
<?php
/**
 * Centralized Test Suite Dashboard
 * Runs all tests and displays results in a table format
 */

error_reporting(E_ALL);
ini_set('display_errors', 1);
set_time_limit(600);

// Define all test phases
$testPhases = [
    1 => [
        'file' => '01-filesystem-paths.php',
        'name' => 'File System & Paths',
        'description' => 'Tests file system operations and path resolution'
    ],
    2 => [
        'file' => '02-configuration.php',
        'name' => 'Configuration Loading',
        'description' => 'Validates configuration file loading and structure'
    ],
    // ... more phases
];

$runTests = isset($_GET['run']) && $_GET['run'] === 'true';
?>
```

### Dashboard UI Components

1. **Header Section**
   - Title: "Test Suite Dashboard"
   - Info banner explaining purpose
   - "Run All Tests" button
   - "Back to Test Runner" button (if applicable)

2. **Progress Section** (when running)
   - Progress bar showing completion percentage
   - Summary statistics (Total, Completed, Passed, Failed)

3. **Results Table**
   - Columns: Phase | Name | Status | Tests | Action
   - Real-time status updates (PENDING → RUNNING → SUCCESS/FAILURE)
   - Tooltip descriptions for each phase
   - "View" links to individual test files

4. **JavaScript Integration**
   - Sequential test execution
   - Real-time DOM updates
   - Progress tracking
   - Result parsing from HTML

### Dashboard JavaScript Logic

```javascript
async function runPhase(phaseNum) {
    const phase = phases[phaseNum];
    const statusEl = document.getElementById(`status-${phaseNum}`);
    const testsEl = document.getElementById(`tests-${phaseNum}`);

    // Update status to running
    statusEl.className = 'status-badge status-running';
    statusEl.innerHTML = '<span class="spinner"></span>RUNNING';

    try {
        // Fetch and execute the test
        const response = await fetch(phase.file);
        const html = await response.text();

        // Parse results from the HTML
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');

        // Extract test results
        const allTestTiles = doc.querySelectorAll('.test-tile');
        const total = allTestTiles.length;
        let passed = 0, failed = 0;

        allTestTiles.forEach(tile => {
            const statusElement = tile.querySelector('.test-status');
            if (statusElement) {
                if (statusElement.classList.contains('success')) {
                    passed++;
                } else if (statusElement.classList.contains('failed')) {
                    failed++;
                }
            }
        });

        // Update display
        const success = failed === 0;
        statusEl.className = `status-badge ${success ? 'status-success' : 'status-failure'}`;
        statusEl.textContent = success ? 'SUCCESS' : 'FAILURE';
        testsEl.textContent = `${passed}/${total}`;

        // Update totals and progress
        // ... (see full dashboard.php for complete implementation)

    } catch (error) {
        console.error(`Error running phase ${phaseNum}:`, error);
        statusEl.className = 'status-badge status-failure';
        statusEl.textContent = 'ERROR';
    }

    // Run next phase
    if (phaseNum < totalPhases) {
        setTimeout(() => runPhase(phaseNum + 1), 500);
    }
}
```

---

## Test Helper Functions

### Purpose

The test-helper.php file provides shared utilities for all test files.

### Required Global Variables

```php
$GLOBALS['test_results'] = [
    'total' => 0,
    'passed' => 0,
    'failed' => 0,
    'errors' => [],
    'detailed_results' => [],
    'start_time' => microtime(true)
];
```

### Core Function: testFunction()

```php
/**
 * Test function executor with result tracking
 * CONTINUES ON FAILURE - does not stop execution
 *
 * @param string $name Test name (displayed to user)
 * @param callable $callable Function to execute
 * @param string $description Optional description
 */
function testFunction($name, $callable, $description = '') {
    $totalTests = &$GLOBALS['test_results']['total'];
    $passedTests = &$GLOBALS['test_results']['passed'];
    $failedTests = &$GLOBALS['test_results']['failed'];
    $errors = &$GLOBALS['test_results']['errors'];
    $detailedResults = &$GLOBALS['test_results']['detailed_results'];

    $totalTests++;
    $testStart = microtime(true);
    $status = 'FAILED';
    $message = '';
    $exception = null;
    $result = null;

    // Display test tile
    echo "<div class='test-tile'>";
    echo "<div class='test-content'>";
    echo "<div class='test-header'>";
    echo "<span class='test-number'>#$totalTests</span>";
    echo "<span class='test-name'>$name</span>";
    echo "</div>";

    if ($description) {
        echo "<div class='test-description'>$description</div>";
    }

    try {
        $result = $callable();
        $testDuration = round((microtime(true) - $testStart) * 1000, 2);

        if ($result === false) {
            echo "<div class='test-status failed'>FAILED</div>";
            $status = 'FAILED';
            $failedTests++;
            $errors[] = $name;
        } else {
            echo "<div class='test-status success'>SUCCESS</div>";
            $status = 'SUCCESS';
            $passedTests++;
        }

        // Display result output
        if (is_string($result) || is_array($result)) {
            echo "<div class='test-output'>";
            echo "<pre>";
            echo is_array($result) ? json_encode($result, JSON_PRETTY_PRINT) : htmlspecialchars($result);
            echo "</pre>";
            echo "</div>";
            $message = is_array($result) ? json_encode($result) : $result;
        }
    } catch (Exception $e) {
        $testDuration = round((microtime(true) - $testStart) * 1000, 2);
        echo "<div class='test-status failed'>EXCEPTION</div>";
        echo "<div class='test-output'>";
        echo "<div class='exception-message'>" . htmlspecialchars($e->getMessage()) . "</div>";
        echo "<pre class='error-trace'>";
        echo htmlspecialchars($e->getTraceAsString());
        echo "</pre>";
        echo "</div>";

        $status = 'EXCEPTION';
        $message = $e->getMessage();
        $exception = [
            'message' => $e->getMessage(),
            'trace' => $e->getTraceAsString()
        ];
        $failedTests++;
        $errors[] = $name . " (Exception)";
    }

    echo "</div>"; // test-content
    echo "<div class='test-footer'>";
    echo "<span class='test-duration'>{$testDuration}ms</span>";
    echo "<span class='test-timestamp'>" . date('H:i:s') . "</span>";
    echo "</div>";
    echo "</div>"; // test-tile

    // Store detailed result for logging
    $detailedResults[] = [
        'test_number' => $totalTests,
        'test_name' => $name,
        'description' => $description,
        'status' => $status,
        'duration_ms' => $testDuration,
        'timestamp' => date('Y-m-d H:i:s'),
        'message' => $message,
        'exception' => $exception
    ];

    flush();
}
```

### Support Functions

```php
/**
 * Get test results summary
 */
function getTestResults() {
    return $GLOBALS['test_results'];
}

/**
 * Reset test counters (for fresh phase)
 */
function resetTestCounters() {
    $GLOBALS['test_results'] = [
        'total' => 0,
        'passed' => 0,
        'failed' => 0,
        'errors' => [],
        'detailed_results' => [],
        'start_time' => microtime(true)
    ];
}

/**
 * Log full test report to JSON file
 */
function logTestReport($phaseName, $phaseNumber = null) {
    $results = getTestResults();
    $duration = round(microtime(true) - $results['start_time'], 2);

    $report = [
        'phase_name' => $phaseName,
        'phase_number' => $phaseNumber,
        'timestamp' => date('Y-m-d H:i:s'),
        'summary' => [
            'total_tests' => $results['total'],
            'passed' => $results['passed'],
            'failed' => $results['failed'],
            'success_rate' => $results['total'] > 0 ? round(($results['passed'] / $results['total']) * 100, 2) : 0,
            'duration_seconds' => $duration
        ],
        'failed_tests' => $results['errors'],
        'detailed_results' => $results['detailed_results']
    ];

    // Ensure logs directory exists
    $logsDir = __DIR__ . '/../../docs';
    if (!is_dir($logsDir)) {
        mkdir($logsDir, 0777, true);
    }

    // Create filename
    $filename = $phaseNumber ? sprintf('%02d', $phaseNumber) . '-' : '';
    $filename .= preg_replace('/[^a-z0-9-]/', '-', strtolower($phaseName));
    $filename .= '-' . date('Y-m-d') . '.json';

    $logFile = $logsDir . '/' . $filename;
    file_put_contents($logFile, json_encode($report, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES));

    return $logFile;
}

/**
 * Display test summary with enhanced dark theme
 */
function displayTestSummary($phaseName, $phaseNumber = null) {
    $results = getTestResults();
    $duration = round(microtime(true) - $results['start_time'], 2);
    $successRate = $results['total'] > 0 ? round(($results['passed'] / $results['total']) * 100, 2) : 0;

    $logFile = logTestReport($phaseName, $phaseNumber);
    $logFilename = basename($logFile);

    echo "<div class='summary'>";
    echo "<h3>$phaseName - Test Results</h3>";

    echo "<div class='stats-grid'>";
    echo "<div class='stat-box'>";
    echo "<div class='stat-label'>Total Tests</div>";
    echo "<div class='stat-value'>{$results['total']}</div>";
    echo "</div>";

    echo "<div class='stat-box'>";
    echo "<div class='stat-label'>Passed</div>";
    echo "<div class='stat-value success'>{$results['passed']}</div>";
    echo "</div>";

    echo "<div class='stat-box'>";
    echo "<div class='stat-label'>Failed</div>";
    echo "<div class='stat-value failed'>{$results['failed']}</div>";
    echo "</div>";

    echo "<div class='stat-box'>";
    echo "<div class='stat-label'>Success Rate</div>";
    echo "<div class='stat-value'>{$successRate}%</div>";
    echo "</div>";

    echo "<div class='stat-box'>";
    echo "<div class='stat-label'>Duration</div>";
    echo "<div class='stat-value'>{$duration}s</div>";
    echo "</div>";
    echo "</div>";

    if (!empty($results['errors'])) {
        echo "<h4 style='color: #fca5a5; margin-top: 1rem;'>Failed Tests:</h4>";
        echo "<ul style='color: #fca5a5;'>";
        foreach ($results['errors'] as $error) {
            echo "<li>$error</li>";
        }
        echo "</ul>";
    } else {
        echo "<h4 style='color: #6ee7b7; margin-top: 1rem;'>ALL TESTS PASSED!</h4>";
    }

    echo "</div>";

    echo "<div class='log-notice'>";
    echo "<strong>Test Report Logged:</strong> /docs/$logFilename";
    echo "</div>";

    echo "<a href='../test-runner.php' class='back-btn'>Back to Test Runner</a>";
    echo "<a href='dashboard.php' class='back-btn'>View Dashboard</a>";
}
```

---

## Test File Structure

### Template for Test Phase Files

```php
<?php
/**
 * Phase {NUMBER}: {CATEGORY NAME}
 */

error_reporting(E_ALL);
ini_set('display_errors', 1);
set_time_limit(120);

require_once __DIR__ . '/test-helper.php';

echo "<html><head><title>Phase {NUMBER}: {CATEGORY NAME}</title>";
echo getTestStyles();
echo "</head><body>";

echo "<div class='container'>";
echo "<h1>Phase {NUMBER}: {CATEGORY NAME}</h1>";
echo "<div class='info'>{DESCRIPTION OF WHAT THIS PHASE TESTS}</div>";
echo "<hr>";

// ============================================================================
// PHASE {NUMBER}: {CATEGORY NAME}
// ============================================================================

testFunction("Test Name 1", function() {
    // Test implementation
    // Return false for failure
    // Return true or data for success
    return "Success message or data";
});

testFunction("Test Name 2", function() {
    // Another test
    return [
        'result_key_1' => 'value1',
        'result_key_2' => 'value2'
    ];
});

// ... more tests

// Display summary
displayTestSummary("Phase {NUMBER}: {CATEGORY NAME}", {NUMBER});

echo "</div>"; // container
echo "</body></html>";
?>
```

### Test Function Return Values

- **Success**: Return `true`, a string message, or an array of results
- **Failure**: Return `false`
- **Exception**: Automatically caught and displayed with trace

### Example Test Patterns

**Simple Boolean Test**
```php
testFunction("Check file exists", function() {
    $path = __DIR__ . '/../../config.json';
    if (!file_exists($path)) return false;
    return "File found at: $path";
});
```

**Test with Data Return**
```php
testFunction("Load configuration", function() {
    $config = json_decode(file_get_contents('config.json'), true);
    return [
        'sections' => count($config),
        'has_api_keys' => isset($config['api']),
        'has_trading_config' => isset($config['trading'])
    ];
});
```

**Test with API Call**
```php
testFunction("Fetch current price", function() {
    global $apiClient;
    $price = $apiClient->getCurrentPrice('BTCUSD');
    if (!$price) return false;
    return [
        'symbol' => 'BTCUSD',
        'price' => $price,
        'timestamp' => date('Y-m-d H:i:s')
    ];
});
```

**Test with Calculation**
```php
testFunction("Calculate position size", function() {
    global $riskManager;
    $size = $riskManager->calculatePositionSize(50000, 1.5, 15, 'Buy');
    return [
        'entry_price' => 50000,
        'sl_percent' => 1.5,
        'risk_amount' => 15,
        'position_size' => $size
    ];
});
```

---

## Styling & UI Guidelines

### Color Scheme

```css
/* Dark Theme Palette */
--bg-primary: #0f172a      /* Main background */
--bg-secondary: #1e293b    /* Cards, tiles */
--bg-tertiary: #334155     /* Headers, hover states */

--text-primary: #f1f5f9    /* Main text */
--text-secondary: #e2e8f0  /* Body text */
--text-muted: #94a3b8      /* Labels, meta */

--accent-primary: #3b82f6  /* Primary blue */
--accent-success: #10b981  /* Success green */
--accent-danger: #ef4444   /* Error red */
--accent-warning: #f59e0b  /* Warning orange */

--border-color: #334155    /* Borders */
```

### Typography

- **Font Family**: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
- **Font Weights**: 300, 400, 500, 600, 700
- **Line Height**: 1.6
- **Code Font**: 'Courier New', monospace

### Component Styles

**Test Tile**
```css
.test-tile {
    background: #1e293b;
    border: 1px solid #334155;
    border-radius: 0.5rem;
    margin: 0.75rem 0;
    overflow: hidden;
    transition: all 0.2s ease;
}

.test-tile:hover {
    border-color: #3b82f6;
    box-shadow: 0 4px 12px rgba(59, 130, 246, 0.1);
}
```

**Status Badges**
```css
.status-badge {
    display: inline-block;
    padding: 0.375rem 0.875rem;
    border-radius: 0.25rem;
    font-weight: 700;
    font-size: 0.875rem;
    letter-spacing: 0.05em;
}

.status-success {
    background: #065f46;
    color: #6ee7b7;
    border: 1px solid #10b981;
}

.status-failure {
    background: #7f1d1d;
    color: #fca5a5;
    border: 1px solid #ef4444;
}

.status-running {
    background: #1e40af;
    color: #93c5fd;
    border: 1px solid #3b82f6;
    animation: pulse 2s infinite;
}
```

**Buttons**
```css
.btn {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    color: white;
    text-decoration: none;
    border-radius: 0.375rem;
    font-weight: 600;
    font-size: 1rem;
    transition: all 0.2s ease;
    border: 1px solid #059669;
    cursor: pointer;
}

.btn:hover {
    background: linear-gradient(135deg, #059669 0%, #047857 100%);
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
    transform: translateY(-1px);
}
```

---

## Test Glossary Documentation

### Purpose

The test-glossary.json file provides comprehensive documentation of all tests.

### Structure

```json
{
  "test_suite": {
    "name": "Application Test Suite Name",
    "version": "1.0",
    "total_phases": 12,
    "total_tests": 60,
    "description": "Comprehensive testing suite for functionality validation"
  },
  "phases": [
    {
      "phase": 1,
      "name": "File System & Path Checks",
      "file": "01-filesystem-paths.php",
      "category": "File System",
      "tests": [
        {
          "number": 1,
          "name": "Check helpers.php exists",
          "description": "Verifies that the helpers.php file exists and can be loaded",
          "expected_result": "File loads successfully"
        },
        {
          "number": 2,
          "name": "Test PathHelper::buildPath()",
          "description": "Tests the buildPath() method to ensure it returns valid directory paths",
          "expected_result": "Returns valid data directory path"
        }
      ]
    }
  ],
  "categories": {
    "File System": {
      "tests": 6,
      "description": "Tests for file operations and path management"
    },
    "Configuration": {
      "tests": 7,
      "description": "Tests for configuration loading and validation"
    }
  },
  "statistics": {
    "total_phases": 12,
    "total_tests": 60,
    "total_categories": 12,
    "success_rate": "100%",
    "status": "All tests passing"
  }
}
```

### Glossary Fields

- **test_suite**: Overall test suite metadata
- **phases**: Array of all test phases
  - **phase**: Phase number
  - **name**: Human-readable phase name
  - **file**: Test file name
  - **category**: Logical category
  - **tests**: Array of individual tests
    - **number**: Test number within phase
    - **name**: Test name (matches testFunction() call)
    - **description**: Detailed description of what is tested
    - **expected_result**: What a successful test returns
- **categories**: Summary of test categories
- **statistics**: Overall statistics

---

## Best Practices

### 1. Test Naming

- Use descriptive, action-oriented names
- Start with verbs: "Check", "Load", "Test", "Calculate", "Validate"
- Be specific: "Get account balance (UNIFIED)" instead of "Test balance"

### 2. Test Organization

- Group related tests together in phases
- Order tests by dependency (foundational tests first)
- Keep phases focused (5-10 tests per phase)
- Use clear phase numbering (01, 02, 03...)

### 3. Error Handling

- Always use try-catch in testFunction
- Return false for expected failures
- Let exceptions bubble up for unexpected errors
- Log all errors with context

### 4. Test Isolation

- Each test should be independent
- Clean up resources (temp files, test data)
- Don't rely on previous test state
- Use global variables sparingly

### 5. Output Quality

- Return structured data (arrays) when possible
- Include relevant context in outputs
- Format numbers appropriately
- Use consistent units (USD, percentage, etc.)

### 6. Performance

- Set appropriate timeouts (120s for most, 600s for dashboard)
- Use flush() to show progress in real-time
- Keep individual tests under 30 seconds
- Cache API results when possible

### 7. Documentation

- Document every test in glossary
- Include expected results
- Explain what constitutes success/failure
- Add inline comments for complex logic

### 8. Maintenance

- Update glossary when adding tests
- Keep test numbers sequential
- Version control all test files
- Archive old test results

---

## Example Workflow

### For AI Assistants Creating Test Suites

**Step 1: Analyze Codebase**

```
Task: Analyze the entire codebase and identify all functions and classes.

Actions:
1. Scan all PHP files in the project
2. Extract all class definitions and methods
3. Extract all standalone functions
4. Identify dependencies between components
5. Group functions into logical categories
```

**Step 2: Plan Test Phases**

```
Task: Create a test phase plan based on dependencies.

Output Format:
Phase 01: File System & Paths (6 tests)
  - Test 1: Check helpers.php exists
  - Test 2: Test PathHelper::buildPath()
  - ...

Phase 02: Configuration Loading (7 tests)
  - Test 1: Load active config
  - Test 2: Validate API configuration
  - ...
```

**Step 3: Create Test Files**

```
Task: For each phase, create a test file with all tests.

Template:
- Use standard test file structure
- Include all required tests
- Add descriptive names and comments
- Implement testFunction() for each test
```

**Step 4: Create Dashboard**

```
Task: Create dashboard.php with all phases listed.

Include:
- All phase definitions
- JavaScript for sequential execution
- Progress tracking
- Result display
```

**Step 5: Create Test Helper**

```
Task: Create test-helper.php with all utility functions.

Include:
- testFunction()
- getTestResults()
- displayTestSummary()
- logTestReport()
- getTestStyles()
```

**Step 6: Create Glossary**

```
Task: Document all tests in test-glossary.json.

Include:
- All phases and tests
- Descriptions and expected results
- Categories and statistics
```

**Step 7: Verify**

```
Task: Run the complete test suite and verify all tests execute.

Checks:
- Dashboard loads correctly
- All phases execute sequentially
- Results are displayed properly
- Logs are created
- No syntax errors
```

---

## Advanced Features

### 1. Test Filtering

Allow running specific phases or categories:

```php
$selectedPhases = $_GET['phases'] ?? 'all';
if ($selectedPhases !== 'all') {
    $phasesToRun = explode(',', $selectedPhases);
    // Filter $testPhases array
}
```

### 2. Parallel Test Execution

For independent tests, consider parallel execution:

```javascript
// Run multiple phases in parallel
await Promise.all([
    runPhase(1),
    runPhase(2),
    runPhase(3)
]);
```

### 3. Test Retries

Add automatic retry logic for flaky tests:

```php
function testFunctionWithRetry($name, $callable, $description = '', $maxRetries = 3) {
    for ($attempt = 1; $attempt <= $maxRetries; $attempt++) {
        $result = $callable();
        if ($result !== false) {
            return testFunction($name, function() use ($result) {
                return $result;
            }, $description);
        }
        sleep(1); // Wait before retry
    }
    return testFunction($name, function() { return false; }, $description);
}
```

### 4. Test Data Generation

Create test data generators:

```php
function generateTestData($type) {
    switch ($type) {
        case 'balance':
            return rand(100, 10000) / 100;
        case 'price':
            return rand(30000, 70000);
        case 'timestamp':
            return time();
    }
}
```

### 5. Screenshot Capture

For visual tests, capture screenshots:

```php
testFunction("Capture dashboard screenshot", function() {
    $url = 'http://localhost/test-suite/dashboard.php';
    $screenshotPath = __DIR__ . '/../../docs/screenshots/dashboard.png';
    // Use Playwright or similar tool to capture
    return "Screenshot saved to: $screenshotPath";
});
```

---

## Troubleshooting

### Common Issues

**Issue 1: Tests timing out**
```
Solution: Increase set_time_limit() value
set_time_limit(600); // 10 minutes
```

**Issue 2: Dashboard not updating status**
```
Solution: Check CORS settings and network tab
- Ensure test files are accessible
- Check for JavaScript errors
- Verify fetch() permissions
```

**Issue 3: Test results not being logged**
```
Solution: Check directory permissions
chmod 777 /docs
```

**Issue 4: Tests failing with "Class not found"**
```
Solution: Check require_once paths
require_once __DIR__ . '/../path/to/class.php';
```

---

## Conclusion

This guideline provides a comprehensive framework for creating systematic, visually appealing, and maintainable test suites for PHP applications. By following these standards, you ensure:

- **Complete Coverage**: All functions are tested
- **Maintainability**: Tests are organized and documented
- **User Experience**: Clean, professional UI
- **Reliability**: Tests are isolated and repeatable
- **Visibility**: Real-time progress and detailed logging

Use this document as a reference when creating new test suites or expanding existing ones.

---

**Document Version:** 1.0
**Last Updated:** 2025-01-04
**Maintained By:** Development Team

# GSoC'25 - Keploy Performance & Security Features

This repository showcases my Google Summer of Code 2025 contributions to the Keploy project, where I implemented three major features: **TestSuite Execution**, **Load Testing**, and **Security Checking**.

## 🎯 Project Overview

As a GSoC'25 contributor to Keploy, I developed a comprehensive testing solution that enhances Keploy's capabilities with advanced performance and security testing features. These features work together to provide developers with a complete API testing toolkit.

### 🔗 Key Links

- **Main PR**: [keploy/keploy#2736](https://github.com/keploy/keploy/pull/2736)
- **Load Testing Dashboard**: [KLT_Dashboard Repository](https://github.com/AHMED-D007A/KLT_Dashboard)

### Original Proposal vs Final Implementation

**Initial Project**: TestSuite Idempotency Checker  
**Final Project**: Performance & Security Testing Suite

During the course of GSoC'25, my project underwent a significant transformation that better aligned with Keploy's evolving needs and community requirements:

**📋 Original Scope (TestSuite Idempotency Checker):**
- Focus on detecting non-idempotent API behaviors
- Verification of API state consistency across multiple calls
- Identification of side effects in supposedly safe operations

**🎯 Evolved Scope (Performance & Security Testing):**
- Comprehensive testing solution covering functional, performance, and security aspects
- Response to actual user needs for load testing and security scanning capabilities

**Why the Change?**
- Active discussions with mentors and community revealed higher priority needs for performance and security testing

## 🚀 Features Implemented

### 1. TestSuite Execution Engine
A declarative YAML-based system for defining and executing API test sequences.

**Key Capabilities:**
- ✅ Variable extraction and interpolation between test steps
- ✅ HTTP request/response validation and assertions
- ✅ Rate limiting for controlled execution
- ✅ Comprehensive execution reporting

**Usage:**
```bash
keploy testsuite --base-url "http://localhost:8080"
```

### 2. Load Testing
Performance testing capability that executes test suites under various load profiles.

**Key Features:**
- 🔥 **Built-in Dashboard**: Embedded React dashboard that auto-launches in browser
- 📊 **Real-time Monitoring**: Live metrics for VU count, RPS, response times
- 🎯 **Multiple Load Profiles**: Constant and ramping virtual user patterns
- 📈 **Threshold-based Evaluation**: Automated pass/fail criteria
- 🚀 **Zero Configuration**: No external dependencies required

**Usage:**
```bash
# Automatically opens dashboard at http://localhost:3000
keploy load --base-url "http://localhost:8080" --vus 20 --duration 10m
```

### 3. Security Checking System
Automated security vulnerability scanning for API endpoints.

**Security Checks Include:**
- 🔒 Missing security headers (HSTS, CSP, X-Frame-Options)
- 🛡️ Sensitive data exposure detection
- 🍪 Cookie security validation
- 🔧 Custom security checks via YAML configuration
- 📋 Severity-based filtering and allowlists

**Usage:**
```bash
keploy secure --base-url "http://localhost:8080"
```

## 🏗️ Architecture Overview

All three features are built on a unified architecture:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   TestSuite     │    │   Load Testing  │    │   Security      │
│   Execution     │    │                 │    │   Checking      │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • HTTP Executor │    │ • VU Scheduler  │    │ • Rule Engine   │
│ • Variable Mgmt │    │ • Metrics       │    │ • Custom Checks │
│ • Assertions    │    │ • Dashboard     │    │ • Allowlists    │
│ • Rate Limiting │    │ • Thresholds    │    │ • Severity      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                        │                        │
         └────────────────────────┼────────────────────────┘
                                  │
                    ┌─────────────────────────┐
                    │    Service Factory      │
                    │   (Dependency Injection)│
                    └─────────────────────────┘
```

## 📝 Configuration Example

Here's a complete test suite configuration showcasing all features:

```yaml
# suite-0.yaml
version: api.keploy.io/v2beta1
kind: TestSuite
name: Todo_CRUD_Operations
spec:
  metadata:
    description: Test CRUD operations for Todo API
  
  # Security Configuration
  security:
    ruleset: basic
    severity_threshold: MEDIUM
    allowlist:
      headers: ["Server"]
      keys: ["data.debug"]
  
  # Load Testing Configuration
  load:
    profile: ramping_vus
    vus: 30
    duration: 3m
    stages:
      - duration: 30s
        target: 10
      - duration: 2m
        target: 30
    thresholds:
      - metric: http_req_duration_p95
        condition: "< 500ms"
        severity: high
      - metric: http_req_failed_rate
        condition: "<= 1%"
        severity: critical
  
  # Test Steps
  steps:
    - name: Create_todo
      method: POST
      url: /todos
      headers:
        Content-Type: application/json
      body: |
        {
          "title": "Test Todo",
          "completed": false
        }
      extract:
        todo_id: id
      assert:
        - type: status_code
          expected_string: "201"
    
    - name: Get_todo
      method: GET
      url: /todos/{{todo_id}}
      assert:
        - type: status_code
          expected_string: "200"
```

## 🎨 Load Testing Dashboard

One of the standout features is the **integrated load testing dashboard**:

### Dashboard Highlights:
- **Embedded Server**: Runs directly within Keploy binary
- **Auto-Launch**: Automatically opens browser to `http://localhost:3000`
- **Real-time Updates**: Live metrics streaming

## 🔧 Technical Implementation

### Service Factory Pattern
All features use a unified dependency injection system:

```go
func (n *ServiceProvider) GetService(ctx context.Context, cmd string) (interface{}, error) {
    switch cmd {
    case "testsuite":
        return testsuite.NewTSExecutor(n.cfg, n.logger, false)
    case "load":
        return load.NewLoadTester(n.cfg, n.logger)
    case "secure":
        return secure.NewSecurityChecker(n.cfg, n.logger)
    }
}
```

### Key Components:
- **TSExecutor**: Core test suite execution engine
- **LoadTester**: Performance testing coordinator
- **SecurityChecker**: Vulnerability scanning engine
- **MetricsCollector**: Real-time metrics aggregation
- **DashboardExposer**: Embedded web server for dashboard

## 📊 Metrics and Reporting

The system provides comprehensive metrics collection:

### Load Testing Metrics:
- Request count and failure rates
- Response time percentiles (P90, P95, P99)
- Throughput (requests per second)
- Data transfer (bytes sent/received)
- Virtual user count over time

### Security Metrics:
- Vulnerability count by severity
- Check pass/fail status
- Custom rule evaluation results
- Allowlist filtering statistics

## 🎯 Impact and Benefits

### For Developers:
- **Unified Testing**: Single tool for functional, performance, and security testing
- **Zero Setup**: Built-in dashboard with no external dependencies
- **Real-time Feedback**: Live monitoring during load tests
- **Customizable**: Flexible configuration for different testing scenarios

### For DevOps Teams:
- **CI/CD Integration**: Command-line interface suitable for automation
- **Threshold-based Evaluation**: Automated pass/fail criteria
- **Comprehensive Reports**: JSON and CLI output formats
- **Security Compliance**: Built-in security vulnerability scanning

## 🛠️ Installation and Usage

1. **Install Keploy**

2. **Create a Test Suite**:
   ```bash
   mkdir -p keploy/testsuite
   # Add your suite-0.yaml file
   ```

3. **Run Tests**:
   ```bash
   # Functional testing
   keploy testsuite --base-url "http://localhost:8080"
   
   # Load testing
   keploy load --base-url "http://localhost:8080"
   
   # Security scanning
   keploy secure --base-url "http://localhost:8080"
   ```

## 📚 Documentation

For detailed technical documentation, see the [complete documentation](./docs/TECHNICAL_DOCUMENTATION.md) included in this repository.

## 🙏 Acknowledgments

Special thanks to the Keploy team and my mentors for their guidance throughout GSoC'25. This project wouldn't have been possible without their support and the amazing open-source community.

## 📧 Contact

- **GitHub**: [@AHMED-D007A](https://github.com/AHMED-D007A)
- **Project**: [Keploy](https://github.com/keploy/keploy)
- **GSoC**: Google Summer of Code 2025

---

*This project is part of Google Summer of Code 2025 under the Keploy organization.*

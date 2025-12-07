# Day 6: Code Quality & Security with SonarQube

## Table of Contents
1. [Introduction to Code Quality](#introduction-to-code-quality)
2. [Understanding SonarQube](#understanding-sonarqube)
3. [SonarQube Architecture & Components](#sonarqube-architecture--components)
4. [Installation & Setup](#installation--setup)
5. [Integrating SonarQube with Jenkins](#integrating-sonarqube-with-jenkins)
6. [Code Analysis for Different Languages](#code-analysis-for-different-languages)
7. [Quality Gates & Metrics](#quality-gates--metrics)
8. [Real-World Use Cases](#real-world-use-cases)
9. [Production Setup with SSL](#production-setup-with-ssl)
10. [Best Practices & Lessons Learned](#best-practices--lessons-learned)

---

## Introduction to Code Quality

### Why Code Quality Matters

As a DevOps engineer, I've learned that **catching bugs early saves exponentially more time and money** than fixing them in production. Code quality isn't just about clean code—it's about:

- **Security**: Preventing vulnerabilities before deployment
- **Maintainability**: Making code understandable for future developers
- **Reliability**: Reducing production incidents
- **Cost Efficiency**: Fixing issues early in the development cycle
- **Team Productivity**: Clear, well-tested code speeds up development

### The Testing Pyramid

Understanding where SonarQube fits in the testing strategy:

```
        /\
       /  \  E2E Tests (Manual/Automated)
      /----\
     /      \  Integration Tests
    /--------\
   /          \  Unit Tests
  /--------------\
 /    Static      \  <- SonarQube operates here
/  Code Analysis   \
--------------------
```

**Key Insight**: Static analysis is the **first line of defense** in testing. It catches issues before code even runs.

### SAST vs DAST

After working with both, here's what I've learned:

| Aspect | SAST (Static) | DAST (Dynamic) |
|--------|---------------|----------------|
| **When** | Code is still in repository | Application is running/deployed |
| **What** | Source code analysis | Runtime behavior analysis |
| **Tools** | SonarQube, Checkmarx | OWASP ZAP, Burp Suite |
| **Finds** | Code smells, bugs, vulnerabilities | Runtime vulnerabilities, pen testing |
| **Speed** | Fast (minutes) | Slower (hours) |
| **Coverage** | 100% of code | Only accessible paths |

**My Approach**: Use SAST (SonarQube) in CI/CD pipeline for every commit, and DAST (OWASP ZAP) before production releases.

---

## Understanding SonarQube

### What is SonarQube?

**SonarQube** is an open-source platform for **continuous inspection of code quality**. It performs automatic reviews with static analysis to detect:

- 🐛 **Bugs**: Code that causes incorrect behavior
- 🔒 **Vulnerabilities**: Security risks (SQL injection, XSS, hardcoded credentials)
- 💨 **Code Smells**: Maintainability issues (long methods, complex logic)
- 📋 **Duplications**: Repeated code blocks
- 📊 **Code Coverage**: Percentage of code covered by tests
- ⏱️ **Technical Debt**: Estimated time to fix all issues

### The Problem SonarQube Solves

**Before SonarQube:**
```
Developer writes code → Push to repo → Code review (manual) → 
Merge → Build → Deploy → Bug found in production → Hotfix → Repeat
```

**After SonarQube:**
```
Developer writes code → Push to repo → SonarQube analysis (automated) → 
Issues flagged immediately → Fix before merge → Deploy clean code
```

**Result**: Issues caught at commit time, not in production.

### What SonarQube Checks

#### 1. Bugs
**Definition**: Code written incorrectly that causes runtime errors or unexpected behavior.

**Example:**
```java
// Bug: Null pointer exception
String name = null;
System.out.println(name.length()); // Will crash
```

**SonarQube flags this**: "Possible NullPointerException"

#### 2. Vulnerabilities
**Definition**: Security risks that could be exploited.

**Example:**
```java
// Vulnerability: SQL Injection
String query = "SELECT * FROM users WHERE username = '" + userInput + "'";
```

**SonarQube flags this**: "SQL injection vulnerability detected"

**Fix:**
```java
// Use prepared statements
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE username = ?");
stmt.setString(1, userInput);
```

#### 3. Code Smells
**Definition**: Issues that don't immediately break functionality but hurt maintainability.

**Example:**
```java
// Code smell: Method too long (100+ lines)
public void processOrder() {
    // 100 lines of code doing everything
    // Hard to understand and test
}
```

**SonarQube suggests**: Break into smaller, focused methods.

#### 4. Duplications
**Definition**: Same code repeated in multiple places.

**Example:**
```java
// Duplicated in 3 different classes
public double calculateDiscount(double price) {
    return price * 0.1;
}
```

**Fix**: Create a shared utility class.

#### 5. Code Coverage
**Definition**: Percentage of code executed by unit tests.

**Example:**
```
Total Lines: 1000
Lines Covered by Tests: 800
Code Coverage: 80%
```

**Industry Standard**: Aim for 80%+ coverage for critical business logic.

#### 6. Technical Debt
**Definition**: Estimated time required to fix all code quality issues.

**Example:**
```
Critical Issues: 5 (10 hours)
Major Issues: 20 (15 hours)
Minor Issues: 50 (10 hours)
Total Technical Debt: 35 hours
```

---

## SonarQube Architecture & Components

### High-Level Architecture

```
┌─────────────────────────────────────────────────┐
│              Jenkins Pipeline                    │
│  (Triggers analysis on every commit)            │
└─────────────────┬───────────────────────────────┘
                  │
                  ├─── Maven/Gradle builds code
                  │
                  ├─── JaCoCo generates coverage report
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│           SonarQube Scanner                      │
│  (Analyzes source code & test results)          │
└─────────────────┬───────────────────────────────┘
                  │
                  │ Sends analysis data
                  ▼
┌─────────────────────────────────────────────────┐
│          SonarQube Server                        │
│  ┌──────────────────────────────────────┐      │
│  │  Compute Engine                       │      │
│  │  (Processes analysis reports)         │      │
│  └──────────────┬───────────────────────┘      │
│                 │                                │
│                 ▼                                │
│  ┌──────────────────────────────────────┐      │
│  │  PostgreSQL Database                  │      │
│  │  (Stores metrics, issues, history)    │      │
│  └──────────────┬───────────────────────┘      │
│                 │                                │
│                 ▼                                │
│  ┌──────────────────────────────────────┐      │
│  │  Web Server (Port 9000)               │      │
│  │  (Dashboard, reports, configuration)  │      │
│  └──────────────────────────────────────┘      │
└─────────────────────────────────────────────────┘
                  │
                  │ Webhook callback
                  ▼
┌─────────────────────────────────────────────────┐
│              Jenkins                             │
│  (Receives Quality Gate status)                 │
└─────────────────────────────────────────────────┘
```

### Component Breakdown

#### 1. SonarQube Server
- **Web Server**: Provides UI for viewing reports and managing configuration
- **Compute Engine**: Processes analysis reports asynchronously
- **Elasticsearch**: Search engine for fast queries (embedded in SonarQube)

#### 2. Database (PostgreSQL)
- Stores project metrics
- Historical data for trend analysis
- Quality profiles and rules
- User permissions and settings

**Why PostgreSQL?**
- Required for production deployments
- H2 database (embedded) only for testing
- Better performance and scalability

#### 3. SonarQube Scanner
- Analyzes source code
- Collects metrics
- Sends data to SonarQube server
- Language-specific scanners available

#### 4. Quality Profiles
- Set of rules applied to code analysis
- Different profiles for different languages
- Can be customized per project

**Example Quality Profile Rules:**
- Maximum method length: 50 lines
- Maximum cyclomatic complexity: 10
- Minimum code coverage: 80%
- No hardcoded credentials

### Two-Way Communication Flow

**Jenkins → SonarQube:**
1. Jenkins triggers build
2. Scanner analyzes code
3. Results sent to SonarQube
4. SonarQube processes and stores data

**SonarQube → Jenkins:**
1. Quality Gate evaluation completes
2. Webhook notifies Jenkins
3. Jenkins receives pass/fail status
4. Pipeline continues or aborts

---

## Installation & Setup

### Method 1: Docker Installation (Fastest for Learning)

This is how I started learning SonarQube—quickest way to get hands-on experience.

#### Prerequisites
- Docker installed
- At least 4 GB RAM available
- Ports 9000 and 9092 available

#### Installation Steps

**Step 1: Run SonarQube Container**

```bash
# Pull and run SonarQube (Community Edition)
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:lts-community
```

**Step 2: Access SonarQube**

```bash
# Check if container is running
docker ps

# Access SonarQube
# Open browser: http://localhost:9000
# Default credentials: admin / admin
```

**Step 3: First Login**

1. Navigate to `http://localhost:9000`
2. Login with: `admin` / `admin`
3. You'll be prompted to change the password
4. Set new password (remember this!)

**Why Docker for Learning?**
- ✅ No complex setup
- ✅ Clean environment
- ✅ Easy to reset (just recreate container)
- ✅ Portable across machines

**Common Docker Issue I Faced:**

```bash
# Problem: Permission denied error
Error: docker: Got permission denied while trying to connect to the Docker daemon socket

# Solution: Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker ps
```

### Method 2: Production Setup on Ubuntu (Linux)

After learning with Docker, I moved to production-grade setup. Here's what I learned.

#### Prerequisites

```bash
# System Requirements
- Ubuntu 22.04 (or similar)
- Minimum 4 GB RAM (8 GB recommended)
- 2 CPU cores
- 30 GB free disk space
```

#### Step-by-Step Installation

**Step 1: Update System**

```bash
sudo apt update
sudo apt upgrade -y
```

**Step 2: Install Java 17**

SonarQube requires Java. I use Java 17 (LTS version).

```bash
# Install OpenJDK 17
sudo apt install openjdk-17-jdk -y

# Verify installation
java -version
```

**Expected output:**
```
openjdk version "17.0.x" 2024-xx-xx
OpenJDK Runtime Environment (build 17.0.x)
OpenJDK 64-Bit Server VM (build 17.0.x)
```

**Step 3: Install PostgreSQL 15**

SonarQube requires a database for production use.

```bash
# Install dependencies
sudo apt install curl ca-certificates -y

# Add PostgreSQL GPG key
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail \
  https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Add PostgreSQL repository
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] \
  https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > \
  /etc/apt/sources.list.d/pgdg.list'

# Install PostgreSQL 15
sudo apt update
sudo apt install postgresql-15 -y

# Verify PostgreSQL is running
sudo systemctl status postgresql
```

**Step 4: Configure PostgreSQL Database**

Create a dedicated database and user for SonarQube.

```bash
# Switch to postgres user
sudo -i -u postgres

# Create SonarQube user
createuser sonar

# Create SonarQube database
createdb sonar -O sonar

# Set password for sonar user
psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'YourStrongPassword123';
\q

# Exit postgres user
exit
```

**Important**: Replace `'YourStrongPassword123'` with a strong password. Save it securely!

**Step 5: Download and Install SonarQube**

```bash
# Download SonarQube (Community Edition)
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip

# Install unzip if not available
sudo apt install unzip -y

# Extract SonarQube
sudo unzip sonarqube-10.5.1.90531.zip

# Rename for easier access
sudo mv sonarqube-10.5.1.90531 sonarqube
```

**Step 6: Create SonarQube System User**

Security best practice: Don't run SonarQube as root.

```bash
# Create sonarqube user (no login, system user)
sudo adduser --system --no-create-home --group --disabled-login sonarqube

# Change ownership
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

**Step 7: Configure SonarQube**

Edit SonarQube configuration to connect to PostgreSQL.

```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

**Uncomment and configure these lines:**

```properties
# Database Configuration
sonar.jdbc.username=sonar
sonar.jdbc.password=YourStrongPassword123
sonar.jdbc.url=jdbc:postgresql://localhost/sonar

# Web Server Configuration
sonar.web.host=0.0.0.0
sonar.web.port=9000

# Elasticsearch storage path
sonar.path.data=/opt/sonarqube/data
sonar.path.temp=/opt/sonarqube/temp
```

**Key Configuration Points:**
- `sonar.jdbc.url`: Database connection string
- `sonar.web.host`: `0.0.0.0` allows external access
- `sonar.web.port`: Default is 9000

**Step 8: Configure System Limits**

SonarQube requires specific system limits to run properly.

```bash
# Edit limits configuration
sudo nano /etc/security/limits.conf
```

**Add these lines at the end:**

```
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```

**Configure kernel parameters:**

```bash
# Set vm.max_map_count
sudo sysctl -w vm.max_map_count=262144

# Make it persistent
sudo nano /etc/sysctl.conf
```

**Add this line:**
```
vm.max_map_count=262144
```

**Apply changes:**
```bash
sudo sysctl -p
```

**Why these limits?**
- `nofile`: SonarQube opens many files simultaneously
- `nproc`: Elasticsearch (embedded) needs multiple processes
- `vm.max_map_count`: Elasticsearch memory mapping requirement

**Step 9: Create Systemd Service**

Automate SonarQube startup and management.

```bash
sudo nano /etc/systemd/system/sonarqube.service
```

**Add this configuration:**

```ini
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

**Step 10: Start SonarQube Service**

```bash
# Reload systemd
sudo systemctl daemon-reload

# Start SonarQube
sudo systemctl start sonarqube

# Enable on boot
sudo systemctl enable sonarqube

# Check status
sudo systemctl status sonarqube
```

**Expected output:**
```
● sonarqube.service - SonarQube service
     Loaded: loaded (/etc/systemd/system/sonarqube.service; enabled)
     Active: active (running) since Wed 2024-12-04 10:00:00 UTC
```

**Check logs if something goes wrong:**
```bash
sudo tail -f /opt/sonarqube/logs/sonar.log
sudo tail -f /opt/sonarqube/logs/web.log
sudo tail -f /opt/sonarqube/logs/ce.log
```

**Step 11: Configure Firewall**

```bash
# Allow SonarQube port
sudo ufw allow 9000/tcp

# Check firewall status
sudo ufw status
```

**Step 12: Access SonarQube**

1. Open browser: `http://<your-server-ip>:9000`
2. Login: `admin` / `admin`
3. Change default password (forced on first login)

**Verification Checklist:**
- ✅ PostgreSQL running: `sudo systemctl status postgresql`
- ✅ SonarQube service running: `sudo systemctl status sonarqube`
- ✅ Port 9000 accessible: `sudo netstat -tulpn | grep 9000`
- ✅ Web UI accessible: `http://your-ip:9000`

### Common Installation Issues I Faced

#### Issue 1: SonarQube Won't Start

```bash
# Check logs
sudo tail -100 /opt/sonarqube/logs/sonar.log

# Common cause: Insufficient memory
# Solution: Increase RAM or adjust Java heap size
sudo nano /opt/sonarqube/conf/sonar.properties

# Add:
sonar.web.javaOpts=-Xmx512m -Xms128m
sonar.ce.javaOpts=-Xmx512m -Xms128m
```

#### Issue 2: Database Connection Failed

```bash
# Test database connection
psql -h localhost -U sonar -d sonar

# If connection fails, check PostgreSQL configuration
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Ensure this line exists:
local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
```

#### Issue 3: Port Already in Use

```bash
# Check what's using port 9000
sudo lsof -i :9000

# Kill the process or change SonarQube port
sudo nano /opt/sonarqube/conf/sonar.properties
# Change: sonar.web.port=9001
```

---

## Integrating SonarQube with Jenkins

This integration is crucial—it automates code quality checks in your CI/CD pipeline.

### Architecture Overview

```
Developer Push → GitHub → Jenkins Webhook Trigger → 
Jenkins Pipeline → Maven Build → JaCoCo Coverage → 
SonarQube Scanner → SonarQube Server Analysis → 
Quality Gate Check → Pass/Fail → Continue/Abort Pipeline
```

### Prerequisites

1. ✅ Jenkins installed and running
2. ✅ SonarQube installed and accessible
3. ✅ Maven/Gradle project with tests
4. ✅ JaCoCo plugin for code coverage

### Step-by-Step Integration

#### Step 1: Install Jenkins Plugins

Navigate to: **Jenkins Dashboard** → **Manage Jenkins** → **Manage Plugins** → **Available**

Install these plugins:
- **SonarQube Scanner for Jenkins**
- **Sonar Quality Gates Plugin**

**Restart Jenkins after installation:**
```bash
sudo systemctl restart jenkins
```

#### Step 2: Configure SonarQube Scanner in Jenkins

**Navigate to:** Manage Jenkins → Global Tool Configuration → SonarQube Scanner

**Add SonarQube Scanner:**
```
Name: sonar-scanner
☑ Install automatically
Version: SonarQube Scanner 5.0.1.3006 (or latest)
```

**Click:** Save

#### Step 3: Generate SonarQube Token

**On SonarQube Server:**

1. Login to SonarQube (`http://your-sonarqube:9000`)
2. Click on **User Icon** (top right) → **My Account**
3. Go to **Security** tab
4. Under **Tokens**, enter:
   - Name: `jenkins-token`
   - Type: Global Analysis Token
   - Expires in: No expiration (or set appropriate time)
5. Click **Generate**
6. **Copy the token immediately** (you won't see it again!)

**Example token:** `squ_8f441f761b16f4c440a470533eb1a91481d5d7a6`

#### Step 4: Add SonarQube Token to Jenkins

**Navigate to:** Manage Jenkins → Manage Credentials → (global) → Add Credentials

**Configure:**
```
Kind: Secret text
Scope: Global
Secret: <paste-your-sonarqube-token>
ID: sonar-token
Description: SonarQube authentication token
```

**Click:** Create

#### Step 5: Configure SonarQube Server in Jenkins

**Navigate to:** Manage Jenkins → Configure System → SonarQube servers

**Add SonarQube:**
```
☑ Enable injection of SonarQube server configuration

Name: sonar-server
Server URL: http://<your-sonarqube-ip>:9000
Server authentication token: Select 'sonar-token' from dropdown
```

**Click:** Save

**Verification:**
- Test connection (should show green checkmark)
- If red X, check firewall and SonarQube accessibility

#### Step 6: Configure Webhook in SonarQube

This enables SonarQube to send Quality Gate results back to Jenkins.

**On SonarQube Server:**

1. Go to **Administration** → **Configuration** → **Webhooks**
2. Click **Create**
3. Configure:
   ```
   Name: Jenkins
   URL: http://<jenkins-ip>:8080/sonarqube-webhook/
   Secret: (leave empty for now)
   ```
4. Click **Create**

**Important**: The URL must end with `/sonarqube-webhook/`

#### Step 7: Prepare Maven Project for Code Coverage

Code coverage is essential for meaningful SonarQube analysis.

**Edit `pom.xml` and add JaCoCo plugin:**

```xml
<properties>
    <java.version>17</java.version>
    <jacoco.version>0.8.8</jacoco.version>
</properties>

<build>
    <plugins>
        <!-- JaCoCo Maven Plugin for Code Coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>${jacoco.version}</version>
            <executions>
                <!-- Prepares JaCoCo agent for test execution -->
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <!-- Generates coverage report after tests -->
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**What this does:**
1. **prepare-agent**: Attaches JaCoCo agent to JVM during test execution
2. **report**: Generates XML/HTML coverage reports after tests run

**JaCoCo generates:** `target/site/jacoco/jacoco.xml` (used by SonarQube)

---

## Code Analysis for Different Languages

### Java Project Analysis

Here's my complete working Jenkins pipeline for Java applications.

#### Complete Jenkins Pipeline for Java

```groovy
pipeline {
    agent any
    
    tools {
        maven 'maven3'    // Name from Global Tool Configuration
        jdk 'jdk17'       // Name from Global Tool Configuration
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // SonarQube Scanner tool
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/your-repo/java-app.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=MyJavaApp \
                        -Dsonar.projectKey=MyJavaApp \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.junit.reportPaths=target/surefire-reports \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                    '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded! Code quality checks passed.'
        }
        failure {
            echo 'Pipeline failed! Check SonarQube report for issues.'
        }
    }
}
```

#### Understanding the Pipeline Parameters

**SonarQube Scanner Parameters Explained:**

| Parameter | Purpose | Example Value |
|-----------|---------|---------------|
| `-Dsonar.projectName` | Display name in SonarQube UI | `MyJavaApp` |
| `-Dsonar.projectKey` | Unique identifier (must be unique) | `MyJavaApp` |
| `-Dsonar.java.binaries` | Compiled class files location | `target/classes` |
| `-Dsonar.sources` | Source code location | `src/main/java` |
| `-Dsonar.tests` | Test code location | `src/test/java` |
| `-Dsonar.junit.reportPaths` | JUnit test results | `target/surefire-reports` |
| `-Dsonar.coverage.jacoco.xmlReportPaths` | JaCoCo coverage report | `target/site/jacoco/jacoco.xml` |

**Key Insights:**

1. **`withSonarQubeEnv('sonar-server')`**: 
   - Injects SonarQube server configuration
   - Automatically adds authentication token
   - Sets up environment variables

2. **`waitForQualityGate abortPipeline: true`**:
   - Waits for SonarQube to complete analysis
   - Receives webhook callback with Quality Gate status
   - Aborts pipeline if Quality Gate fails

3. **`timeout(time: 1, unit: 'HOURS')`**:
   - Prevents pipeline from hanging forever
   - Typical analysis takes 2-5 minutes
   - 1 hour is generous timeout

### Node.js Project Analysis

Node.js requires different configuration since it uses npm/jest instead of Maven.

#### Prerequisites for Node.js

**Install NodeJS Plugin in Jenkins:**
- Go to **Manage Jenkins** → **Manage Plugins**
- Install **NodeJS Plugin**
- Configure Node.js in **Global Tool Configuration**

**Configure package.json for coverage:**

```json
{
  "name": "nodejs-app",
  "version": "1.0.0",
  "scripts": {
    "test": "jest --coverage",
    "test:ci": "jest --coverage --coverageReporters=lcov"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  },
  "jest": {
    "coverageDirectory": "coverage",
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/**/*.test.js"
    ]
  }
}
```

#### Complete Jenkins Pipeline for Node.js

```groovy
pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-repo/nodejs-app.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                nodejs('nodejs20') {  // Name from Global Tool Configuration
                    sh 'npm install'
                }
            }
        }
        
        stage('Run Tests with Coverage') {
            steps {
                nodejs('nodejs20') {
                    sh 'npm run test:ci'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=NodeJSApp \
                        -Dsonar.projectKey=NodeJSApp \
                        -Dsonar.sources=. \
                        -Dsonar.tests=. \
                        -Dsonar.test.inclusions=**/*.test.js \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                        -Dsonar.exclusions=node_modules/**,coverage/**,dist/**
                    '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```

#### Node.js Specific Parameters

| Parameter | Purpose | Value |
|-----------|---------|-------|
| `-Dsonar.sources` | All source files | `.` (current directory) |
| `-Dsonar.tests` | Test file location | `.` (same as sources) |
| `-Dsonar.test.inclusions` | Pattern to identify test files | `**/*.test.js` |
| `-Dsonar.javascript.lcov.reportPaths` | LCOV coverage report location | `coverage/lcov.info` |
| `-Dsonar.exclusions` | Files to exclude from analysis | `node_modules/**,coverage/**` |

**Why these parameters?**

1. **`-Dsonar.sources=.`**: Node.js projects don't have strict src/main structure
2. **`-Dsonar.test.inclusions=**/*.test.js`**: Identifies Jest test files
3. **`-Dsonar.javascript.lcov.reportPaths`**: Jest generates LCOV format coverage
4. **`-Dsonar.exclusions`**: Exclude dependencies and generated files

---

## Quality Gates & Metrics

### What are Quality Gates?

**Quality Gates** are the gatekeepers of your codebase. They define the minimum quality standards code must meet before it can be merged or deployed.

**Real-World Analogy:**
Think of Quality Gates like airport security. Just as you can't board a plane without passing security checks, code can't be deployed without passing quality checks.

### Default Quality Gate

SonarQube comes with a "Sonar way" quality gate:

```
Conditions on New Code:
✓ Coverage ≥ 80%
✓ Duplicated Lines ≤ 3%
✓ Security Hotspots Reviewed = 100%
✓ Maintainability Rating = A
✓ Reliability Rating = A
✓ Security Rating = A
```

**Key Concept**: Quality Gates focus on **new code**, not legacy code. This is brilliant because:
- Legacy code might have issues (technical debt)
- You can't fix everything at once
- But you can ensure **new code is clean**

### Creating Custom Quality Gates

Based on my experience, here's how I create project-specific quality gates.

#### Step 1: Navigate to Quality Gates

**SonarQube Dashboard** → **Quality Gates** → **Create**

#### Step 2: Define Conditions

**Example: Strict Quality Gate for Critical Projects**

```
Name: High Security Standards

Conditions on Overall Code:
- Critical Issues = 0
- Blocker Issues = 0
- Security Vulnerabilities = 0

Conditions on New Code:
- Coverage ≥ 90%
- Duplicated Lines ≤ 2%
- Maintainability Rating = A
- Reliability Rating = A
- Security Rating = A
- Security Hotspots Reviewed = 100%
- Code Smells ≤ 10
```

**Example: Balanced Quality Gate for Fast-Moving Projects**

```
Name: Balanced Standards

Conditions on New Code:
- Coverage ≥ 75%
- Critical Issues = 0
- Blocker Issues = 0
- Security Vulnerabilities = 0
- Duplicated Lines ≤ 5%
```

#### Step 3: Assign Quality Gate to Project

**Project Settings** → **Quality Gate** → Select your gate → **Save**

### Understanding Metrics

#### 1. Code Coverage

**Definition**: Percentage of code executed by unit tests.

**Formula:**
```
Coverage = (Lines Executed by Tests / Total Lines) × 100
```

**Example:**
```java
public class Calculator {
    public int add(int a, int b) {        // Line 1
        return a + b;                     // Line 2
    }
    
    public int subtract(int a, int b) {   // Line 3
        return a - b;                     // Line 4
    }
}

// Test covers add() but not subtract()
@Test
public void testAdd() {
    assertEquals(5, calculator.add(2, 3));  // Covers lines 1-2
}

// Coverage: 50% (2 out of 4 lines covered)
```

**Industry Standards:**
- **Critical Business Logic**: 90%+
- **General Application Code**: 80%+
- **UI/Presentation Layer**: 60-70%

**My Learning**: 100% coverage isn't always necessary. Focus on critical paths.

#### 2. Code Duplication

**Definition**: Percentage of duplicated code blocks.

**Example:**
```java
// Duplicated code (bad practice)
public void processOrder() {
    if (isValid(order)) {
        double price = order.getPrice();
        double tax = price * 0.1;
        double total = price + tax;
        saveOrder(order);
    }
}

public void processQuote() {
    if (isValid(quote)) {
        double price = quote.getPrice();
        double tax = price * 0.1;      // Duplicated logic
        double total = price + tax;     // Duplicated logic
        saveQuote(quote);
    }
}
```

**Better approach:**
```java
// Refactored to eliminate duplication
private double calculateTotal(double price) {
    double tax = price * 0.1;
    return price + tax;
}

public void processOrder() {
    if (isValid(order)) {
        double total = calculateTotal(order.getPrice());
        saveOrder(order);
    }
}
```

**Target**: Keep duplications below 3%

#### 3. Technical Debt

**Definition**: Estimated time to fix all code quality issues.

**Calculation:**
```
Issue Type          Time to Fix     Count     Total
Critical Issues     30 min          5         2.5 hours
Major Issues        15 min          20        5 hours
Minor Issues        5 min           50        4.2 hours
                                    Total:    11.7 hours
```

**Technical Debt Ratio:**
```
Debt Ratio = (Remediation Cost / Development Cost) × 100

Example:
- Technical Debt: 12 hours
- Development Cost: 100 hours
- Debt Ratio: 12%
```

**Industry Benchmarks:**
- **≤ 5%**: Excellent (A rating)
- **6-10%**: Good (B rating)
- **11-20%**: Moderate (C rating)
- **21-50%**: Poor (D rating)
- **> 50%**: Critical (E rating)

**My Experience**: Keep debt ratio below 10% for maintainable codebases.

#### 4. Security Hotspots

**Definition**: Code patterns that might be security vulnerabilities.

**Example Hotspots:**
```java
// Hotspot: Weak cryptography
MessageDigest md = MessageDigest.getInstance("MD5");  // MD5 is weak

// Hotspot: Hardcoded credentials
String password = "admin123";  // Should use environment variables

// Hotspot: SQL Injection risk
String query = "SELECT * FROM users WHERE id = " + userId;  // Use prepared statements
```

**Best Practice**: Review and resolve 100% of security hotspots.

#### 5. Bugs vs Vulnerabilities vs Code Smells

| Type | Impact | Severity | Example |
|------|--------|----------|---------|
| **Bug** | Incorrect behavior | High | NullPointerException, Array index out of bounds |
| **Vulnerability** | Security risk | Critical | SQL Injection, XSS, Hardcoded secrets |
| **Code Smell** | Maintainability | Medium | Long methods, high complexity, duplications |

### Real-World Quality Gate Scenario

**Scenario**: You're working on a financial application.

**Initial Analysis Results:**
```
Overall Code:
- Coverage: 65%
- Critical Issues: 3
- Security Vulnerabilities: 2
- Technical Debt: 15 hours

New Code (your latest commit):
- Coverage: 85%
- Critical Issues: 0
- Security Vulnerabilities: 0
- Technical Debt: 0.5 hours
```

**Quality Gate (Strict):**
```
✅ New Code Coverage ≥ 80% (Passed: 85%)
✅ New Code Critical Issues = 0 (Passed: 0)
✅ New Code Vulnerabilities = 0 (Passed: 0)
❌ Overall Critical Issues = 0 (Failed: 3 exist)
❌ Overall Security Vulnerabilities = 0 (Failed: 2 exist)
```

**Result**: Quality Gate **FAILS** because overall code has issues.

**My Learning**: This forces teams to fix legacy issues gradually while ensuring new code is clean.

**Decision Options:**
1. **Fix legacy issues** (recommended for critical projects)
2. **Adjust Quality Gate** to focus only on new code (pragmatic approach)
3. **Create exemption** with proper justification (last resort)

---

## Real-World Use Cases

Based on my hands-on experience, here are practical scenarios where SonarQube proved invaluable.

### Use Case 1: Preventing Security Vulnerabilities

**Situation**: Team was pushing code with hardcoded credentials.

**Before SonarQube:**
```java
public class DatabaseConfig {
    private static final String DB_PASSWORD = "prod_password_123";  // Hardcoded!
    
    public Connection getConnection() {
        return DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
    }
}
```

**Problem**: 
- Credentials exposed in Git history
- Security audit failed
- Emergency hotfix required

**After SonarQube Integration:**

Pipeline failed with:
```
Quality Gate FAILED
Critical: Remove this hard-coded password
Vulnerability: Credentials should not be hardcoded
```

**Solution:**
```java
public class DatabaseConfig {
    private static final String DB_PASSWORD = System.getenv("DB_PASSWORD");
    
    public Connection getConnection() {
        return DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
    }
}
```

**Impact:**
- ✅ Security issue caught before production
- ✅ Forced developers to use environment variables
- ✅ Passed security audit

### Use Case 2: Reducing Technical Debt

**Situation**: Legacy codebase with 50% technical debt ratio.

**Strategy:**
```
Week 1: Set Quality Gate for new code only
Week 2-4: Refactor critical paths
Week 5-8: Address high-severity issues
Week 9+: Maintain clean state
```

**Results:**
```
Month 0: 50% debt ratio (250 hours)
Month 1: 45% debt ratio (225 hours) - 10% improvement
Month 3: 30% debt ratio (150 hours) - 40% improvement
Month 6: 15% debt ratio (75 hours) - 70% improvement
```

**My Learning**: Focus on new code first, then gradually fix legacy issues.

### Use Case 3: Code Coverage Enforcement

**Situation**: Team had inconsistent test coverage (20-90% across modules).

**Before:**
```
Module A: 85% coverage (well tested)
Module B: 25% coverage (critical but untested)
Module C: 60% coverage (moderate)
```

**SonarQube Quality Gate:**
```
New Code Coverage ≥ 80%
```

**After 3 Months:**
```
Module A: 90% coverage (maintained)
Module B: 75% coverage (significantly improved)
Module C: 82% coverage (improved)

Overall: 82% average coverage
```

**Impact:**
- ✅ Found 15 bugs through new tests
- ✅ Increased developer confidence
- ✅ Reduced production incidents by 40%

### Use Case 4: Multi-Branch Analysis

**Situation**: Different branches had different code quality.

**Challenge**: Community edition only analyzes one branch.

**Solution**: Used Docker containers with branch-specific instances.

```bash
# Main branch
docker run -d --name sonarqube-main -p 9000:9000 sonarqube:lts-community

# Development branch
docker run -d --name sonarqube-dev -p 9001:9000 sonarqube:lts-community

# Feature branches (temporary)
docker run -d --name sonarqube-feature -p 9002:9000 sonarqube:lts-community
```

**Alternative**: Upgrade to Developer Edition for native multi-branch support.

### Use Case 5: CI/CD Integration

**Full Pipeline Scenario:**

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') { /* ... */ }
        
        stage('Unit Tests') { /* ... */ }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Deploying to staging...'
            }
        }
    }
}
```

**Flow:**
1. Code pushed to repository
2. Jenkins webhook triggered
3. Build and tests run
4. SonarQube analyzes code
5. Quality Gate evaluated
6. **If passed**: Deploy to staging
7. **If failed**: Pipeline aborts, notify team

**Result**: Only clean code reaches staging/production.

---

## Production Setup with SSL

For production environments, always secure SonarQube with HTTPS.

### Architecture with Nginx Reverse Proxy

```
Internet → HTTPS (443) → Nginx → HTTP (9000) → SonarQube
```

**Benefits:**
- ✅ SSL/TLS encryption
- ✅ Professional domain (sonarqube.company.com)
- ✅ Better performance (Nginx caching)
- ✅ Additional security layer

### Step-by-Step SSL Setup

#### Step 1: Install Nginx

```bash
sudo apt install nginx -y
```

#### Step 2: Configure Nginx for SonarQube

```bash
sudo nano /etc/nginx/sites-available/sonarqube.conf
```

**Add configuration:**

```nginx
server {
    listen 80;
    server_name sonarqube.yourdomain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sonarqube.yourdomain.com;
    
    # SSL Certificate paths (will be added by Certbot)
    ssl_certificate /etc/letsencrypt/live/sonarqube.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sonarqube.yourdomain.com/privkey.pem;
    
    # SSL Security settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Proxy settings
    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Increase timeouts for large file uploads
    client_max_body_size 50M;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;
    proxy_read_timeout 300;
}
```

**Enable the configuration:**

```bash
sudo ln -s /etc/nginx/sites-available/sonarqube.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### Step 3: Install Certbot for Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx -y
```

#### Step 4: Obtain SSL Certificate

```bash
sudo certbot --nginx -d sonarqube.yourdomain.com
```

**Follow prompts:**
```
Enter email: your-email@company.com
Agree to Terms of Service: Yes
Share email with EFF: Your choice
```

**Certbot will:**
- Verify domain ownership
- Generate SSL certificate
- Configure Nginx automatically
- Set up auto-renewal

#### Step 5: Verify SSL Certificate

```bash
# Check certificate expiry
sudo certbot certificates

# Test auto-renewal
sudo certbot renew --dry-run
```

#### Step 6: Update SonarQube Configuration

```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

**Update:**
```properties
# Change web host to localhost only (Nginx will handle external access)
sonar.web.host=127.0.0.1

# Base URL for webhooks and callbacks
sonar.web.context=/
sonar.web.port=9000
```

**Restart SonarQube:**
```bash
sudo systemctl restart sonarqube
```

#### Step 7: Test HTTPS Access

1. Open browser: `https://sonarqube.yourdomain.com`
2. Verify SSL certificate (should show padlock icon)
3. Login and test functionality

**Security Verification:**
```bash
# Test SSL configuration
curl -I https://sonarqube.yourdomain.com

# Should return:
HTTP/2 200
server: nginx
...
```

### Firewall Configuration for Production

```bash
# Allow HTTPS
sudo ufw allow 443/tcp

# Allow HTTP (for Let's Encrypt verification)
sudo ufw allow 80/tcp

# Block direct access to SonarQube port
sudo ufw deny 9000/tcp

# Enable firewall
sudo ufw enable

# Verify rules
sudo ufw status
```

**Result**: SonarQube only accessible through Nginx (secured with SSL).

---

## Best Practices & Lessons Learned

### 1. Start with Realistic Quality Gates

**Mistake I made**: Set 90% coverage requirement on legacy project.

**Result**: Developers spent weeks writing tests for old code instead of new features.

**Better Approach**:
```
Phase 1 (Week 1-2): Quality Gate for new code only
- New Code Coverage ≥ 70%
- New Code Critical Issues = 0

Phase 2 (Month 2-3): Gradually increase standards
- New Code Coverage ≥ 80%
- Overall Critical Issues < 10

Phase 3 (Month 4+): Strict standards
- New Code Coverage ≥ 85%
- Overall Critical Issues = 0
```

### 2. Focus on New Code First

**Key Insight**: It's impossible to fix all legacy issues immediately.

**Strategy**:
```
✅ Enforce strict rules on NEW code
✅ Gradually refactor legacy code
✅ Prioritize critical security vulnerabilities
✅ Track technical debt trend (should decrease over time)
```

**Metrics to Watch**:
```
New Code Coverage: 90% (Excellent!)
Overall Coverage: 65% (Improving from 60% last month)
```

### 3. Automate Everything

**Manual Analysis**: Developers forget to run SonarQube.

**Automated Analysis**: Every commit triggers analysis.

**Implementation**:
- ✅ Jenkins webhook on every push
- ✅ Quality Gate blocks merge if failed
- ✅ Slack notifications for failures
- ✅ Dashboard visible to entire team

### 4. Review Security Hotspots Immediately

**Lesson Learned**: Security hotspots aren't always vulnerabilities, but they need review.

**Process**:
```
1. SonarQube flags security hotspot
2. Developer reviews code
3. If safe: Mark as "Reviewed - Safe"
4. If risky: Fix immediately
5. Document decision
```

**Example**:
```java
// Hotspot: Weak cryptography
MessageDigest md = MessageDigest.getInstance("SHA-1");

// Review Decision:
// - Used for non-security purposes (cache key generation)
// - Not used for passwords or sensitive data
// - Marked as "Reviewed - Safe"
```

### 5. Don't Chase 100% Coverage

**My Experience**: Spent days getting from 90% to 95% coverage.

**Diminishing Returns**:
- 90% → 95%: 2 weeks effort, found 1 bug
- 70% → 80%: 3 days effort, found 12 bugs

**Better Approach**:
```
Critical Business Logic: 90%+ coverage
Standard Services: 80%+ coverage
UI/Controllers: 70%+ coverage
Getters/Setters: Don't bother (low value)
```

### 6. Customize Rules for Your Context

**Default Rules**: Good starting point, but not perfect.

**Customization Examples**:
```
Project: Microservices
- Enable: API documentation checks
- Disable: Long method checks (some methods handle HTTP requests)

Project: Data Processing
- Enable: Performance rules
- Increase: Complexity threshold (data transformations are complex)

Project: Security-Critical
- Enable: All security rules
- Zero tolerance: Security vulnerabilities
```

### 7. Integrate with Developer Workflow

**Poor Integration**: Developers only see results in Jenkins logs.

**Better Integration**:
- ✅ IDE plugins (SonarLint for IntelliJ, VS Code)
- ✅ Pre-commit hooks (optional, but helpful)
- ✅ Pull request comments (shows issues in context)
- ✅ Daily email summaries
- ✅ Team dashboard on monitor

### 8. Regular Rule Updates

**Mistake**: Kept same rules for 2 years.

**Better Approach**:
```
Quarterly Review:
1. Check for new rule sets
2. Review false positives
3. Add rules for new technologies
4. Remove obsolete rules
5. Update quality gates based on trends
```

### 9. Document Exemptions

**Scenario**: Some code legitimately can't meet standards.

**Example**:
```java
// SonarQube: "Method has 150 lines (max 50)"
// Exemption reason: Legacy code, used by 10+ systems
// Refactoring risk: HIGH
// Decision: Suppress warning, refactor in Q4
// Approved by: Tech Lead
// Date: 2024-03-15
```

**Track Exemptions**: Review annually to prevent accumulation.

### 10. Monitor Trends, Not Just Snapshots

**Single Snapshot**: "We have 500 issues!"

**Trend Analysis**: "We had 600 issues last month, 550 last week, 500 today. Improving!"

**Dashboards I Use**:
- Technical debt trend (weekly)
- Code coverage trend (per sprint)
- New issues introduced (daily)
- Security vulnerabilities (real-time alerts)

---

## Troubleshooting Common Issues

### Issue 1: Quality Gate Timeout

**Problem**:
```
stage('Quality Gate Check') {
    timeout(time: 1, unit: 'HOURS') {
        waitForQualityGate abortPipeline: true
    }
}
// Hangs indefinitely
```

**Cause**: Webhook not configured or not reaching Jenkins.

**Solution:**
```bash
# 1. Verify webhook in SonarQube
Administration → Webhooks → Check URL

# 2. Test webhook
curl -X POST http://your-jenkins:8080/sonarqube-webhook/

# 3. Check Jenkins logs
tail -f /var/log/jenkins/jenkins.log

# 4. Verify firewall
sudo ufw status
```

### Issue 2: Scanner Fails with "Out of Memory"

**Error**:
```
java.lang.OutOfMemoryError: Java heap space
```

**Solution**:
```bash
# Increase scanner memory
export SONAR_SCANNER_OPTS="-Xmx1024m"

# Or in Jenkins pipeline
environment {
    SONAR_SCANNER_OPTS = "-Xmx1024m"
}
```

### Issue 3: Coverage Report Not Found

**Error**:
```
WARN: Coverage report 'target/site/jacoco/jacoco.xml' not found
```

**Cause**: JaCoCo plugin not configured or tests not run.

**Solution**:
```bash
# 1. Verify JaCoCo in pom.xml
grep -A 10 "jacoco-maven-plugin" pom.xml

# 2. Run tests to generate report
mvn clean test

# 3. Verify report exists
ls -la target/site/jacoco/jacoco.xml

# 4. Check SonarQube scanner logs
```

### Issue 4: Analysis Takes Too Long

**Problem**: Analysis taking 30+ minutes for medium project.

**Solutions**:
```bash
# 1. Exclude unnecessary files
-Dsonar.exclusions=**/test/**,**/node_modules/**,**/dist/**

# 2. Disable unused languages
# In SonarQube UI: Administration → General Settings → Languages

# 3. Increase scanner memory
-Xmx2048m

# 4. Use incremental analysis (Enterprise only)
-Dsonar.incremental=true
```

### Issue 5: False Positives

**Problem**: SonarQube flags code that's actually fine.

**Solution**:
```java
// Suppress specific rule
@SuppressWarnings("java:S3776")  // Cognitive Complexity
public void complexButNecessaryMethod() {
    // Complex logic that can't be simplified
}

// Or mark as false positive in SonarQube UI
// Project → Issues → Mark as "False Positive" → Add comment
```

---

## Key Metrics Summary
**1. Measurable Improvements**
```
Before: 45% code coverage, 250 critical issues
After: 82% code coverage, 0 critical issues
Impact: 60% reduction in production bugs
```

**2. Process Implementation**
```
- Integrated SonarQube with Jenkins CI/CD
- Enforced Quality Gates (blocked 15 bad deployments)
- Reduced code review time by 40%
```

**3. Technical Skills**
```
- Configured SonarQube with PostgreSQL backend
- Set up SSL with Nginx reverse proxy
- Created custom Quality Gates for microservices
- Integrated with Java, Node.js, and Python projects
```

**4. Security Focus**
```
- Caught 23 security vulnerabilities before production
- Implemented SAST in development pipeline
- Achieved 100% security hotspot review rate
```

---

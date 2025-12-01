# Module 4: Build Tools - Complete Guide

## Table of Contents
1. [Introduction to Build Tools](#introduction-to-build-tools)
2. [Maven - Java Build Tool](#maven---java-build-tool)
3. [Node.js and npm](#nodejs-and-npm)
4. [.NET Build Tools](#net-build-tools)
5. [Apache Tomcat Setup & Deployment](#apache-tomcat-setup--deployment)
6. [Practical Examples & Best Practices](#practical-examples--best-practices)

---

## Introduction to Build Tools

### What are Build Tools?

Build tools automate the process of converting source code into executable packages that users can run. They handle three key components:

1. **Source Code**: The actual application code written by developers
2. **Dependencies**: External libraries and packages the application needs
3. **Configuration**: Database information, API keys, and other settings

**Why Build Tools?**
- Source code alone doesn't show how an application really works
- Raw code isn't user-friendly or deployable
- Build tools package everything into a format (JAR, WAR, ZIP) that can be deployed and executed

### What are Artifacts?

Artifacts are the specific packages generated when you build an application:
- **Java**: JAR (Java Archive) or WAR (Web Application Archive) files
- **Node.js**: ZIP or TAR files with node_modules
- **.NET**: DLL or EXE files

---

## Maven - Java Build Tool

### Overview

Apache Maven is a build automation and project management tool that:
- Manages project dependencies
- Provides a standardized build lifecycle
- Handles artifact creation and deployment
- Uses convention over configuration

### Maven Project Structure

```
project-root/
│
├── src/
│   ├── main/
│   │   ├── java/              # Java source code
│   │   ├── resources/         # Configuration files
│   │   └── webapp/            # Web application content
│   │
│   └── test/
│       ├── java/              # Test source code
│       └── resources/         # Test resources
│
├── target/                    # Compiled classes and artifacts
├── pom.xml                    # Project Object Model (POM)
└── README.md                  # Project documentation
```

### The POM File (pom.xml)

The Project Object Model is the heart of Maven. It contains:

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- Project dependencies -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.32</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Build plugins -->
        </plugins>
    </build>
</project>
```

**Key Elements:**
- `groupId`: Organization or group identifier
- `artifactId`: Project name
- `version`: Project version
- `dependencies`: External libraries needed
- `build`: Build configuration and plugins

### Maven Lifecycle

Maven has three main lifecycles:

#### 1. Clean Lifecycle
- **clean**: Deletes the `target/` directory and all generated files

#### 2. Default Lifecycle (Main Build Lifecycle)
- **validate**: Validates project structure and configuration
- **compile**: Compiles source code (checks for syntax errors)
- **test**: Runs unit tests
- **package**: Creates distributable package (JAR/WAR)
- **verify**: Runs checks to ensure package quality
- **install**: Installs package to local Maven repository (~/.m2)
- **deploy**: Copies package to remote repository (Nexus/Artifactory)

#### 3. Site Lifecycle
- **site**: Generates project documentation

**Important**: When you run a Maven phase, all preceding phases execute automatically.

### Essential Maven Commands

```bash
# Clean the project
mvn clean

# Compile source code
mvn compile

# Run tests
mvn test

# Package the application
mvn package

# Skip tests during build
mvn package -DskipTests=true

# Clean and package
mvn clean package

# Install to local repository
mvn install

# Deploy to remote repository
mvn deploy

# Update dependencies
mvn clean package -U
```

### Maven Repositories

**Three Types:**

1. **Local Repository** (~/.m2/repository)
   - On developer's machine
   - Stores downloaded dependencies

2. **Central Repository** (https://repo.maven.apache.org/maven2)
   - Default public Maven repository
   - Contains most open-source libraries

3. **Remote Repository** (Nexus/Artifactory)
   - Organization's private repository
   - Stores internal artifacts

### Dependency Management

**Dependency Scopes:**
- `compile`: Default, available everywhere
- `test`: Only for testing
- `runtime`: Not needed for compilation, only runtime
- `provided`: Provided by runtime environment (e.g., servlet API)

**Example:**
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

### Maven Plugins

Plugins extend Maven's functionality:

**Common Plugins:**

1. **maven-compiler-plugin**: Compiles Java code
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

2. **maven-surefire-plugin**: Runs unit tests
3. **maven-jar-plugin**: Creates JAR files
4. **maven-war-plugin**: Creates WAR files for web applications

### Maven Arguments

Common `-D` arguments to customize builds:

```bash
# Skip tests
-DskipTests=true

# Set Java version
-Dmaven.compiler.source=11 -Dmaven.compiler.target=11

# Custom local repository
-Dmaven.repo.local=/custom/path

# Skip install/deploy
-Dmaven.skip.install=true
-Dmaven.skip.deploy=true

# Verbose output
-Dmaven.verbose=true

# Ignore test failures
-Dmaven.test.failure.ignore=true
```

### Practical Maven Workflow

```bash
# 1. Create new Maven project
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=my-app \
  -DarchetypeArtifactId=maven-archetype-quickstart

# 2. Navigate to project
cd my-app

# 3. First-time build (downloads dependencies)
mvn clean package

# 4. Subsequent builds (faster, uses cached dependencies)
mvn package

# 5. Run the application
java -jar target/my-app-1.0.jar

# 6. Install to local repository
mvn install

# 7. Deploy to Nexus
mvn deploy
```

---

## Node.js and npm

### Overview

**Node.js**: JavaScript runtime environment for server-side execution
- Built on Chrome's V8 JavaScript engine
- Event-driven, non-blocking I/O
- Efficient for handling concurrent connections

**npm (Node Package Manager)**: Default package manager for Node.js
- Manages project dependencies
- Hosts public registry of packages
- Provides scripts for build automation

### Node.js Project Structure

```
my-nodejs-project/
│
├── node_modules/          # Installed dependencies
├── public/                # Static assets
│   ├── index.html
│   └── styles/
│       └── main.css
├── src/                   # Source code
│   └── index.js
├── .gitignore            # Git ignore rules
├── package.json          # Project metadata
└── README.md             # Documentation
```

### package.json - Project Configuration

```json
{
  "name": "my-nodejs-project",
  "version": "1.0.0",
  "description": "A Node.js project",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "build": "webpack --mode production",
    "test": "jest",
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.15",
    "jest": "^27.5.0"
  },
  "engines": {
    "node": ">=14.0.0"
  }
}
```

**Key Sections:**
- **scripts**: Custom commands you can run with `npm run`
- **dependencies**: Required for production
- **devDependencies**: Only needed during development
- **engines**: Specifies Node.js version requirements

### npm Commands

```bash
# Install all dependencies
npm install

# Install specific package
npm install express

# Install as dev dependency
npm install --save-dev jest

# Install globally
npm install -g nodemon

# Update dependencies
npm update

# Update specific package
npm update express

# Run scripts
npm run start
npm run build
npm run test

# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix
```

### Semantic Versioning (SemVer)

Format: **MAJOR.MINOR.PATCH** (e.g., 1.4.2)

- **MAJOR**: Breaking changes (incompatible API changes)
- **MINOR**: New features (backward-compatible)
- **PATCH**: Bug fixes (backward-compatible)

**Version Ranges in package.json:**
- `"express": "4.18.0"` - Exact version
- `"express": "^4.18.0"` - Compatible with 4.x.x (< 5.0.0)
- `"express": "~4.18.0"` - Compatible with 4.18.x

### npm Scripts Best Practices

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "build": "webpack --mode production",
    "test": "jest --coverage",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.js",
    "clean": "rm -rf dist node_modules"
  }
}
```

### Global vs Local Packages

**Local Installation** (default):
```bash
npm install express
```
- Installed in `node_modules/` of current project
- Available only to this project

**Global Installation**:
```bash
npm install -g nodemon
```
- Installed system-wide
- Use for CLI tools you want everywhere
- Examples: nodemon, create-react-app, typescript

---

## .NET Build Tools

### Overview

.NET is Microsoft's free, open-source, cross-platform framework for building modern applications.

**Supports Multiple Languages:**
- C#
- F#
- Visual Basic

**Platforms:**
- Windows
- macOS
- Linux

### .NET Project Structure

```
MyDotNetProject/
│
├── MyDotNetProject.sln      # Solution file (manages multiple projects)
├── src/
│   └── MyDotNetProject/
│       ├── Program.cs        # Entry point (Main method)
│       ├── MyDotNetProject.csproj  # Project file
│       └── Controllers/      # Application code
├── test/
│   └── MyDotNetProject.Tests/
│       ├── MyDotNetProject.Tests.csproj
│       └── UnitTests.cs
├── obj/                      # Intermediate build files
├── bin/                      # Compiled binaries
├── .gitignore
└── README.md
```

### Project File (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.1" />
  </ItemGroup>
</Project>
```

### .NET CLI Commands

```bash
# Create new project
dotnet new console -n MyApp
dotnet new web -n MyWebApp
dotnet new classlib -n MyLibrary

# Restore dependencies
dotnet restore

# Build the project
dotnet build

# Build in Release mode
dotnet build -c Release

# Run the application
dotnet run

# Run tests
dotnet test

# Publish for deployment
dotnet publish -c Release -o ./publish

# Clean build artifacts
dotnet clean

# Add NuGet package
dotnet add package Newtonsoft.Json

# Remove NuGet package
dotnet remove package Newtonsoft.Json

# List installed packages
dotnet list package
```

### .NET Build Process

1. **Restore**: Downloads NuGet packages
2. **Build**: Compiles source code to IL (Intermediate Language)
3. **Test**: Runs unit tests
4. **Publish**: Creates deployment-ready output

### Configuration Files

**appsettings.json** - Application configuration:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=mydb;User=sa;Password=Pass123"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

### NuGet Package Management

NuGet is the package manager for .NET (like npm for Node.js, Maven for Java).

```bash
# Install package
dotnet add package EntityFramework

# Install specific version
dotnet add package EntityFramework --version 6.4.4

# Update all packages
dotnet restore
```

---

## Apache Tomcat Setup & Deployment

### What is Apache Tomcat?

Apache Tomcat is an open-source Java web server and servlet container:
- Implements Java Servlet and JSP specifications
- Suitable for deploying Java web applications (WAR files)
- Lightweight and easy to configure

**Key Components:**
- **Catalina**: Servlet container
- **Coyote**: HTTP connector
- **Jasper**: JSP engine

### Installing Tomcat (Version 9.0.65)

```bash
# 1. Switch to superuser
sudo su

# 2. Navigate to /opt directory
cd /opt

# 3. Download Tomcat
sudo wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz

# 4. Extract the archive
sudo tar -xvf apache-tomcat-9.0.65.tar.gz

# 5. Navigate to conf directory
cd /opt/apache-tomcat-9.0.65/conf

# 6. Configure users
sudo vi tomcat-users.xml
```

### Configuring Tomcat Users

Add this line before `</tomcat-users>`:

```xml
<user username="admin" password="admin1234" roles="admin-gui,manager-gui,manager-script"/>
```

**Roles:**
- `admin-gui`: Access to Host Manager
- `manager-gui`: Access to Manager App GUI
- `manager-script`: Access for scripts/tools

### Creating Symbolic Links

```bash
# Create easy-to-use commands
sudo ln -s /opt/apache-tomcat-9.0.65/bin/startup.sh /usr/bin/startTomcat
sudo ln -s /opt/apache-tomcat-9.0.65/bin/shutdown.sh /usr/bin/stopTomcat

# Now you can use:
startTomcat
stopTomcat
```

### Configuring Remote Access

By default, Tomcat Manager is accessible only from localhost. To enable remote access:

**For Manager App:**
```bash
sudo vi /opt/apache-tomcat-9.0.65/webapps/manager/META-INF/context.xml
```

Comment out the Valve:
```xml
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```

**For Host Manager:**
```bash
sudo vi /opt/apache-tomcat-9.0.65/webapps/host-manager/META-INF/context.xml
```

Comment out the same Valve section.

### Starting Tomcat

```bash
# Start Tomcat
sudo startTomcat

# Stop Tomcat
sudo stopTomcat

# Check if Tomcat is running
sudo lsof -i :8080
```

**Access Tomcat:**
- Default URL: `http://<your-server-ip>:8080`
- Manager App: `http://<your-server-ip>:8080/manager`
- Host Manager: `http://<your-server-ip>:8080/host-manager`

### Deploying Applications to Tomcat

**Method 1: Copy WAR file**
```bash
# Copy WAR to webapps directory
sudo cp myapp.war /opt/apache-tomcat-9.0.65/webapps/

# Tomcat auto-deploys and extracts the WAR
# Access at: http://localhost:8080/myapp
```

**Method 2: Using Manager GUI**
1. Go to `http://localhost:8080/manager`
2. Login with configured credentials
3. Scroll to "Deploy" section
4. Upload WAR file
5. Click "Deploy"

**Method 3: Maven Deploy Plugin**

Configure in `pom.xml`:
```xml
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://localhost:8080/manager/text</url>
        <server>TomcatServer</server>
        <path>/myapp</path>
    </configuration>
</plugin>
```

Deploy:
```bash
mvn tomcat7:deploy
```

---

## Practical Examples & Best Practices

### Example 1: Building a Java Project with Maven

```bash
# Clone repository
git clone https://github.com/example/java-app.git
cd java-app

# Check pom.xml for dependencies

# First build (downloads dependencies)
mvn clean package -DskipTests=true

# Run the application
java -jar target/app-1.0.jar

# If using Spring Boot
mvn spring-boot:run
```

### Example 2: Database Configuration

**application.properties** (Java/Spring Boot):
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

**Connecting to MySQL:**
```bash
# Install MySQL
sudo apt install mysql-server

# Login to MySQL
sudo mysql -u root -p

# Create database and user
CREATE DATABASE mydb;
CREATE USER 'appuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

### Example 3: Setting up Nexus Repository in Maven

**pom.xml configuration:**
```xml
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://nexus-server:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://nexus-server:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

**settings.xml** (~/.m2/settings.xml):
```xml
<servers>
    <server>
        <id>nexus-releases</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
    <server>
        <id>nexus-snapshots</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

### Best Practices

#### Maven Best Practices

1. **Always use clean before important builds**
   ```bash
   mvn clean package
   ```

2. **Use dependency management for multi-module projects**
   ```xml
   <dependencyManagement>
       <dependencies>
           <!-- Centralized version management -->
       </dependencies>
   </dependencyManagement>
   ```

3. **Avoid SNAPSHOT versions in production**
   - Use stable versions for production releases
   - SNAPSHOTs are for development

4. **Profile for different environments**
   ```xml
   <profiles>
       <profile>
           <id>dev</id>
           <!-- Dev configuration -->
       </profile>
       <profile>
           <id>prod</id>
           <!-- Production configuration -->
       </profile>
   </profiles>
   ```

#### Node.js Best Practices

1. **Lock dependency versions**
   - Commit `package-lock.json` to version control
   - Ensures consistent builds

2. **Separate dependencies**
   - Use `dependencies` for production packages
   - Use `devDependencies` for development tools

3. **Use npm scripts for common tasks**
   ```json
   {
     "scripts": {
       "start": "node server.js",
       "test": "jest",
       "build": "webpack"
     }
   }
   ```

#### .NET Best Practices

1. **Use appropriate target framework**
   ```xml
   <TargetFramework>net6.0</TargetFramework>
   ```

2. **Publish in Release mode**
   ```bash
   dotnet publish -c Release
   ```

3. **Use configuration transformations**
   - `appsettings.Development.json`
   - `appsettings.Production.json`

#### Tomcat Best Practices

1. **Change default credentials immediately**
2. **Use appropriate Java version** (Java 11 or 17 recommended)
3. **Configure memory settings** in `setenv.sh`:
   ```bash
   CATALINA_OPTS="-Xms512m -Xmx1024m"
   ```

4. **Enable access logs** for troubleshooting
5. **Regular security updates** - keep Tomcat updated

### Common Issues and Solutions

#### Maven Issues

**Issue**: Dependencies not downloading
```bash
# Solution: Force update
mvn clean install -U
```

**Issue**: Build fails due to test failures
```bash
# Solution: Skip tests temporarily
mvn package -DskipTests=true
```

#### Node.js Issues

**Issue**: Module not found
```bash
# Solution: Clean install
rm -rf node_modules package-lock.json
npm install
```

**Issue**: Permission errors
```bash
# Solution: Fix npm permissions
sudo chown -R $(whoami) ~/.npm
```

#### Tomcat Issues

**Issue**: Port 8080 already in use
```bash
# Solution: Find and kill process
sudo lsof -i :8080
kill -9 <PID>
```

**Issue**: Cannot access Manager App
```bash
# Solution: Check context.xml and restart
sudo vi webapps/manager/META-INF/context.xml
# Comment out RemoteAddrValve
sudo stopTomcat && sudo startTomcat
```

---

## Quick Reference

### Maven Cheat Sheet

| Command | Description |
|---------|-------------|
| `mvn clean` | Delete target directory |
| `mvn compile` | Compile source code |
| `mvn test` | Run tests |
| `mvn package` | Create JAR/WAR |
| `mvn install` | Install to local repo |
| `mvn deploy` | Deploy to remote repo |
| `mvn clean package -DskipTests` | Build without tests |

### npm Cheat Sheet

| Command | Description |
|---------|-------------|
| `npm install` | Install all dependencies |
| `npm install <pkg>` | Install specific package |
| `npm run <script>` | Run npm script |
| `npm update` | Update dependencies |
| `npm audit` | Check vulnerabilities |
| `npm test` | Run tests |

### .NET CLI Cheat Sheet

| Command | Description |
|---------|-------------|
| `dotnet restore` | Restore dependencies |
| `dotnet build` | Build project |
| `dotnet run` | Run application |
| `dotnet test` | Run tests |
| `dotnet publish` | Publish for deployment |
| `dotnet clean` | Clean build artifacts |

### Tomcat Commands

| Command | Description |
|---------|-------------|
| `startTomcat` | Start Tomcat server |
| `stopTomcat` | Stop Tomcat server |
| `sudo lsof -i :8080` | Check port 8080 |
| Access Manager | `http://localhost:8080/manager` |

---

## Summary

**Build Tools** automate the process of transforming source code into deployable artifacts:

- **Maven**: Java's powerful build automation tool with dependency management
- **npm**: Node.js package manager with script automation
- **.NET CLI**: Microsoft's cross-platform build tools
- **Tomcat**: Java web server for deploying WAR applications

**Key Takeaways:**
1. Build tools manage dependencies, compile code, run tests, and create artifacts
2. Each tool has its own structure and conventions
3. Understanding lifecycles and commands is crucial
4. Configuration files (pom.xml, package.json, .csproj) are the heart of projects
5. Repository management (Maven, npm, NuGet) ensures consistent builds

Master these tools to efficiently build, test, and deploy applications in DevOps pipelines!

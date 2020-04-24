<a href="https://www.automation.lambdatest.com">![LambdaTest Logo](https://www.lambdatest.com/static/images/logo.svg)</a>

# Geb Lambdatest Plugin Sample using Gradle Project

## Running Geb Automation Tests On LambdaTest Selenium Grid

Based on Groovy, Geb is a web testing automation framework. It is build as a wrapper over Selenium WebDriver which makes it ideal for automated browser testing. Geb automation testing framework can help you automate cross browser testing over every browser that is supported by Selenium WebDriver. Geb automation testing framework offers excellent features for web testing, screen scraping, and more. What makes Geb unique is the way it queries your HTML pages, it does so by offering the jQuery-like syntax. Also, it provides built-in support for the Page Object pattern.

LambdaTest integration with Geb will help you execute your Geb test automation scripts on our online Selenium Grid of 2000+ real browsers to ensure your web application renders well on as many browsers as you are targeting to test.


## Prerequisites To Run Geb automation Scripts On Our Selenium Grid

### Environment Setup 

### 1. Java Installation
   
   i.   For Windows :
   
   You can download Java for Windows from [here](http://www.java.com/en/download/manual.jsp)
   
   Run the installer and follow the setup wizard to install Java.
   
   create a new JAVA_HOME environment variable and set variable value as path value to JDK folder.
   
   #### This is Windows environment variable location :
   Control Panel > All Control Panel Items > System > Advanced system settings > Environment Variables
   
   ![altext](https://github.com/keshavissar001/selenide-testng-sample/blob/keshavissar001-patch-1/Img1.png)
   
   ii. For Linux :
   
   use this command :
   ```
   sudo apt-get install openjdk-8-jre
   ```
   iii. For Mac
   
   Java should already be present on Mac OS X by default
   
   ### 2. Maven Installation
   
   Install Maven from [here](https://maven.apache.org/install.html)
   
### 3 Setup

   You can download the file. To do this click on “Clone or download” button. You can download zip file.
   
   Right click on this zip file and extract files in your desired location.

   Go to “geb-lambdatest-master” folder and copy its path.
   Open command prompt and run :
   
    cd <path> (that you have copied)
    
    (please ignore "<" , ">" symbols)
    

![altext](https://github.com/keshavissar001/images/blob/master/SetPathGeb.png)


To clone the file, click on “Clone or download” button and copy the link.

	Then open the command prompt in the folder you want to clone the file. Run the command:

      git clone <paste link here> 
      
 Add your lambdatest `username` and `accessKey` to the `./build.gradle` in “geb-lambdatest-master” in account{} section [For Lambdatest Credentials, Go to Lambdatest Profile Page][account_details] 
      
This is `./build.gradle` file :

```

apply plugin: "idea"
apply plugin: "geb-lambdatest"

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.gebish:geb-gradle:3.3'
    }
}

allprojects {
    apply plugin: "groovy"
}

group 'org.lambdatest'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

ext {
    gebVersion = '3.3'
    seleniumVersion = '3.14.0'
}

dependencies {
    // If using Spock, need to depend on geb-spock
    testCompile "org.gebish:geb-spock:$gebVersion"
    compile "org.spockframework:spock-core:0.7-groovy-2.0"
    compile "org.seleniumhq.selenium:selenium-firefox-driver:$seleniumVersion"
    compile "org.seleniumhq.selenium:selenium-chrome-driver:$seleniumVersion"
}

repositories {
    jcenter()
}

//test task will be used when you run the tests locally, not with cloud
test {
    systemProperty "geb.build.reportsDir", reporting.file("$project.buildDir/test-results/$name/geb")
    systemProperty "geb.env", 'firefox' //select an env/browser for local testing
}

lambdaTest {
    browsers{
        firefox_windows
    }
    task {
//            testClassesDir = rootProject.test.testClassesDir
//            testSrcDirs = rootProject.test.testSrcDirs
//            classpath = rootProject.test.classpath
        systemProperty "geb.build.reportsDir", reporting.file("$project.buildDir/test-results/$name/geb")
    }
     account {
        username = 'lambdatest username'      //Put Your username here
        accessKey = 'lambdatest access token' //Put Your access token here
    }
    tunnelOps {
        tunnelName = 'geb-'+UUID.randomUUID().toString()
    }
}

```
### 5. Running your tests

This is an example of incorporating Geb Lambdatest Plugin into a Gradle build.
The build is setup to work with Firefox Windows combination on Lambdatest Automation Hub. 
To use it with different capabilities, please have a look at  `src/test/resources/GebConfig.groovy` file.


This is the GebConfig.groovy file :

```

import org.openqa.selenium.chrome.ChromeDriver
import org.openqa.selenium.firefox.FirefoxDriver
import org.openqa.selenium.remote.CapabilityType
import org.openqa.selenium.remote.DesiredCapabilities
import org.openqa.selenium.remote.RemoteWebDriver

def lambdaTestBrowser = System.getProperty("geb.lambdatest.browser")
println(lambdaTestBrowser);

if (lambdaTestBrowser) {
    driver = {
        def username = System.getenv("GEB_LAMBDATEST_USERNAME")
        assert username
        def accessKey = System.getenv("GEB_LAMBDATEST_AUTHKEY")
        assert accessKey

        final url = "https://${username}:${accessKey}@hub.lambdatest.com/wd/hub"
        println(url)
        caps = new DesiredCapabilities()
        caps.setCapability(CapabilityType.PLATFORM_NAME,"win10")
        caps.setCapability(CapabilityType.BROWSER_NAME,"firefox")
        caps.setCapability(CapabilityType.BROWSER_VERSION,"latest")
        caps.setCapability("build", "geb-lambdatest")
        def tunnelName = System.getenv("GEB_LAMBDATEST_TUNNELID")
        if(tunnelName != "null"){
            caps.setCapability("tunnel", "true")
            caps.setCapability("tunnelName", tunnelName)
        }
        println(caps)
        new RemoteWebDriver(new URL(url), caps)
    }
}

//Default browser to run on local machine
environments {

    firefox {
        driver = { new FirefoxDriver() }
    }

    chrome {
        driver = { new ChromeDriver() }
    }

}

```

Following tests will run with above mentioned capabilities.

This is the LambdaTestSpec.groovy file :

```

import geb.spock.GebReportingSpec

class LambdaTestSpec extends GebReportingSpec {

    def "Open LambdaTest"() {
        when:
        to LambdaTestPage

        then:
        waitFor { at LambdaTestResultPage }

    }

}

```

This is the LambdaTestPage.groovy file :

```
import geb.Page

class LambdaTestPage extends Page {

    static url = "https://www.lambdatest.com"
    static at = { title == "Cross Browser Testing Tools | Free Automated Website Testing | LambdaTest" }

}

```

This is the LambdaTestResultPage.groovy file :

```

import geb.Page

class LambdaTestResultPage extends Page {
    static at = { title == "Cross Browser Testing Tools | Free Automated Website Testing | LambdaTest" }
}

```


### The following command will launch the tests on lambdatest :

For windows :
```
    gradlew.bat clean allLambdaTestTests --info
```

For Bash/Linux/Mac :
```
   $ gradle clean allLambdaTestTests --info
```

![altext](https://github.com/keshavissar001/images/blob/master/GebResult1.png)
![altext](https://github.com/keshavissar001/images/blob/master/GebResult2.png)


## Questions and issues

Please ask questions on [Lambdatest Support][lambdatest_support] and raise issues on [Lambdatest Community][lambdatest_community].


[lambdatest_support]: https://www.lambdatest.com/support
[lambdatest_community]: https://community.lambdatest.com
[account_details]: https://accounts.lambdatest.com/detail/profile

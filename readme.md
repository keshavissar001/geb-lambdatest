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

These are the screenshots of the output :

![altext](https://github.com/keshavissar001/images/blob/master/GebResult1.png)

![altext](https://github.com/keshavissar001/images/blob/master/GebResult2.png)

##  Testing Locally Hosted or Privately Hosted Projects

To help you perform cross browser testing of your locally stored web pages, LambdaTest provides an SSH(Secure Shell) tunnel connection with the name Lambda Tunnel. With Lambda Tunnel, you can test your locally hosted files before you make them live over the internet. You could even perform cross browser testing from different IP addresses belonging to various geographic locations. You can also use LambdaTest Tunnel to test web-apps and websites that are permissible inside your corporate firewall.

* Set tunnel value to True in test capabilities
> OS specific instructions to download and setup tunnel binary can be found at the following links.
>    - [Windows](https://www.lambdatest.com/support/docs/display/TD/Local+Testing+For+Windows)
>    - [Mac](https://www.lambdatest.com/support/docs/display/TD/Local+Testing+For+MacOS)
>    - [Linux](https://www.lambdatest.com/support/docs/display/TD/Local+Testing+For+Linux)

After setting tunnel you can also see the active tunnel in our LambdaTest dashboard:


![tn](https://github.com/Apoorvlt/test/blob/master/tn.PNG)

### Important Note:
Some Safari & IE browsers, doesn't support automatic resolution of the URL string "localhost". Therefore if you test on URLs like "http://localhost/" or "http://localhost:8080" etc, you would get an error in these browsers. A possible solution is to use "localhost.lambdatest.com" or replace the string "localhost" with machine IP address. For example if you wanted to test "http://localhost/dashboard" or, and your machine IP is 192.168.2.6 you can instead test on "http://192.168.2.6/dashboard" or "http://localhost.lambdatest.com/dashboard".

## About LambdaTest

[LambdaTest](https://www.lambdatest.com/) is a cloud based selenium grid infrastructure that can help you run automated cross browser compatibility tests on 2000+ different browser and operating system environments. LambdaTest supports all programming languages and frameworks that are supported with Selenium, and have easy integrations with all popular CI/CD platforms. It's a perfect solution to bring your [selenium automation testing](https://www.lambdatest.com/selenium-automation) to cloud based infrastructure that not only helps you increase your test coverage over multiple desktop and mobile browsers, but also allows you to cut down your test execution time by running tests on parallel.

## Questions and issues

Please ask questions on [Lambdatest Support][lambdatest_support] and raise issues on [Lambdatest Community][lambdatest_community].


[lambdatest_support]: https://www.lambdatest.com/support
[lambdatest_community]: https://community.lambdatest.com
[account_details]: https://accounts.lambdatest.com/detail/profile

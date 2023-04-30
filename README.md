# Getting started with Serenity and Cucumber

Serenity BDD is a library that makes it easier to write high quality automated acceptance tests, with powerful reporting and living documentation features. It has strong support for both web testing with Selenium, and API testing using RestAssured.

Serenity strongly encourages good test automation design, and supports several design patterns, including classic Page Objects, the newer Lean Page Objects/ Action Classes approach, and the more sophisticated and flexible Screenplay pattern.

The latest version of Serenity supports Cucumber 6.x.

## The starter project
The best place to start with Serenity and Cucumber is to clone or download the starter project on Github ([https://github.com/DishaOwitechs/Whales-ILIP](https://github.com/DishaOwitechs/Whales-ILIP)). This project gives you a basic project setup, along with some sample tests and supporting classes.

### The project directory structure
The project has build scripts for both Maven and Gradle, and follows the standard directory structure used in most Serenity projects:
```Gherkin
src
  + main
  + test
    + java                        Test runners and supporting code
    + resources
      + features                  Feature files
     + search                  Feature file subdirectories 
             Login.feature
```

Serenity 2.2.13 introduced integration with WebdriverManager to download webdriver binaries.

## The sample scenario
Both variations of the sample project uses the sample Cucumber scenario. In this scenario, User is performing a Signup operation on the Whales Website:

```Gherkin
Feature: Invalid Data for Login

  Scenario: Login with invalid data
    Given web URL
    When enter email address
    Then enter first name
    Then enter last name and click on check box
    And click on Signup button
```

### The Screenplay implementation
The sample code in the master branch uses the Screenplay pattern. The Screenplay pattern describes tests in terms of actors and the tasks they perform. Tasks are represented as objects performed by an actor, rather than methods. This makes them more flexible and composable, at the cost of being a bit more wordy. Here is an example:
```java
    @Given("web URL")
    public void web_URL() throws Throwable {
      //Setting system properties of FirefoxDriver
      
    }

    @When("enter email address")
    public void enter_email_address() throws Throwable {
        //email > name locator for text box
    }

    @Then("enter first name")
    public void enter_first_name() throws Throwable {
        //firstname > name locator for text box

    }

    @Then("enter last name and click on check box")
    public void enter_last_name_and_click_on_check_box() throws Throwable {
        //lastname > name locator for text box

    }

    @Then("click on Signup button")
    public void click_on_Signup_button() throws Throwable {
        //id locator for button
        
    }
```

## Executing the tests
To run the sample project, you can either just run the `CucumberTestSuite` test runner class, or run either `mvn verify` or `gradle test` from the command line.

By default, the tests will run using Chrome. You can run them in Firefox by overriding the `driver` system property, e.g.
```json
$ mvn clean verify -Ddriver=firefox
```
Or
```json
$ gradle clean test -Pdriver=firefox
```

The test results will be recorded in the `target/site/serenity` directory.

## Generating the reports
Since the Serenity reports contain aggregate information about all of the tests, they are not generated after each individual test (as this would be extremenly inefficient). Rather, The Full Serenity reports are generated by the `serenity-maven-plugin`. You can trigger this by running `mvn serenity:aggregate` from the command line or from your IDE.

They reports are also integrated into the Maven build process: the following code in the `pom.xml` file causes the reports to be generated automatically once all the tests have completed when you run `mvn verify`?

```
             <plugin>
                <groupId>net.serenity-bdd.maven.plugins</groupId>
                <artifactId>serenity-maven-plugin</artifactId>
                <version>${serenity.maven.version}</version>
                <configuration>
                    <tags>${tags}</tags>
                </configuration>
                <executions>
                    <execution>
                        <id>serenity-reports</id>
                        <phase>post-integration-test</phase>
                        <goals>
                            <goal>aggregate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

## Simplified WebDriver configuration and other Serenity extras
The sample projects both use some Serenity features which make configuring the tests easier. In particular, Serenity uses the `serenity.conf` file in the `src/test/resources` directory to configure test execution options.  
### Webdriver configuration
The WebDriver configuration is managed entirely from this file, as illustrated below:
```java
webdriver {
     driver = firefox

 #// Add the WebDriver proxy capability.
 #   Proxy proxy = new Proxy();
 #   proxy.setHttpProxy("myhttpproxy:3337");
 #   options.setCapability("proxy", proxy);

 #  // Add a ChromeDriver-specific capability.
 #   options.addExtensions(new File("/path/to/extension.crx"));
 #   ChromeDriver driver = new ChromeDriver(options);
 #   driver = chrome

        capabilities {
                browserName = "firefox"
                acceptInsecureCerts = true
                "goog:chromeOptions" {
                args = ["remote-allow-origins=*","test-type", "no-sandbox", "ignore-certificate-errors", "--window-size=1000,800",
                    "incognito", "disable-infobars", "disable-gpu", "disable-default-apps", "disable-popup-blocking"]
                }

        #
        # Chrome options can be defined using the chrome.switches property
        #
        chrome.switches="""--start-maximized;--test-type;--no-sandbox;--ignore-certificate-errors;
                           --disable-popup-blocking;--disable-default-apps;--disable-extensions-file-access-check;
                           --incognito;--disable-infobars,--disable-gpu"""
        #
        # Define drivers for different platforms. Serenity will automatically pick the correct driver for the current platform
        #
        
        drivers {
          windows {
            webdriver.chrome.driver = "src/test/resources/webdrivers/windows/chromedriver.exe"
            webdriver.gecko.driver = "src/test/resources/webdrivers/windows/geckodriver.exe"
          }
          linux {
            webdriver.chrome.driver = "src/test/resources/webdrivers/linux/chromedriver"
            webdriver.gecko.driver = "src/test/resources/webdrivers/linux/geckodriver"
          }
        }


    }
```

Serenity uses WebDriverManager to download the WebDriver binaries automatically before the tests are executed.

### Environment-specific configurations > specially used for high scale project
We can also configure environment-specific properties and options, so that the tests can be run in different environments. Here, we configure three environments, __dev__, _staging_ and _prod_, with different starting URLs for each:
```json
environments {
  default {
    webdriver.base.url = "https://apps.usewhale.io/trial"
  }
  dev {
    webdriver.base.url = "https://apps.usewhale.io/trial"
  }
  staging {
    webdriver.base.url = "https://apps.usewhale.io/trial/staging"
  }
  prod {
    webdriver.base.url = "https://apps.usewhale.io/trial/prod"
  }
}
```

You use the `environment` system property to determine which environment to run against. For example to run the tests in the staging environment, you could run:
```json
$ mvn clean verify -Denvironment=staging
```

See [**this article**](https://johnfergusonsmart.com/environment-specific-configuration-in-serenity-bdd/) for more details about this feature.

## Want to learn more?
For more information about Serenity BDD, you can read the [**Serenity BDD Book**](https://serenity-bdd.github.io/theserenitybook/latest/index.html), the official online Serenity documentation source. Other sources include:
* **[Learn Software Automation Testing using Serenity BDD Online](https://ilip.owitechs.com/top-job-ready-programs-information-technology/#)** with online courses from ILIP Job Ready Program powered by Owitechs
* ** Follow for Course and Internship on [LinkedIn](https://www.linkedin.com/company/owitechs-pty-ltd/) and [Facebook](https://www.facebook.com/Owitechs/)
* ** Follow for Daily updates on [Instagram](https://www.instagram.com/owitechs_pty_ltd/)

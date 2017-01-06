# An example of how to use Jenkins to build a maven selenium Junit test
 In this blog, I am going to state:
 
  1) how to use maven to run a selenium test which was written in Java language.
  
  2) how to use Jenkins to build this maven project.

## Download and install Maven
Download and install maven following the instruction on <https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html>. 
To check Maven is correctly installed or not, you can open a terminal and run the following command:

        mvn -version
The output should be the same as:
        
        Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00) 
        Maven home: /home/vagrant/apache-maven-3.3.9
## Quickly create a new Maven project
On command line, execute the follwoing Maven goal:

        mvn archetype:generate -DgroupId=com.wu.excample -DartifactId=firsttest -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
A quick start "FirstTest" application is created by maven using archetype to generate goal with given groupid and artifactid. After the command executed successfully, a directory "FirstTest" is created under the current working directory with the following file and sub-directories:

        .
        ./pom.xml
        ./src
        ./src/test
        ./src/test/java
        ./src/test/java/com
        ./src/test/java/com/wu
        ./src/test/java/com/wu/example
        ./src/test/java/com/wu/example/AppTest.java
        ./src/main
        ./src/main/java
        ./src/main/java/com
        ./src/main/java/com/wu
        ./src/main/java/com/wu/example
        ./src/main/java/com/wu/example/App.java

Make sure all of the following commands in this article are executed under the "FirstTest" directory.

I will add a selenium test class and use JUnit as the test runner to run the selenium test.

- Clean up

  Delete the example class "/src/test/java/com/wu/example/AppTest.java"

- Create a new file "/src/test/java/com/wu/example/FirstTest.java" with the following content:

        package com.wu.example;

        import org.junit.*;
        import org.openqa.selenium.By;
        import org.openqa.selenium.WebDriver;
        import org.openqa.selenium.WebElement;
        import org.openqa.selenium.firefox.FirefoxDriver;
        import org.openqa.selenium.support.ui.ExpectedCondition;
        import org.openqa.selenium.support.ui.WebDriverWait;

        public class FirstTest {
          static WebDriver driver;

          @BeforeClass
          public static void SetUp(){
              driver = new FirefoxDriver();
              driver.get("http://www.google.com");
          }

          @Test
          public void displaySearchResult(){
              WebElement element = driver.findElement(By.name("q"));
              element.sendKeys("Cheese!");
              element.submit();
              System.out.println("Page title is: " + driver.getTitle());
              (new WebDriverWait(driver, 10)).until(new ExpectedCondition<Boolean>() {
              public Boolean apply(WebDriver d) {
                return d.getTitle().toLowerCase().startsWith("cheese!");
                   }
             });
          }
          
          @AfterClass
          public static void cleanup(){
              System.out.println("Page title is: " + driver.getTitle());
              driver.quit();
          }
         }

- Edit "pom.xml" file:
   
   1.Change the Junit version;

     Since in the FirstTest.java code snippet, some JUnit annotations are used to setup some test hooks. And these annotations are only supported by Junit 4, so I need to update the junit version to 4.12 in the "pom.xml" file junit dependency section:

        <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.12</version>
          <scope>test</scope>
        </dependency>

    2.Add the following selenium driver as a dependency in the "pom.xml" file dependencis section:

        <dependency>
          <groupId>org.seleniumhq.selenium</groupId>
          <artifactId>selenium-java</artifactId>
          <version>2.52.0</version>
          <scope>test</scope>
        </dependency>
## Run the test:

- On command line "FirstTest" directory, execute the following command:

        mvn clean test

- The test result should be like this:

        [INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ firsttest ---
        [INFO] Surefire report directory: /home/vagrant/maven/firsttest/target/surefire-reports

        -------------------------------------------------------
         T E S T S
        -------------------------------------------------------
        Running com.wu.example.FirstTest
        Page title is: Google
        Page title is: Cheese! - Google Search
        Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 19.624 sec

        Results :

        Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

        [INFO] ------------------------------------------------------------------------
        [INFO] BUILD SUCCESS
        [INFO] ------------------------------------------------------------------------
        [INFO] Total time: 33.338 s
        [INFO] Finished at: 2016-03-01T15:55:03+00:00
        [INFO] Final Memory: 15M/38M
        [INFO] ------------------------------------------------------------------------
## Build test using Jenkins:
- Download Jenkins.war and move it to C:\Jenkins\ directory
- Run and launch Jenkins
  -- Execute the following command:
        
        Java -jar C:\Jenkins\Jenkins.war
  
  Jenkins is running locally on my machine. 
  Note: Jenkins instruction with a temporary password is also displayed on the command prompt window.
  -- Open browser and enter "Localhost:8080", Jenkins page is displayed.
  -- Login:
  
        User: admin        Password: temporary password

  -- Create a new job with the name "FirstMavenTest";
     on the configure page "Build" section, select "Invoke top-level Maven targets" build step and:
     
     . Enter "test" to the "Goals" textbox;
     
     . Enter pom.xml directory path to the "POM" textbox.
     
     . Save it

  -- Build the new project

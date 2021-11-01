# From Java to Python for Web UI Tests

## Part I – Basic setup for Python project + intro Python and Selene

* open project in PyCharm

  * clone https://github.com/autotests-cloud/qa_guru_5_9_jenkins_owner/
  * `git checkout -t origin/refactored-by-commits-per-idea`
  * `git checkout -b switch-to-python 9dd35df4209c657e4e2585fdbb7183d8ed0dd90d  `
    * = new branch from commit https://github.com/autotests-cloud/qa_guru_5_9_jenkins_owner/commit/9dd35df4209c657e4e2585fdbb7183d8ed0dd90d
  * open it in PyCharm

* refactor build.gradle

  * remove: 

    * plugins
    * allure {…} config
    * repositories
      * show https://pypi.org/ as main package repository
        * instead of https://search.maven.org/
    * variables
    * tasks.withType(JavaCompile)
      * python is not compiled lang…
      * skip sourceCompatibility for simplicity
        * but you can simulate this via linters
          * and PyCharm also checks this via Editor/Inspections/Python/Code Compatibility inspection
      * utf8 is default, no need to specify
    * systemProperties(System.getProperties())
      * python just have direct access to OS with all envs
    * testLogging
      * let's just use defaults
    * useJunitPlatform
      * just works in Python:)
        * But in PyCharm you have to set **Preferences/Tools/Python Integrated Tools/Testing/Default test runner** to corresponding test runner, in our case we will use **pytest** (you can set it later…)

  * refactor dependencies to the format of `pyproject.toml ` file

    * `[tool.poetry.dependecies]`

      * opens new block

    * rename file to `pyproject.toml`

    * remove testRuntimeOnly

      * not actual in not compiled lang

    * but can separate .dependencies from .dev-dependencies

      * but no need, because our project is already a test project;)

    * from `"group:name:number"` to `name = "^number"`

      * any not upgraded from major number
      * remove all groups (in python just libraries names are used, «group-ids» are not relevant)

    * remove

      * aspectjweaver
        * in java was needed for some logging magic, that in python we achieve in a different way

    * selenide => selene

      * = {versioin = "^2.0.0-alpha38", allow-prereleases = true}

    * javafaker => Faker

    * owner => pydantic + python-dotenv

    * junit => pytest + pytest-xdist (parallelisation)

    * allure-selenide => allure-pytest

    * here we also add python dependency… 

      * python = "^3.8"

    * finalize versions, just copy&paste (relevant for 29.10.2021):

      ```toml
      [tool.poetry.dependencies]
      python = "^3.8"
      selene = {version = "^2.0.0-alpha.40", allow-prereleases = true}
      pytest = "^6.2.5"
      pytest-xdist = "^2.4.0"
      allure-pytest = "^2.9.45"
      pydantic = "^1.8.2"
      python-dotenv = "^0.19.1"
      Faker = "^9.7.1"
      ```

      

  * add required project properties

    * ```toml
      [tool.poetry]
      name = "python-web-test-kit"
      version = "0.1.0"
      description = "python + pytest + selene + allure web ui tests project template"
      authors = ["Iakiv Kramarenko <yashaka@gmail.com>"]
      ```

* install everything

  * remove if anything still exist: .gradle, gradle, gradlew, gradlew.bat

  * according to added to README.MD

    * ````markdown
      # Initial Project Setup
      
      ## Prerequisites
      * pyenv
        ** python 3.8.2
      * poetry
      
      ## Setup
      
      Open terminal in your project folder and do:
      
      ```bash
      pyenv local 3.8.2
      poetry install
      ```
      
      Set a project interpreter in your editor to: `"$(poetry env info -p)/bin/python"`, and you should be ready to go;)
      ````

    * pyenv local 3.8.2

    * poetry install

      * **poetry** is the tool that we use in python instead of **gradle**

    * set path to interpreter

      * in PyCharm: Preferences/Project/Python Interpreter/Python Interpreter ⚙️/Add…/Existing  environment/`"$(poetry env info -p)/bin/python"`

    * ensure pytest is set as test runner in PyCharm at:

      * Preferences/Tools/Python Integrated Tools/Testing/Default test runner

  * check it works: 

    * create demo_web_script.py

      ```python
      from selene.support.shared import browser
      
      browser.config.hold_browser_open = True
      
      browser.open("https://google.com/ncr")
      browser.element("[name=q]").type("qa guru").press_enter()
      ```
      or
      ```python
      from selene.support.shared import browser
      from selene.support.shared.jquery_style import s
      
      browser.config.hold_browser_open = True
      
      browser.open("https://google.com/ncr")
      s("[name=q]").type('qa guru').press_enter()
      ```

      * notice `browser.open` vs `open`, `browser.element` & `s` vs `$`

* refactor structure

  * test/ to top level and rename it to `tests`
  * resources/ to top level
  * confg/, helpers/ to <project_name_written_in_snake_case>/
    * that is analog to **main** in java
  * add `__init.py__` to all "folders with code aka python packages"
    * same as index.js as in JavaScript
    * but is kind of mandatory in Python for **regular packages** ([question on stackoverflow](https://stackoverflow.com/questions/37139786/is-init-py-not-required-for-packages-in-python-3-3))
  * when project is generated from scratch (`poetry init`, the structure will already be precreated;)

* start refactor StudentRegistrationForm.java

  * keep it as .java

  * imports

    * to `from faker import Faker`

    * remove `@Test` import

      * refactor test names (add test_, and refactor to snake_case)

    * static imports are evil;) => `have, be, by`

      * with exceptions to $ & s
      
    * Selenide => selene.support.shared.browser

    * => `from allure import step`

    * let's add `s` import to use `s` instead of `$` already

    * corresponding bulk replacements

      * first remove all comments (explaining…)

        * `// ...` is `#`

        * 
          ```java
          /* 
          ... 
          */
          ```
      
          can be simulated with syntax for multi-line strings in Python (they are regulary used as docsctrings)
      
          ```python
          """
          ...
          """
          ```
      
      * `;\n` to `\n`
      
      * `Have\(text\(` to `(have.text(`
      
      * `(\$|\$x)\(`to `s()`
      
      * `byText` to `by.text` 
      
      * `.parent\(\)` to `.element("..")` or  `.s("..")`
      
      * `.val` to `.set_value`
      
      * everything:
      
        ```python
        public class StudentRegistrationFormTests extends ConfTest {
        
            Faker faker = new Faker()
        
            String firstName = faker.name().firstName(),
                    lastName = faker.name().lastName(),
                    email = faker.internet().emailAddress(),
                    gender = "Other",
                    mobile = faker.number().digits(10),
                    dayOfBirth = "10",
                    monthOfBirth = "May",
                    yearOfBirth = "1988",
                    subject1 = "Chemistry",
                    subject2 = "Commerce",
                    hobby1 = "Sports",
                    hobby2 = "Reading",
                    hobby3 = "Music",
                    picture = "1.png",
                    currentAddress = faker.address().fullAddress(),
                    state = "Uttar Pradesh",
                    city = "Merrut"
        
            void successfulFillFormTest() {
                step("Open students registration form", () -> {
                    browser.open("https://demoqa.com/automation-practice-form")
                    s(".practice-form-wrapper").should(have.text("Student Registration Form"))
                })
        
                step("Fill students registration form", () -> {
                    step("Fill common data", () -> {
                        s("#firstName").set_value(firstName)
                        s("#lastName").set_value(lastName)
                        s("#userEmail").set_value(email)
                        s("#genterWrapper").s(by.text(gender)).click()
                        s("#userNumber").set_value(mobile)
                    })
                    step("Set date", () -> {
                        s("#dateOfBirthInput").clear()
                        s(".react-datepicker__month-select").selectOption(monthOfBirth)
                        s(".react-datepicker__year-select").selectOption(yearOfBirth)
                        s(".react-datepicker__day--0" + dayOfBirth).click()
                    })
                    step("Set subjects", () -> {
                        s("#subjectsInput").set_value(subject1)
                        s(".subjects-auto-complete__menu-list").s(by.text(subject1)).click()
                        s("#subjectsInput").set_value(subject2)
                        s(".subjects-auto-complete__menu-list").s(by.text(subject2)).click()
                    })
                    step("Set hobbies", () -> {
                        s("#hobbiesWrapper").s(by.text(hobby1)).click()
                        s("#hobbiesWrapper").s(by.text(hobby2)).click()
                        s("#hobbiesWrapper").s(by.text(hobby3)).click()
                    })
                    step("Upload image", () ->
                            s("#uploadPicture").uploadFromClasspath("img/" + picture))
                    step("Set address", () -> {
                        s("#currentAddress").set_value(currentAddress)
                        s("#state").scrollTo().click()
                        s("#stateCity-wrapper").s(by.text(state)).click()
                        s("#city").click()
                        s("#stateCity-wrapper").s(by.text(city)).click()
                    })
                    step("Submit form", () ->
                        s("#submit").click())
                })
        
                step("Verify successful form submit", () -> {
                    s("#example-modal-sizes-title-lg").should(have.text("Thanks for submitting the form"))
                    s("//td[text()='Student Name']").s("..").should(have.text(firstName + " " + lastName))
                    s("//td[text()='Student Email']").s("..").should(have.text(email))
                    s("//td[text()='Gender']").s("..").should(have.text(gender))
                    s("//td[text()='Mobile']").s("..").should(have.text(mobile))
                    s("//td[text()='Date of Birth']").s("..").should(have.text(dayOfBirth + " " + monthOfBirth + "," + yearOfBirth))
                    s("//td[text()='Subjects']").s("..").should(have.text(subject1 + ", " + subject2))
                    s("//td[text()='Hobbies']").s("..").should(have.text(hobby1 + ", " + hobby2 + ", " + hobby3))
                    s("//td[text()='Picture']").s("..").should(have.text(picture))
                    s("//td[text()='Address']").s("..").should(have.text(currentAddress))
                    s("//td[text()='State and City']").s("..").should(have.text(state + " " + city))
                })
            }
        
            void negativeFillFormTest() {
                step("Open students registration form", () -> {
                    browser.open("https://demoqa.com/automation-practice-form")
                    s(".practice-form-wrapper").should(have.text("Student Registration Form"))
                })
        
                step("Fill students registration form", () -> {
                    step("Fill common data", () -> {
                        s("#firstName").set_value(firstName)
                        s("#lastName").set_value(lastName)
                        s("#userEmail").set_value(email)
                        s("#genterWrapper").s(by.text(gender)).click()
                        s("#userNumber").set_value(mobile)
                    })
                    step("Set date", () -> {
                        s("#dateOfBirthInput").clear()
                        s(".react-datepicker__month-select").selectOption(monthOfBirth)
                        s(".react-datepicker__year-select").selectOption(yearOfBirth)
                        s(".react-datepicker__day--0" + dayOfBirth).click()
                    })
                    step("Set subjects", () -> {
                        s("#subjectsInput").set_value(subject1)
                        s(".subjects-auto-complete__menu-list").s(by.text(subject1)).click()
                        s("#subjectsInput").set_value(subject2)
                        s(".subjects-auto-complete__menu-list").s(by.text(subject2)).click()
                    })
                    step("Set hobbies", () -> {
                        s("#hobbiesWrapper").s(by.text(hobby1)).click()
                        s("#hobbiesWrapper").s(by.text(hobby2)).click()
                        s("#hobbiesWrapper").s(by.text(hobby3)).click()
                    })
                    step("Upload image", () ->
                            s("#uploadPicture").uploadFromClasspath("img/" + picture))
                    step("Set address", () -> {
                        s("#currentAddress").set_value(currentAddress)
                        s("#state").scrollTo().click()
                        s("#stateCity-wrapper").s(by.text(state)).click()
                        s("#city").click()
                        s("#stateCity-wrapper").s(by.text(city)).click()
                    })
                    step("Submit form", () ->
                            s("#submit").click())
                })
        
                step("Verify successful form submit", () -> {
                    s("#example-modal-sizes-title-lg").should(have.text("Thanks for submitting the form"))
                    s("//td[text()='Student Name']").s("..").should(have.text(firstName + " " + lastName))
                    s("//td[text()='Student Email']").s("..").should(have.text(email))
                    s("//td[text()='Gender']").s("..").should(have.text(gender))
                    s("//td[text()='Mobile']").s("..").should(have.text(mobile))
                    s("//td[text()='Date of Birth']").s("..").should(have.text(dayOfBirth + " " + monthOfBirth + "," + yearOfBirth))
                    s("//td[text()='Subjects']").s("..").should(have.text(subject1 + ", " + subject2))
                    s("//td[text()='Hobbies']").s("..").should(have.text(hobby1 + ", " + hobby2 + ", " + hobby3))
                    s("//td[text()='Picture']").s("..").should(have.text(picture))
                    s("//td[text()='Address']").s("..").should(have.text(currentAddress))
                    s("//td[text()='State and City']").s("..").should(have.text(state + " error " + city))
                })
            }
        }
        ```
      
        

  * classes and class fields

    * everything is `public`

      * `_underscore_before_name` just simulates `privacy`, but yet you can access this entity
      * thouth editors like PyCharm lastly stopped to autocomplete such methods

    * `class` is same

    * `extends Parent` is `(Parent)`

    * `TestBase.java` can be changed to `conftest.py` that is automatic per folder

    * `{/*code*/}` are 

      ```python
      :
          # code
      ```

    * `Type field` is `field: Type` 

      * and typing is optional 
        * kind of by default – same as `var` in Java
      * usuful  for Self-Documented Code practice and Autocomplete

    * `new Clazz()` is `Clazz()` 

      * `new` is limiting btw, see [Eric Elliot's article](https://medium.com/javascript-scene/why-composition-is-harder-with-classes-c3e627dcd0aa) ;)

    * `// ...` is `#`

    * `/* ... */` can be simulated with `"""..."""`

    * `facker.*` to `faker.first_name()`, `faker.last_name()`, `faker.email`, `faker.random_number(digits=10)`, `faker.street_address`

      * autocomplete doesn't work :(

    * camelCase should be snake_case

  * functions

    * `void` does not exist in Python
      * instead of `void` we have to specify javish' `null` that is `None` in python, after fn_name() after `->` , like `fn() -> None:`
        * again, typing is optional;)
    * `def` is mandatory: `def fn():`
    * `self` is needed as first param for methods of class objects (`def method(self):`)
      * without `self` the method will be considered a "class method", kind of "static method in Java"
        * moreover - it's signature would be not fully valid, to finalize it, you have to add @staticmethod before method signature

  * lambdas

    * `() ->` to `lambda:`

      * but lambdas are «one-statement» only
      * workaround via `lambda: [first_call(), second_call()]`
        * `Arrays.asList("a", "b", "c")` to `["a", "b", "c"]`
        * but it's not «The Pythonic Way»
          * better - use `def` instead of lambda

    * function decorators

      ```python
      # python_web_test_kit/helpers/reporting.py
      from allure import step as allure_step
      
      
      def step(title):
          def decorator(fn):
              return allure_step(title)(fn)()
      
          return decorator
      ```
    
      
    
      ```python
      class StudentRegistrationFormTests:
      
          faker = Faker()
      
          first_name = faker.first_name()
          last_name = faker.last_name()
          email = faker.email()
          gender = "Other"
          mobile = faker.random_number(digits=10)
          day_of_birth = "10"
          month_of_birth = "May"
          year_of_birth = "1988"
          subject1 = "Chemistry"
          subject2 = "Commerce"
          hobby1 = "Sports"
          hobby2 = "Reading"
          hobby3 = "Music"
          picture = "1.png"
          current_address = faker.street_address()
          state = "Uttar Pradesh"
          city = "Merrut"
      
          def test_successful_fill_form_test(self):
              @step("Open students registration form")
              def _():
                  browser.open("https://demoqa.com/automation-practice-form")
                  s(".practice-form-wrapper").should(have.text("Student Registration Form"))
      
              @step("Fill students registration form")
              def _():
                  @step("Fill common data")
                  def _():
                      s("#first_name").set_value(self.first_name)
                      s("#last_name").set_value(self.last_name)
                      s("#userEmail").set_value(self.email)
                      s("#genterWrapper").s(by.text(self.gender)).click()
                      s("#userNumber").set_value(self.mobile)
      
                  @step("Set date")
                  def _():
                      s("#dateOfBirthInput").clear()
                      s(".react-datepicker__month-select").selectOption(self.month_of_birth)
                      s(".react-datepicker__year-select").selectOption(self.year_of_birth)
                      s(".react-datepicker__day--0" + self.day_of_birth).click()
      
                  @step("Set subjects")
                  def _():
                      s("#subjectsInput").set_value(self.subject1)
                      s(".subjects-auto-complete__menu-list").s(by.text(self.subject1)).click()
                      s("#subjectsInput").set_value(self.subject2)
                      s(".subjects-auto-complete__menu-list").s(by.text(self.subject2)).click()
      
                  @step("Set hobbies")
                  def _():
                      s("#hobbiesWrapper").s(by.text(self.hobby1)).click()
                      s("#hobbiesWrapper").s(by.text(self.hobby2)).click()
                      s("#hobbiesWrapper").s(by.text(self.hobby3)).click()
      
                  @step("Upload image")
                  def _():
                      s("#uploadPicture").uploadFromClasspath("img/" + self.picture)
      
                  @step("Set address")
                  def _():
                      s("#current_address").set_value(self.current_address)
                      s("#state").scrollTo().click()
                      s("#stateCity-wrapper").s(by.text(self.state)).click()
                      s("#city").click()
                      s("#stateCity-wrapper").s(by.text(self.city)).click()
      
                  @step("Submit form")
                  def _():
                      s("#submit").click()
      
              @step("Verify successful form submit")
              def _():
                  s("#example-modal-sizes-title-lg").should(have.text("Thanks for submitting the form"))
                  s("//td[text()='Student Name']").s("..").should(have.text(self.first_name + " " + self.last_name))
                  s("//td[text()='Student Email']").s("..").should(have.text(self.email))
                  s("//td[text()='Gender']").s("..").should(have.text(self.gender))
                  s("//td[text()='Mobile']").s("..").should(have.text(self.mobile))
                  s("//td[text()='Date of Birth']").s("..").should(
                      have.text(self.day_of_birth + " " + self.month_of_birth + "," + self.year_of_birth))
                  s("//td[text()='Subjects']").s("..").should(have.text(self.subject1 + ", " + self.subject2))
                  s("//td[text()='Hobbies']").s("..").should(have.text(self.hobby1 + ", " + self.hobby2 + ", " + self.hobby3))
                  s("//td[text()='Picture']").s("..").should(have.text(self.picture))
                  s("//td[text()='Address']").s("..").should(have.text(self.current_address))
                  s("//td[text()='State and City']").s("..").should(have.text(self.state + " " + self.city))
      
          def test_negative_fill_form_test(self):
              @step("Open students registration form")
              def _():
                  open("https://demoqa.com/automation-practice-form")
                  s(".practice-form-wrapper").should(have.text("Student Registration Form"))
      
              @step("Fill students registration form")
              def _():
                  @step("Fill common data")
                  def _():
                      s("#first_name").set_value(self.first_name)
                      s("#last_name").set_value(self.last_name)
                      s("#userEmail").set_value(self.email)
                      s("#genterWrapper").s(by.text(self.gender)).click()
                      s("#userNumber").set_value(self.mobile)
      
                  @step("Set date")
                  def _():
                      s("#dateOfBirthInput").clear()
                      s(".react-datepicker__month-select").selectOption(self.month_of_birth)
                      s(".react-datepicker__year-select").selectOption(self.year_of_birth)
                      s(".react-datepicker__day--0" + self.day_of_birth).click()
      
                  @step("Set subjects")
                  def _():
                      s("#subjectsInput").set_value(self.subject1)
                      s(".subjects-auto-complete__menu-list").s(by.text(self.subject1)).click()
                      s("#subjectsInput").set_value(self.subject2)
                      s(".subjects-auto-complete__menu-list").s(by.text(self.subject2)).click()
      
                  @step("Set hobbies")
                  def _():
                      s("#hobbiesWrapper").s(by.text(self.hobby1)).click()
                      s("#hobbiesWrapper").s(by.text(self.hobby2)).click()
                      s("#hobbiesWrapper").s(by.text(self.hobby3)).click()
      
                  @step("Upload image")
                  def _():
                      s("#uploadPicture").uploadFromClasspath("img/" + self.picture)
      
                  @step("Set address")
                  def _():
                      s("#current_address").set_value(self.current_address)
                      s("#state").scrollTo().click()
                      s("#stateCity-wrapper").s(by.text(self.state)).click()
                      s("#city").click()
                      s("#stateCity-wrapper").s(by.text(self.city)).click()
      
                  @step("Submit form")
                  def _():
                      s("#submit").click()
      
              @step("Verify successful form submit")
              def _():
                  s("#example-modal-sizes-title-lg").should(have.text("Thanks for submitting the form"))
                  s("//td[text()='Student Name']").s("..").should(have.text(self.first_name + " " + self.last_name))
                  s("//td[text()='Student Email']").s("..").should(have.text(self.email))
                  s("//td[text()='Gender']").s("..").should(have.text(self.gender))
                  s("//td[text()='Mobile']").s("..").should(have.text(self.mobile))
                  s("//td[text()='Date of Birth']").s("..").should(
                      have.text(self.day_of_birth + " " + self.month_of_birth + "," + self.year_of_birth))
                  s("//td[text()='Subjects']").s("..").should(have.text(self.subject1 + ", " + self.subject2))
                  s("//td[text()='Hobbies']").s("..").should(have.text(self.hobby1 + ", " + self.hobby2 + ", " + self.hobby3))
                  s("//td[text()='Picture']").s("..").should(have.text(self.picture))
                  s("//td[text()='Address']").s("..").should(have.text(self.current_address))
                  s("//td[text()='State and City']").s("..").should(have.text(self.state + " error " + self.city))
      
      ```
    
      
    
    * contextmanagers are also relevant to this case, might be easier to understand for someone in context of implementation, but would be less KISS/simple for now, less fundamental

* Low level commands

  * `.uploadFromClasspath("img/" + picture)` to `.type(os.path.abspath("../resources/img/" + picture))`
    * yet it's a weak solution, because it depends from where we run tesets…
      * if we run tests from the folder `project/tests` then it will work
      * if we run tests from the folder `project` then we have to change it to  `.type(os.path.abspath("./resources/img/" + picture))`
  * `scrollTo` to `.perform(command.js.scroll_into_view)`
    * it's longer, but it's also explicit, emphasizing the fact that we are using workaround vis JS.

* PageObjects as Components

  * `.selectOption` to `Dropdown.select`: 
    `DropDown(".react-datepicker__month-select").select(self.month_of_birth)`
    where:
    
    ```python
    # python_web_test_kit/model/controls.py
    
    from selene import have
    from selene.support.shared.jquery_style import s
    
    
    class DropDown:
        def __init__(self, selector: str):
            self.selector = selector
    
        def select(self, text):
            s(self.selector).click()
            s(self.selector).all("option").element_by(have.text(text)).click()
    ```

* FIX: `mobile = faker.random_number(digits=10)` to `mobile = str(faker.random_number(digits=10))`

  * python has dynamic typing but yet it's a strict typing, you can't use a digit where the string is expected
  * that's why using `self.mobile` at `have.text(self.mobile)` will fail, because internally `have.text` expects passed arg to be a string. And we passed a digit… So we have to convert it explicitely to `str`.

* P.S. modules over classes

  * we can remove class at all, leaving just functions insde a python module (python file)

* P.S. `'single looks cleaner and is easier to type'` than `"double that needs extra shift pressed"` ;)

* P.S. string interpolation

* simple pytest fixtures

  ```python
  # conftest.py
  import pytest
  from selene.support.shared import browser
  
  
  @pytest.fixture(scope="function", autouse=True)
  def manage_browser():
      browser.config.timeout = 3.0
  
      yield
  
      browser.quit()
  ```

  

## Part II – Project Configuration

* project configuration
  from

  ```java
  package config;
  
  import org.aeonbits.owner.Config;
  
  @Config.LoadPolicy(Config.LoadType.MERGE)
  @Config.Sources({
          "system:properties",
          "classpath:config/driver.properties",
  })
  public interface ProjectConfig extends Config {
      @DefaultValue("chrome")
      String browser();
    
      @DefaultValue(3)
      int timeout();
  
      String remoteWebDriverUrl();
  
      String remoteWebDriverUser();
  
      String remoteWebDriverPassword();
  
      String videoStorage();
  }
  ```
  
  ```java
  package config;
  
  import org.aeonbits.owner.ConfigFactory;
  
  public class Project {
      public static ProjectConfig config = ConfigFactory.create(ProjectConfig.class);
  }
  ```
  
  ```properties
  # driver.properties
  remoteWebDriverUser=superhero
  remoteWebDriverPassword=secret
  ```
  
  to
  
  ```python
  # config.py
  from typing import Optional
  
  import pydantic
  
  
  class Settings(pydantic.BaseSettings):
      browser: str = 'chrome'
      timeout: float = 3.0
      remote_webdriver_url: Optional[str] = None
      remote_webdriver_user: Optional[str] = None
      remote_webdriver_password: Optional[str] = None
      video_storage: Optional[str] = None
  
  
  settings = Settings(_env_file='config.env')
  
  ```

  ```properties
  # config.env
  remote_webdriver_user=superhero
  remote_webdriver_password=secret
  ```
  
* base test configuration
  from

  ```java
  public class TestBase {
  
      @BeforeAll
      static void setup() {
          // SKIPPED FOR NOW
          // SelenideLogger.addListener("AllureSelenide", new AllureSelenide());
  
          DesiredCapabilities capabilities = new DesiredCapabilities();
          capabilities.setCapability("enableVNC", true);
          capabilities.setCapability("enableVideo", true);
          Configuration.browserCapabilities = capabilities;
  
          Configuration.browser = Project.config.browser();
          System.out.println("remoteWebDriver: " + Project.config.remoteWebDriverUrl());
  
          if(Project.config.remoteWebDriverUrl() != null) {
              String user = Project.config.remoteWebDriverUser();
              String password = Project.config.remoteWebDriverPassword();
              Configuration.remote = String.format(Project.config.remoteWebDriverUrl(), user, password);
          }
      }
  
      @AfterEach
      void afterEach() {
          // SKIPPED FOR NOW
          // AllureAttachments.addScreenshotAs("Last screenshot");
          // AllureAttachments.addPageSource();
          // AllureAttachments.addBrowserConsoleLogs();
  
          // if(Project.config.videoStorage() != null) {
          //     AllureAttachments.addVideo();
          // }
  
          Selenide.closeWebDriver();
      }
  }
  ```

  to

  ```python
  import config
  
  
  @pytest.fixture(scope='function', autouse=True)
  def manage_browser():
      browser.config.timeout = config.settings.timeout
      driver = None
  
      if config.settings.browser == 'chrome':
          options = webdriver.ChromeOptions()
          # options.headless = True
          if not config.settings.remote_webdriver_url:
              driver = webdriver.Chrome(
                  ChromeDriverManager().install(),
                  options=options,
              )
      elif config.settings.browser == 'firefox':
          options = webdriver.FirefoxOptions()
          if not config.settings.remote_webdriver_url:
              driver = webdriver.Firefox(
                  executable_path=GeckoDriverManager().install(),
                  options=options,
              )
      else:
          raise Exception('not supported browser: ' + config.settings.browser)
  
      if config.settings.remote_webdriver_url:
          options.set_capability('enableVNC', True)
          options.set_capability('enableVideo', True)
          user = config.settings.remote_webdriver_user
          password = config.settings.remote_webdriver_password
          remote_webdriver_url = config.settings.remote_webdriver_url % (user, password)
  
          driver = webdriver.Remote(
              command_executor=remote_webdriver_url,
              options=options,
          )
  
      browser.config.driver = driver
  
      yield
  
      browser.quit()
  ```

* running examples

  * ` env -S "timeout=2.0 browser=firefox" pytest tests/ -n 2 --alluredir=reports`
  * ` env -S "timeout=2.0 remote_web_driver_url=https://%s:%s@selenoid.autotests.cloud/wd/hub/" pytest tests/ -n 2 --alluredir=reports`
    * ensure you edited your config.env with correct user & password

## Part III – Customizing Reporting

* TODO: next time;)


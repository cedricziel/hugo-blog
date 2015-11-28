+++
title = "Using Spring Security in integration tests"
subtitle = "Setup the testing environment to use your application context"
tagline = "Setup the testing environment to use your application context"
author = "Cedric Ziel"
template = "post.html"
keywords = "Cedric Ziel"
changefreq = "monthly"
priority = "0.9"
showdownExtensions = ['github', 'table', 'math', 'smartypants', 'footnotes']
date = "2015-10-09T21:58:29+01:00"
draft =  false

+++

Running Integration tests requires a bit more configuration, especially when you need setups for third-party software
like Liquibase for database migrations.

## The code

To leverage existing configuration, Spring 4.2 provides a set of annotations that will bootstrap the test
into a running Spring application.

Given you have a very simple controller that's secured with an ``SpEL`` expression:

```java
@CrossOrigin
@RestController
@RequestMapping(path = "/exampleRoute")
public class PrivateMessageController {

    /**
    * Returns a String "Hello" if the user has the role `USER`.
    *
    * Fails if the user is unauthenticated.
    */
    @PreAuthorize("hasRole('ROLE_USER')")
    @RequestMapping(method = RequestMethod.GET)
    public String sayHello() {

        return "Hello!";
    }
}
```

To test this behaviour, Spring Security needs to be set up for each test.
We can do this by putting some annotations on the test class, which will
be picked up by ``SpringJUnit4ClassRunner`` on test execution. The next step is to inject
the ``WebApplicationContext`` and configure it. I do it here in a method annotated with ``@Before``.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MyApplication.class)
@WebAppConfiguration
@IntegrationTest
public class MyExampleIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setupMockMvc() {

        this.mockMvc = MockMvcBuilders
            .webAppContextSetup(wac)
            // The important bit
            .apply(springSecurity())
            .build();
    }
    
    /**
    * We can now perform http-requests through the `mockMvc`
    * which will execute the endpoints on the application.
    *
    * The example below expects HTTP status 401 (`Unauthorized`)
    */
    @Test
    public void unauthorizedAccessIsntAThing() throws Exception {

        mockMvc.perform(
            get("/exampleRoute")
                .accept(MediaType.APPLICATION_JSON_VALUE))
            .andExpect(status().isUnauthorized());
    }
}
```

Running the test will result in a fully bootstrapped application. All properties will be picked up
and mockMvc is at your service.

It's a very convenient method to interact with the application for testing reasons.

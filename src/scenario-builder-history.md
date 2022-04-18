# [DRAFT] VanEscola ScenarioBuilder History

> This article is under construction!

### TO DO:

- Add code examples
- Write a second draft

How did we end up with the VanEscola Scenario Builder.

### Test implementation

Let's create a basic test from scratch on C#.

We use a library called Xunit. It allow us to run tests and assert values.

Run this basic test on Visual Studio.

```c#
using Xunit;

namespace Test
{
    public class TestClass
    {
        [Fact]
        public void TestA()
        {
            Assert.Equal(2, 1 + 1);
        }
    }
}

```

## Problem 1: We need tests!

Let's model a fictional integration test.

```c#
[Fact]
public void BasicIntegrationTest() {
    // First, we need to connect to the database
    var database = connectToDatabase();

    // Then, we clean the database so that the last test data doesnt interfere this one
    database.cleanDatabase();

    // Then we fill the database with the data we need for this test
    var user = new User() { };
    database.Users.add(user);

    var pet = new Pet() { owner = user };
    database.Pets.add(pet);

    // We start our API
    var API = initializeApi();
    createMockApi();

    // apply test actions
    ...

    // validate result
    ...
}
```

## Code smell 1: Duplication of test initialization

After a few tests, most of this code will be duplicated.

We then extracted all the repeating code to a class named Scenario that will represent each test environment.

## Code smell 2: Duplication of scenario setup

There is still have a lot of duplication because the test setups will always be slightly different for each test.

Lets create a ScenarioBuilder to facilitate the setup of different Scenarios. Then, let's extract each of our functions to a reusable build step of our builder.

## Code smell 3: Same build steps always appearing together

Note that the "AddLogin" and "AddTokenToClient" always appear together, this points to a bad separation of build steps. What we actually want is to represent the funcionality of "AddUserAuthorization" to use the UserToken in the HttpClient to simulate a loggedIn User. In the same way, we dont want to create a Membership separately from an User because we'll always need both.

## Code smell 4: Multiple implementations of AddUser

Instead of multiple implementations for each user, we can just parameterize our AddUser function for much more flexibility.

## Code smell 5: Passing the same parameters on most AddUser functions

Most of our AddUser functions will always take the same name and most of the time, we dont even care what the user is called, so lets just transform the "name" argument into an optional parameter. This way, we dont have to pass the same name everytime.

## Code smell 6: Too many arguments on a build step

Everytime we needed an entity with a specific value, we parameterized it's creation function. Eventually, our build step will look like this.

Most of the tests that call this function dont need any specific value, but there are a few that need their values to be some exact value.

Let's replace all the step parameters with a single optional setup function. This way, the tests that need a complete setup can make all this changes through a single parameter.

## Code smell 7: Same parameters appearing in multiples tests

### Why?

Some entities have properties that should be unique to each one, for example, the phonenumber of an user. When creating a scenario for a test that needs two users, we have to manually set at least one of the phonenumbers to avoid two users with the same one. And then, even though we dont need any specific value for a phonenumber, we have to set this manually everytime we have at least two users

### How to fix it?

Instead of using the same default phonenumber for each user, we can generate a random phonenumber everytime the AddUser function is called. This way, we can generate multiples users without having to manually setup their phonenumbers to avoid collision.

We use a library called Bogus that have multiple options for randomly generating different data sets like phonenumber, street address, names, etc.

## Problem 2: The Rise of the Heisenbugs

### What?

Some tests failing only sometimes.

### Why?

Due to our randomly generated values, sometimes a generated value breaks our test either because it is invalid or our code is broken, but then running it again generates a different value that works.

### How to fix it?

The Bogus library allows us to set a fixed seed. This way, even though the values used are different between each other, they will always be the same ones between runs.

## Code smell 8: Bogus constructor sucks

### Why?

The expected Bogus use is to create a different builder for each entity. This sucks because most of the time, all we want is a single simple entity and dont want to create a whole builder for it.

### How to fix it?

Bogus allow us to create a function that creates a randomized value. This allow for much simpler and flexible usages of the Bogus library.

## Code smell 9: Setting up entity relationships appearing a lot of times

### Why?

A lot of entities have no meaning by themselves or need another entity to exist, for example Membership and User. There is no Membership without an User. And so, everytime we create a Membership, we have to create a relationship between them.

### How to fix it?

If the entity being created needs another entity to exist, let's expect for it to already be created and automatically relate the created entity to this one.

## Problem 3: Entity expected for relationship doesnt exist

### Why?

Eventually, we'll forget the entities that another entity will need and addMembership without having called a addUser first. This will break the test saying something along the lines of "No User found" and it might take us a few annoying minutes of debugging before realizing our small mistake.

### How to fix it?

Assert that everything that your entity needs to be created is ready, and if not, throw a more descriptive error. For example, when using a addMembership without calling an addUser first can result in a "Could not create Membership. No User was found. Make sure to call addUser() BEFORE calling the addMembership() function". This will save you a lot of time and stress.

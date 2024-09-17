```
dotnet add package Moq
```
[NuGet Package](https://www.nuget.org/packages/Moq)

Moq is an useful C# library used for unit testing. This library makes it possible to mock, setup and verify dependencies, this makes sure the logic being tested is 100% independent. Mocking dependencies is essential for unit testing as it allows to isolate functions and test it's functionalities without worrying about other components. 
### How does mocking work: 
When mocking a method, we do not care about it's internal works at all. What Moq allow us to do is to pretend a function was executed with parameter input and returned a value. Both parameter and return value is specified by the developer on the test.
# Mocking Dependencies
In this example we use a basic **Service Layered Architecture**, where we pass a repository object to a service class. In the main application this is done by dependency injection, but for testing we are going to mock the repository dependency and pass it directly to the service through the constructor.

We will create the mock object by passing it's interface as the generic type and use it to create the service object:
> We can use the interface instead of the implemented class because we will be setting up the input and output later.
```Csharp
    // Make sure the mock property is of type `Mock` and not `IMock`.
public readonly Mock<IUserRepository> _mockRepository;
public readonly IUserService _service;

public UserServiceTests()
{
	_mockRepository = new Mock<IUserRepository>();
	_service = new UserService(_mockRepository.Object);
}
```
We will test the `UserService.FindUser` method. Because this method depends on the `UserRepository.FindByGuid` method, we will have to setup the dependency in our mock.
```CSharp
var foundUser = UserModel.Create("username", "email", "password");
 
_mockRepository.Setup(repo => repo.FindByGuid(It.IsAny<Guid>()))
.ReturnsAsync(User.Create(foundUser));
```
This code sets up the `FindByGuid` method from the mocked repository in a way that, when receiving any GUID, the method will return our `foundUser` object. Giving whoever uses that method the illusion that it executed successfully.

Now that the mock repository has been setup, we will test the actual logic we want to test. In this case the `UserService.FindUser` will use the `UserRepository.FindByGuid` that was mocked. 
```Csharp
var test = await _userService.FindUser(Guid.NewGuid());
```

### Verify
You can check if the mocked object behaved accordingly with out setup using the `Verify` method.
```CSharp
_mockRepository.Verify(repo => repo.SaveUser(It.IsAny<User>()), Times.Once);
```
This line of code verifies if the method `SaveUser` from the mock repository was executed once.
> The input parameter should be the same as the one used in the setup.

# Full Code
```CSharp
public class UserServiceTests
  {
    public readonly IUserService _service;
    public readonly Mock<IUserRepository> _mockRepository;

    public UserServiceTests()
    {
      _mockRepository = new Mock<IUserRepository>();
      _service = new UserService(_mockRepository.Object);
    }

    [Fact]
    public async Task RegisterUserTest_ShouldReturnAUserModelEntity()
    {
      var testEntity = UserModel.Create("username", "email", "password");

      _mockRepository
      .Setup(repo => repo.SaveUser(It.IsAny<User>()))
      .ReturnsAsync(User.Create(testEntity));

      var testMethod = await _service.RegisterUser(testEntity);

      _mockRepository.Verify(repo => repo.SaveUser(It.IsAny<User>()), Times.Once);

      Assert.NotNull(testMethod);
      Assert.IsType<UserModel>(testMethod);
    }
  }
```
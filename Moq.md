```
dotnet add package Moq
```
Moq is an useful C# library used for unit testing. This library makes it possible to mock, setup and verify dependencies, this makes sure the logic being tested is 100% independent
# Mocking Dependencies
In this example we use a basic **Service Layered Architecture**, where we pass a repository object to a service class. In the main application this is done by dependency injection, but for testing we are going to mock the repository dependency and pass it directly to the service through the constructor.

As a good practice we will create the mock object by passing it's interface as the generic type and use it to create the service object:
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
The next step is on the actual test. Right now the mocked repository does not know how to behave, so, we will need to set it up, providing proper context of input and output of it's methods.
```CSharp
    public async Task RegisterUserTest_ShouldReturnAUserModelEntity()
    {
      var testEntity = UserModel.Create("username", "email", "password");
      _mockRepository
      .Setup(repo => repo.SaveUser(It.IsAny<User>()))
      .ReturnsAsync(User.Create(testEntity));
    }
```
In this test, the very first thing we do is `Setup` the method `SaveUser` present in the repository (this method is called by `UserService`). With this setup the mocked repository now knows that it can receive a input parameter of any `User` and asynchronously return the created `User` entity.
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
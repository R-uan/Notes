```
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```
[Nuget Package](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/8.0.6)
# Authentication
Authentication is the process of identifying an user. In .NET this process is pretty straight forward. Actually, too straight forward, a lot of things can be done without much configuration which can give a feeling of being lost with how abstract this is.
### Schemas
The authentication registration and options are called schemas. We specify the schema when adding the service to the application.
```Csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme,
        options => builder.Configuration.Bind("JwtSettings", options))
    .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme,
        options => builder.Configuration.Bind("CookieSettings", options));
```
The `JwtBearerDefaults.AuthenticationScheme` on `AddAuthentication` service is the default authentication method when no other is solicited.
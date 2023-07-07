---
title: 使用ASP.NET自定义自定义API接口
author: 顾一
avatar: https://cdn.jsdelivr.net/gh/Ghamengde/BlogCDN@1.0.0/avatar.jpg
authorLink: Ghamengde.cn
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
fileName: demo
type: test
date: 2023-07-07 10:38:06
tags:
- C#
- ASP.NET
keywords:
description:
photos: https://cdn.jsdelivr.net/gh/Ghamengde/BlogCDN@1.0.3/2/3.jpg
---
## 如何使用C#定义自己的API
我们使用**ASP.NET**框架来更轻松的构建自己的API
##### 1. 我构建代码的环境为:
- VS studio 2022 社区版
- c# 11.0
- ASP.NET 7.0

其他外部库由业务需求决定
数据库选用 **SQL Server**

##### 2. 创建项目:
- 打开VS Studio创建项目
- 使用模板ASP.NET Core Web应用
- 使用以下设置
![](https://cdn.jsdelivr.net/gh/Ghamengde/BlogCDN@1.0.4/ASP.net/1.png)

##### 3. 更改配置文件:
- 创建一个新文件Startup.cs,把Program.cs里的方法封装到Startup.cs里(因为网上大多数都是使用的Startup.cs,这样查询文档的时候更加方便)
###### 以下分别为Program.cs和Startup.cs文件:


        using webapi_2;
            
        var builder = WebApplication.CreateBuilder(args);
        
        var startup = new Startup(builder.Configuration);
        startup.ConfigureServices(builder.Services);
        
        var app = builder.Build();
        startup.Configure(app, builder.Environment);
        
        app.Run();



---
    
        
        using Microsoft.EntityFrameworkCore;
        namespace webapi_2;
        
        public class Startup
        {
            public Startup(IConfiguration configuration) { Configuration = configuration; }
            public IConfiguration Configuration { get; }
        
            public void ConfigureServices(IServiceCollection services)
            {
                services.AddMvc();
                services.AddControllers();
                services.AddEntityFrameworkSqlServer().AddDbContext<CoreContext>
                       (options => options.UseSqlServer(@"Data Source=DESKTOP-EVE8RSL\GHAMENGDESERVER;Initial Catalog=WebDb;Integrated Security=True;TrustServerCertificate=true"));
        
                services.AddRazorPages();
        
                services.AddSession(options =>
                {
                    options.IOTimeout = TimeSpan.FromDays(1); //会话一天超时
                    options.Cookie.HttpOnly = true; //设置为httponly,提高安全性 
                    options.Cookie.IsEssential = true; //设置cookie为必需的
                });
                
            }
        
            public void Configure(WebApplication app, IWebHostEnvironment env) 
            {
                if(!app.Environment.IsDevelopment())
                {
                    app.UseExceptionHandler("/Error");
                    app.UseHsts();
                }
                app.UseHttpsRedirection();
                app.UseStaticFiles();
                app.UseRouting();
                app.UseAuthorization();
                app.MapRazorPages();
                app.MapControllers();
                app.UseSession();
            }
        }
    

##### 3.构建自己的Model类:
- 例如我要构建一个UserModel，创建一个文件UserModel.cs或者创建一个文件夹放置模型文件
- 定义自己的UserModel类如下:


        using System.ComponentModel.DataAnnotations;

        namespace webapi_2
        {
            public class UserModel
            {
                [Key]
                public int Id { get; set; }
                public string Name { get; set; }
                public string Email { get; set; }
                public string Password { get; set; }
                public string UserName { get; set; }
                public int Age { get; set; }
                public string Sex { get; set; }
                //public string PasswordHash { get; set; } = string.Empty;
                public UserModel(int id, string name, string email, string password, string UserName, int Age, String Sex) 
                { 
                    this.Id = id;
                    this.Name = name;
                    this.Email = email;
                    this.Password = password;
                    this.UserName = UserName;
                    this.Age = Age;
                    this.Sex = Sex;
                }
            }
        }



##### 4. 编写[数据库上下文]文件:
- 创建一个类CoreContext继承至DBContext.
- 把UserModel添加到该类中，代码如下：

        using Microsoft.EntityFrameworkCore;

        namespace webapi_2
        {
            public class CoreContext : DbContext
            {
                public CoreContext(DbContextOptions options) : base(options) { }
                public DbSet<UserModel> Users { get; set; }
            }
        }


- 把CoreContext添加到Startup.cs的ConfigureServices中,添加以下代码:


        services.AddEntityFrameworkSqlServer().AddDbContext<CoreContext>
               (options => options.UseSqlServer(ConnectString));


-其中的ConeectString为连接字符串，用于连接数据库，如果不知道，
在vs studio中工具栏里面连接到数据库,对数据库右键，查看属性，右下角显示连接字符串:

![](https://cdn.jsdelivr.net/gh/Ghamengde/BlogCDN@1.0.4/ASP.net/2.png)

##### 5. 编写配置文件：
- appsettings.json:在此文件第一行添加
"ConnectionString":{"DefaultConnection": String}
此处String为连接字符串

##### 6. 数据库迁移：
- 有数据库上下文并且配置好数据库连接后，在导航栏的工具里，点击NuGet管理器，
打开程序包管理器控制台.
- 在控制台输入指令 Add-Migration . 它会在项目文件下寻找上下文文件并进行迁移.
- 成功之后，输入 Update-Database . 刷新数据库会发现出现了你自己建的Model.

##### 7. 创建Web API控制器
- 创建一个Controller文件夹放置控制器.
- 右击Controller文件夹,点击添加,点击控制器.
- 在出现的窗口左边选择[通用]里的[API].
- 选择 [其操作使用EntityFrameWork的API控制器],点击确定,
会生成一个具有四种http请求方式的API模板,我们可以对其进行
修改.


---

### 结语

**在上述步骤完成后.你就成功地使用C#定义了自己的最基础API.现在你可以根据你的业务需求和实际情况进行进一步开发和定制化.可以通过在控制器中编写不同的动作方法（ActionMethod）来处理客户端请求.并使用定义的模型类和数据库上下文来与数据库进行交互.**

**记得在开发过程中，要注意保护和验证用户的输入，以及进行身份验证和授权等安全性考虑。此外，还可以通过配置路由、使用中间件等来进一步定制和优化你的 API.**

**希望这些信息能够帮助你开始构建自己的C#API！如果你有任何进一步的问题,请随时提问.**
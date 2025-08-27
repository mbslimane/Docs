# Dotnet_postgresql

This file is designed to document how to set up a full dotnet project and link
it to a db.

## Docker Compose

Developing in a machine is best when separating the between projects using
docker containers,
for this projects we set the docker compose for both DB and Dotnet Back-end

``` yml
services:
  app:
    image: slimane16/dotnet-dev-img
    container_name: dotnet_amichai
    user: ramboe
    volumes:
      - ./backend/:/app       # Mount your host App folder into container
      - /home/ramboe/.config/nvim:/home/ramboe/.config/nvim #Copy the nvim
      #setting from the host with the same user as the host
      - /home/ramboe/.local/share/nvim:/home/ramboe/.local/share/nvim
      - /home/ramboe/.local/state/nvim:/home/ramboe/.local/state/nvim
    working_dir: /app
    ports:
      - "5001:5001"
    environment:
      - ASPNETCORE_URLS=http://+:5001
      - ASPNETCORE_ENVIRONMENT=Development
    stdin_open: true
    tty: true
    depends_on:
      - db
    networks:
      - devnet_amichai

  db:
    image: postgres:16
    container_name: pg_dev_amichai
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: devdb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - devnet_amichai

volumes:
  pgdata:

networks:
  devnet_amichai:

```

Start this compose by

```
  docker compose up -d
  docker exec -it <containera name>
```

## Configure the Db with dotnet

Depending on what db server to use follow the installation step `postgresql` or `sqlite`;

### SetUp the to dotnet

Add In the `appsetting.json`

``` json
{
  "ConnectionStrings":{
    "DefaultConnection":"host=<db-container-name db>;prot=5321;Database=<DBname>;Username=<username>;Password=<Password>;TrustServerCertificate=True"
  }
}
```

#### Add this packages

```
  dotnet add package Npgsql.EntityFrameworkCore.PostgreSql
  dotnet add package Microsoft.EntityFrameworkCore.design
  dotnet add package Microsoft.EntityFrameworkCore.tools
```

To check the packages installed go to the `projec.csproj` and see the packages
add are there.

#### Create the context ex: BuberDinnerContext

``` csharp
  using Microsoft.EntityFrameworkCore;
  namesapce AppName.dir
  {
    public BuberDinnerContext(DbContextOptions<BuberDinnerContext>options): base(options){}

    public DbSet<Book> Books {get;set;}   
     // Book is the modal I have in this exemple
      -- imported form dir models which are the tables to create;
  }
```

#### Update the Program.cs

``` csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

builder.services.AddDbContext<BuberDinnerContext>(options =>
  options.UseNpgsql(connectionString);
)
```

#### create migrations

This creates the migrations -- check it in the migrations dir

```bash
  dotnet ef migrations add "inital-migration" --startup-project
```

Update the db to have this migrations

``` bash
  dotnet ef Database update --startup-project
```

> [!NOTE]
> sometimes that --startup-project is not needed when using docker containers.

check the db, it should have the entities declared on the Models

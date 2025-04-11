# 📘 Entity Framework Code First: Comprehensive Guide

A complete reference for using **Entity Framework (EF) Code First** in .NET projects. This guide covers the fundamentals, configuration approaches, database migrations, relationships, and best practices, with clear examples and clean code snippets.

---

## 🔍 Table of Contents

1. [What is Entity Framework?](#what-is-entity-framework)
2. [Code First Approach](#code-first-approach)
3. [Model Configuration: Data Annotations](#model-configuration-data-annotations)
4. [Model Configuration: Fluent API](#model-configuration-fluent-api)
5. [Database Migrations](#database-migrations)
6. [Model Relationships](#model-relationships)
   - [One-to-Many (1:N)](#one-to-many-1n)
   - [Many-to-Many (MN)](#many-to-many-mn)
   - [One-to-One (11)](#one-to-one-11)
7. [Seeding Initial Data](#seeding-initial-data)
8. [Best Practices](#best-practices)
9. [Summary](#summary)

---

## 🧠 What is Entity Framework?

**Entity Framework (EF)** is an Object-Relational Mapper (ORM) that allows .NET developers to work with databases using C# objects instead of SQL.

### Key Features

- **Code First Approach**: Define DB schema with C# classes.
- **Automatic CRUD**: No SQL needed for basic operations.
- **LINQ Support**: Query DB using LINQ syntax.
- **Migrations**: Incrementally update DB schema.

---

## 🏗 Code First Approach

You define your models first, EF generates the DB schema.

```csharp
public class Student
{
    public int StudentId { get; set; } // Primary Key (auto-detected)
    public string Name { get; set; }
    public int Age { get; set; }
}

public class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; } // Represents a DB table
}
```

---

## 🏷 Model Configuration: Data Annotations

Use attributes to configure model behavior.

| Attribute       | Purpose                          | Example                                               |
|----------------|----------------------------------|-------------------------------------------------------|
| `[Key]`         | Marks property as Primary Key    | `[Key] public int Id { get; set; }`                   |
| `[Required]`    | Field must not be null           | `[Required] public string Name { get; set; }`         |
| `[MaxLength]`   | Sets max string length           | `[MaxLength(50)] public string Title { get; set; }`   |
| `[ForeignKey]`  | Foreign key relationship         | `[ForeignKey("StudentId")] public Student Student`    |
| `[NotMapped]`   | Exclude property from DB         | `[NotMapped] public string FullName { get; set; }`    |

### Example

```csharp
public class Student
{
    [Key]
    public int StudentId { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [NotMapped]
    public string DisplayName => $"Student: {Name}";
}
```

---

## ⚙️ Model Configuration: Fluent API

Advanced configuration using `OnModelCreating`.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Student>()
        .HasKey(s => s.StudentId);

    modelBuilder.Entity<Student>()
        .Property(s => s.Name)
        .IsRequired()
        .HasMaxLength(100);

    modelBuilder.Entity<Course>()
        .HasOne(c => c.Student)
        .WithMany()
        .HasForeignKey(c => c.StudentId);
}
```

---

## 📦 Database Migrations

### Common Commands (PMC)

| Command                               | Purpose                                |
|--------------------------------------|----------------------------------------|
| `Enable-Migrations`                  | Enables migrations (run once)          |
| `Add-Migration <Name>`               | Creates migration class                |
| `Update-Database`                    | Applies latest migration               |
| `Update-Database -TargetMigration:X` | Rollback to specific migration         |

### Example

```bash
Enable-Migrations
Add-Migration InitialCreate
Update-Database
Update-Database -TargetMigration:"PreviousMigrationName"
```

### Migration File

```csharp
public partial class InitialCreate : DbMigration
{
    public override void Up()
    {
        CreateTable(
            "dbo.Students",
            c => new
            {
                StudentId = c.Int(nullable: false, identity: true),
                Name = c.String(nullable: false, maxLength: 100),
            })
            .PrimaryKey(t => t.StudentId);
    }

    public override void Down()
    {
        DropTable("dbo.Students");
    }
}
```

---

## 🔗 Model Relationships

### One-to-Many (1:N)

```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public ICollection<Course> Courses { get; set; }
}

public class Course
{
    public int CourseId { get; set; }
    public string Title { get; set; }
    public int StudentId { get; set; }
    public Student Student { get; set; }
}
```

### Many-to-Many (M:N)

```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public ICollection<StudentCourse> StudentCourses { get; set; }
}

public class StudentCourse
{
    public int StudentId { get; set; }
    public Student Student { get; set; }

    public int CourseId { get; set; }
    public Course Course { get; set; }
}
```

### One-to-One (1:1)

```csharp
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public StudentProfile Profile { get; set; }
}

public class StudentProfile
{
    public int StudentProfileId { get; set; }
    public string Address { get; set; }

    public int StudentId { get; set; }
    public Student Student { get; set; }
}
```

---

## 🌱 Seeding Initial Data

```csharp
protected override void Seed(SchoolContext context)
{
    context.Students.AddOrUpdate(
        s => s.Name,
        new Student { Name = "John Doe", Age = 20 },
        new Student { Name = "Jane Smith", Age = 22 }
    );

    context.SaveChanges();
}
```

---

## ✅ Best Practices

- ✅ Use Fluent API for complex mappings.
- ✅ Create a new `DbContext` per request in web apps.
- ✅ Use `Add-Migration` after each model update.
- ✅ Avoid `AutomaticMigrationsEnabled = true` in production.
- ✅ Use `.AsNoTracking()` for read-only queries.

---

## 📝 Summary

| Topic              | Key Takeaway                                 |
|--------------------|-----------------------------------------------|
| Code First         | Define DB schema using C# classes             |
| Data Annotations   | Quick and easy model configuration            |
| Fluent API         | Precise, advanced control of mappings         |
| Migrations         | Version-controlled schema updates             |
| Relationships      | 1:N, M:N, 1:1 relationship support             |
| Seeding Data       | Populate DB with initial sample data          |

---

> 🔗 Feel free to fork, star, and contribute to this repo. Happy coding with Entity Framework! 🚀

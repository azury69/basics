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

3. 📌 Data Annotations
Data Annotations are attributes applied to model classes to configure database mappings.

Common Data Annotations
Attribute	Purpose	Example
[Key]	Marks a property as Primary Key	[Key] public int Id { get; set; }
[Required]	Ensures the field is not null	[Required] public string Name { get; set; }
[MaxLength]	Sets maximum string length	[MaxLength(50)] public string Title { get; set; }
[ForeignKey]	Defines a foreign key relationship	[ForeignKey("StudentId")] public Student Student { get; set; }
[NotMapped]	Excludes a property from DB mapping	[NotMapped] public string FullName { get; set; }
📦 Example
csharp
Copy
Edit
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

public class Course
{
    [Key]
    public int CourseId { get; set; }

    [Required]
    public string Title { get; set; }

    [ForeignKey("Student")]
    public int StudentId { get; set; }

    public Student Student { get; set; }
}
✅ StudentId is the Primary Key.
✅ Name is required and has a max length of 100.
✅ DisplayName is not mapped to the database.
✅ Course has a foreign key relationship with Student.

4. ⚙️ Fluent API (Advanced Configuration)
Fluent API provides more control over database mappings than Data Annotations.

Common Fluent API Configurations
Method	Purpose	Example
.HasKey()	Sets Primary Key	modelBuilder.Entity<Student>().HasKey(s => s.StudentId);
.Property()	Configures a property	modelBuilder.Entity<Student>().Property(s => s.Name).IsRequired().HasMaxLength(100);
.HasRequired()	Defines a required relationship	modelBuilder.Entity<Course>().HasRequired(c => c.Student).WithMany().HasForeignKey(c => c.StudentId);
.Ignore()	Excludes a property	modelBuilder.Entity<Student>().Ignore(s => s.DisplayName);
📦 Example
csharp
Copy
Edit
public class SchoolContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure Student
        modelBuilder.Entity<Student>()
            .HasKey(s => s.StudentId);

        modelBuilder.Entity<Student>()
            .Property(s => s.Name)
            .IsRequired()
            .HasMaxLength(100);

        // Configure Course
        modelBuilder.Entity<Course>()
            .HasKey(c => c.CourseId);

        modelBuilder.Entity<Course>()
            .HasOne(c => c.Student)
            .WithMany()
            .HasForeignKey(c => c.StudentId);
    }
}
5. 🛠️ Migrations
Migrations allow incremental updates to the database schema.

📋 Commands (Package Manager Console)
Command	Purpose
Enable-Migrations	Enables migrations (only once)
Add-Migration <Name>	Creates a new migration
Update-Database	Applies pending migrations
Update-Database -TargetMigration:<Name>	Rolls back to a specific migration
🔁 Example Workflow
Enable Migrations:

bash
Copy
Edit
Enable-Migrations
Add a Migration:

bash
Copy
Edit
Add-Migration InitialCreate
Update Database:

bash
Copy
Edit
Update-Database
Rollback Migration:

bash
Copy
Edit
Update-Database -TargetMigration:"PreviousMigrationName"
📦 Migration File Example
csharp
Copy
Edit
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
6. 🔗 Relationships in EF Code First
EF supports the following relationships:

One-to-Many (1:N)

Many-to-Many (M:N)

One-to-One (1:1)

(1) One-to-Many (1:N)
A Student can have multiple Courses.

📌 Data Annotations
csharp
Copy
Edit
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
🔧 Fluent API
csharp
Copy
Edit
modelBuilder.Entity<Course>()
    .HasOne(c => c.Student)
    .WithMany(s => s.Courses)
    .HasForeignKey(c => c.StudentId);
(2) Many-to-Many (M:N)
A Student can enroll in multiple Courses, and a Course can have multiple Students.

📌 Data Annotations
csharp
Copy
Edit
public class Student
{
    public int StudentId { get; set; }
    public string Name { get; set; }
    public ICollection<StudentCourse> StudentCourses { get; set; }
}

public class Course
{
    public int CourseId { get; set; }
    public string Title { get; set; }
    public ICollection<StudentCourse> StudentCourses { get; set; }
}

public class StudentCourse
{
    public int StudentId { get; set; }
    public Student Student { get; set; }
    public int CourseId { get; set; }
    public Course Course { get; set; }
}
🔧 Fluent API
csharp
Copy
Edit
modelBuilder.Entity<StudentCourse>()
    .HasKey(sc => new { sc.StudentId, sc.CourseId });

modelBuilder.Entity<StudentCourse>()
    .HasOne(sc => sc.Student)
    .WithMany(s => s.StudentCourses)
    .HasForeignKey(sc => sc.StudentId);

modelBuilder.Entity<StudentCourse>()
    .HasOne(sc => sc.Course)
    .WithMany(c => c.StudentCourses)
    .HasForeignKey(sc => sc.CourseId);
(3) One-to-One (1:1)
A Student has one StudentProfile.

📌 Data Annotations
csharp
Copy
Edit
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
🔧 Fluent API
csharp
Copy
Edit
modelBuilder.Entity<Student>()
    .HasOne(s => s.Profile)
    .WithOne(p => p.Student)
    .HasForeignKey<StudentProfile>(p => p.StudentId);
7. 🌱 Seeding Data
Use Seed() method to pre-populate your DB.

csharp
Copy
Edit
protected override void Seed(SchoolContext context)
{
    context.Students.AddOrUpdate(
        s => s.Name,
        new Student { Name = "John Doe", Age = 20 },
        new Student { Name = "Jane Smith", Age = 22 }
    );

    context.SaveChanges();
}
✅ Runs when Update-Database is executed.

8. ✅ Best Practices
✅ Use Fluent API for complex configurations.

✅ Keep DbContext short-lived (per request).

✅ Run Add-Migration after model changes.

❌ Avoid AutomaticMigrations in production.

✅ Use .AsNoTracking() for read-only queries.

🧾 Final Summary
Topic	Key Takeaway
Code First	Define DB schema using C# classes.
Data Annotations	Simple attributes for model configuration.
Fluent API	Advanced configuration for complex mappings.
Migrations	Incrementally update the database schema.
Relationships	Define 1:N, M:N, and 1:1 relationships.
Seeding Data	Pre-populate the database with initial data.

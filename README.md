# EF Core Array Column

It is currently not supported to have EF Core correctly map an array column of any sort.

```cs
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string[] Hobbies { get; set; }
}
```

`Hobbies` will not be accepted by EF at runtime model validation.

The proposed solutions to modeling this are either to create and entity and have a navigation property for it:

```cs
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public ICollection<PersonHobby> Hobbies { get; set; }
}

public class PersonHobby
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

This will of course create another database table and the hobbies will be joined on the person in case
`.Include(p => p.Hobbies)` is used to enforce eager loading.

Another alternative is to store the values as a serialized string value and have an extra unmapped property which converts back and forth:

```cs
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string RawHobbies { get; set; }

    [NotMapped]
    public string[] Hobbies
    {
        get
        {
            return RawHobbies.Split(';');
        }
        set
        {
            RawHobbies = string.Join(value, ';');
        }
    }
}
```

The choice of the separator needs to be mindful of the possible values of the individual items.

EF Core provides a way to carry out the conversions for you using the fluent API as well as documented here:

https://docs.microsoft.com/en-us/ef/core/modeling/value-conversions

This is a nicer way to do the above extra property, however it requires using the fluent API which I dislike.

## To-Do

### Prototype with the converter

### See how SQLite/SQL Server JSON support could be used (maybe a stored procedure directly or in the converter)
